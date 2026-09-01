# PLAN — Stress-test E2E KiCad MCP — Amplificateur Hi-Fi

## Objectif final

Transformer le cahier des charges en projet KiCad stéréo OPA1612/TPA3255 documenté et aussi fabricable que les preuves disponibles le permettent, principalement via `kicad-control` et son MCP, sans édition directe de `.kicad_sch` ou `.kicad_pcb`.

## Invariants

- Répondre et documenter en français ; préserver exactement noms, erreurs, commandes et chemins.
- Alimentation DC externe uniquement ; aucun secteur 230 V sur le PCB.
- Toute décision critique vient d’une source fabricant actuelle, d’un calcul explicite ou porte `NEEDS_DATA`.
- Toute opération directe sur KiCad appartient exclusivement à `kicad-control`/MCP ; aucun contournement par édition de `.kicad_sch`/`.kicad_pcb`.
- Gate ERC validé explicitement avant tout début de PCB.
- Ne jamais assimiler ERC/DRC à une validation physique de THD+N, SNR, EMI/EMC, thermique ou stabilité réelle.
- Préserver les fichiers utilisateur ; le principal seul possède Git et les checkpoints.

## Critères globaux de réussite

- Statuts séparés : SCHÉMA, PCB, ERC, DRC, CONFORMITÉ DATASHEETS, CRÉDIBILITÉ ÉLECTRONIQUE, AUTONOMIE MCP, PRÊT À FABRIQUER.
- Livrables : projet, schéma, PCB 4 couches, footprints, BOM, ERC, DRC, design review, calculs, hypothèses, limitations, `NEEDS_DATA`, benchmark MCP.
- Métriques MCP uniquement si observées dans l’état live.

# Phase A — Inspection et architecture

## A0 — Établir l’état initial et la continuité

### Objectif

Prouver l’état du workspace, de Git, de KiCad, de l’IPC/API, du MCP et de ses outils avant toute création KiCad.

### Dépendances

Aucune.

### Tâches

- [x] A0.1 Inspecter le workspace et Git sans modification.
- [x] A0.2 Créer `plan.md`, `progress.md` et un rollback Git local.
- [x] A0.3 Sonder KiCad, IPC/API, MCP et outils exposés par des opérations read-only live.
- [x] A0.4 Consigner capacités, observabilité et limitations initiales.

### Validation

Preuves live compactes ; aucune supposition fondée sur une ancienne documentation.

## A1 — Figer l’architecture électronique initiale

### Objectif

Définir architecture, budgets, gains, interfaces, rails, protections, filtres et composants critiques à partir des sources TI et de calculs traçables.

### Dépendances

A0.3.

### Tâches

- [x] A1.1 Vérifier datasheets, EVM et recommandations TI actuelles avec pages/sections.
- [x] A1.2 Établir le schéma-bloc stéréo et le rôle exact de l’OPA1612/volume/interface différentielle.
- [x] A1.3 Calculer niveaux, gains, impédances, headroom, rails et budget de puissance.
- [x] A1.4 Définir alimentation auxiliaire, séquencement, RESET/MUTE/FAULT et protections.
- [x] A1.5 Justifier filtre LC, découplages, thermique, EMI et connectique.
- [x] A1.6 Enregistrer chaque inconnue critique comme `NEEDS_DATA` et valider la revue d’architecture.

### Validation

Chaque connexion/valeur critique est sourcée ou calculée ; architecture cohérente pour 4–8 Ω et ~2 × 100 W/8 Ω.

# Phase B — Schéma KiCad

## B1 — Créer et construire le schéma par blocs

### Dépendances

A1 validée.

### Tâches

- [x] B1.1 Créer le projet via MCP et vérifier sa réouverture.
- [x] B1.2 Ajouter alimentation DC, auxiliaires et connectique.
- [x] B1.3 Ajouter entrée/volume/analogique gauche puis inspecter les nets.
- [x] B1.4 Ajouter entrée/volume/analogique droite puis inspecter les nets.
- [x] B1.5 Ajouter TPA3255, interface différentielle et contrôles puis inspecter les nets.
- [x] B1.6 Ajouter découplages, puissance, filtres LC, sorties et protections.
- [x] B1.7 Revoir références, alimentations, nets critiques et connexions inattendues.

### Validation

Schéma complet inspecté bloc par bloc via la stack et conforme à l’architecture A1.

## B2 — Retouches schématiques issues des décisions mécaniques et de protection

### Objectif

Intégrer au schéma les décisions prises après le gate C1 : déport mécanique des entrées et du volume, et renforcement de la protection d'entrée 48 V.

### Dépendances

C1 PASS ; décisions utilisateur du 2026-08-31 ; sourcing fabricant de la protection 48 V.

### Tâches

- [x] B2.1 Déporter `RV1` hors carte : ajouter un connecteur de volume, recâbler les nets et sortir `RV1` du périmètre PCB. *(Connecteur `J4` créé et nets vérifiés ; exclusion PCB faite en B2.8.)*
- [x] B2.2 Convertir les entrées `J2`/`J3` en connecteur de câblage vers RCA de châssis, avec retour de masse maîtrisé.
- [x] B2.3 Corriger `Q301` : clamp Zener de grille et résistance de grille dimensionnés pour un rail 48 V. L'anti-inversion reste une fonction distincte du hot-swap, la diode de structure d'un MOSFET N côté haut conduisant en inversion.
- [x] B2.4 Dimensionner l'étage `LM5069` sur équations de datasheet : seuils de sous-tension et de surtension avec marge sous 65 V, résistance de shunt, limitation de puissance et temporisateur de défaut.
- [x] B2.5 Vérifier la SOA du MOSFET de hot-swap pendant la charge des 15 400 µF, contre la courbe SOA de la référence retenue. *(`Q302` = `IXTK200N10L2`, SOA garantie 625 W à 75 °C contre 389 W exigés, marge 1,61 ×.)*
- [x] B2.6 Capturer le bloc `LM5069` au schéma : contrôleur, MOSFET série, shunt et réseau de programmation.
- [x] B2.8 Exclure `RV1` du circuit imprimé (attribut `on_board` à `no`) et supprimer la propriété parasite `exclude_from_board` laissée sur ce symbole. *(PASS, par le MCP seul. Le blocage consigné ici — aucun outil n'écrivait `on_board` ni ne supprimait une propriété — a été levé dans Konnect v1.1.4 : `edit_schematic_component` accepte désormais `in_bom`, `on_board` et `dnp` comme attributs natifs du bloc symbole, et `fields: {"clé": null}` retire une propriété. La correction a été faite en un appel, `RV1` adressé par `uuid`. Diff du schéma : une ligne changée, `(on_board yes)` en `(on_board no)`, et les dix lignes du `(property "exclude_from_board" "")` supprimées ; rien d'autre dans le fichier n'a bougé. ERC identique avant et après, 0 erreur et 15 avertissements, inchangé depuis la porte C2. Script rejouable : `scripts/live-b28-on-board.ps1` du dépôt Konnect.)*
- [x] B2.7 Figer `F301` et `D301` sur des références réelles, la TVS étant explicitement cantonnée aux transitoires rapides et non à la protection en surtension. *(`F301` = `Schurter UMT-H` 12,5 A `3403.0285.11` ; `D301` = `SMDJ58CA`, bidirectionnelle pour ne pas annuler l'anti-inversion.)*

- [x] B2.9 Figer `Q301` sur une référence réelle : P-MOS bloquant l'inversion à 56 V plus marge, `I_D` ≥ 12 A continus, `R_DS(on)` faible sous `V_GS` = −15 V, boîtier dissipatif. *(`IPP330P10NM`, Infineon OptiMOS TO-220-3, −100 V / 33 mΩ.)*
- [x] B2.10 Corriger le brochage de `Q301` et `Q302` : les symboles `*_GSD` déclaraient broche 2 = Source alors que les deux composants retenus ont broche 2 = Drain. Défaut invisible à l'ERC, fatal au report PCB. *(Passés en `Q_PMOS_GDS` et `Q_NMOS_GDS` ; correctif géométriquement neutre, positions de broches identiques entre variantes.)*

### Validation

Chaque ajout est sourcé sur datasheet ou calculé explicitement ; aucun seuil de sécurité posé de mémoire ; toute grandeur dépendant d'un composant non figé reste `NEEDS_DATA`. Aucune coordonnée existante déplacée ; nets vérifiés par inspection MCP.

# Phase C — Gate schématique

## C1 — ERC et revue schématique obligatoire

### Dépendances

B1 validée.

### Tâches

- [x] C1.1 Exécuter ERC et archiver le résultat.
- [x] C1.2 Classer chaque erreur/avertissement et corriger les problèmes réels via MCP.
- [x] C1.3 Relancer ERC et vérifier alimentations, nets critiques, découplages et interfaces.
- [x] C1.4 Consigner explicitement PASS/FAIL du gate ; interdire Phase D si FAIL.

### Validation

Gate schématique explicite, reproductible, sans problème réel ERC non traité.

## C2 — Re-gate ERC après B2

### Objectif

Le gate C1 ne couvre plus le schéma une fois B2 appliquée ; le rejouer est obligatoire.

### Dépendances

B2 validée.

### Tâches

- [x] C2.1 Relancer ERC et comparer à la référence du gate C1 : 0 erreur / 14 avertissements.
- [x] C2.2 Traiter tout écart réel, puis consigner explicitement PASS/FAIL ; interdire la reprise de D1 si FAIL. **GATE C2 = PASS** (0 erreur / 15 avertissements : 5 mismatch librairie, 10 off-grid ; `reports/ERC_C2_gate_final-2026-08-31.json`).

### Validation

Gate schématique de nouveau explicite et reproductible sur le schéma modifié.

# Phase D — Footprints

## D1 — Attribuer et valider les empreintes

### Dépendances

C2 PASS.

### Tâches

- [x] D1.1 Associer chaque composant à une empreinte compatible et disponible. *(PASS. **Plus aucun composant du schéma n'est sans empreinte.** Les cinq `PWR_FLAG` n'en requièrent pas, et `RV1` est volontairement hors carte, voir B2.1 et B2.8. Vérifié en itérant sur les blocs `(symbol` de premier niveau du fichier, et non par le MCP. Toutes les empreintes de librairie ont été contrôlées présentes sur disque.)*
- [x] D1.5 Assigner les passifs du bloc de protection ajoutés en B2. *(PASS. `R305` à `R310`, `C325`, `C326` et `D302` assignés et vérifiés au fichier ; ERC inchangé, 15 warnings et 0 erreur, identique à la porte C2. Trois corrections ont été nécessaires sur le travail initial : la référence `MKS2C044701O00KSSD` de `C325` **n'existe pas**, remplacée par `MKS2B044701K00KSSD` ; son empreinte est passée de `W11.0mm` à `W7.2mm` ; `C326` est passé de `100nF` à `100nF/250V`. `D302` = `BZT52C15` lève son `NEEDS_DATA`. Détail et justifications : `docs/protection-48v.md`, section « Passifs du bloc de protection ».)*
- [x] D1.4 Créer les empreintes locales manquantes et choisir la connectique déportée. *(PASS. Connectique tranchée par l'utilisateur : **JST XH au pas de 2,5 mm**, vertical par défaut, l'orientation définitive relevant du placement en Phase E. `J2` et `J3` en `JST_XH_B2B-XH-A_1x02`, `J4` en `JST_XH_B6B-XH-A_1x06` ; nombre de broches confirmé 2/2/6. Ce qui décide est le détrompage de `J4`, dont le faisceau à six conducteurs rebranché à l'envers donnerait une panne silencieuse. Empreinte locale `Fuse_Schurter_UMT-H_5.3x16mm` créée pour `F301` et cotée sur le dessin d'implantation Schurter ; réserves graphiques suivies en D1.8.)*
- [x] D1.2 Vérifier boîtiers fabricant, orientations, courants, connecteurs et contraintes d’assemblage. *(PASS. **Vérifiés à la source et conformes** : `Q301` `IPP330P10NM`, PG-TO220-3, brochage 1 = Gate / 2 = Drain-semelle / 3 = Source, datasheet servie par le serveur Infineon, chiffres thermiques relus et deux conditions d'implantation ajoutées à `docs/protection-48v.md` ; `Q302` en TO-264-3 au pas de 5,45 mm et `Q301` en TO-220-3 au pas de 2,54 mm, les deux empreintes KiCad étant aux pas normalisés, l'orientation verticale restant compatible d'un montage sur radiateur et le choix définitif dépendant du `NEEDS_DATA` dissipateur ; `U8` `LM5069-2` en `MSOP-10_3x3mm_P0.5mm`, correct — la datasheet TI ne donne le composant qu'en VSSOP-10 (DGS) 3 × 3 mm **sans pad exposé**, et KiCad n'a pas d'empreinte `VSSOP-10`, l'équivalent normalisé étant bien MO-187 variation BA. À noter pour H2 : la référence commandable est `LM5069MM-2/NOPB`, absente du schéma. **Défauts ouverts issus de cette tâche** : D1.9 (films 680 nF) et D1.10 (inductances). **Bulks instruits en D1.11, inductances en D1.10, films en D1.9.** **Dernier point clos : le bornier `J1`.** Le dessin fabricant de CIXI MAIXU (`MX126-5.0-XXP`, planche du 14/09/2022) a fini par être obtenu : `www.lcsc.com` et `datasheet.lcsc.com` refusent `curl`, mais **`wmsc.lcsc.com` sert le fichier**. Il n'a aucune couche texte — c'est un plan vectoriel sans texte extractible — et `pdftoppm` n'est pas installé ; il a donc été **rendu en PNG à 200 dpi par `pymupdf` puis lu comme une image**. Il donne **300 V / 10 A en calibre UL et 250 V / 15 A en calibre IEC**, tenue 2000 VAC pendant 1 min, plage de fil **26 à 14 AWG, soit 0,5 à 2,5 mm²**, service de −40 à +105 °C. Les 4,6 A continus passent avec un facteur 2,2 sur le calibre UL, et les 10 A transitoires restent sous le calibre IEC. **L'empreinte est confirmée à la source** : le plan d'implantation impose un perçage de ø1,30 mm au pas de 5,00 mm, ce que `TerminalBlock_MaiXu_MX126-5.0-02P_1x02_P5.00mm` reprend exactement. **Contrainte d'assemblage à porter** : le câble d'alimentation doit rester entre 0,5 et 2,5 mm².)*
- [x] D1.6 Corriger l'annotation des cinq `PWR_FLAG`. *(PASS. Annotés `#FLG01` à `#FLG05` ; `#PWR001`, sixième `PWR_FLAG` préexistant, non touché. Plus aucun symbole ne porte `?`. **Le MCP sait adresser un symbole par `uuid`**, paramètre alternatif à `reference` d'`edit_schematic_component` : c'est ce qui a levé l'ambiguïté des cinq repères identiques. Le défaut était plus grave que consigné : les `PWR_FLAG` non annotés étaient exportés au netlist comme **six composants réels nommés `?`**, que « Update PCB from schematic » aurait tenté de placer sur la carte, sans empreinte. Après correction ils en sont exclus, comme tout symbole à préfixe `#`. Preuve : netlist exporté avant et après, 74 nets inchangés, aucun net créé ni supprimé, et les seuls nœuds retirés sont les cinq `?.1` sur `/+12V-OA`, `/PVDD`, `/GND`, `/+15V` et `/BUCK_VIN` ; ERC inchangé à 15 warnings et 0 erreur, et `kicad-cli` ne signale plus « erreurs de numérotation ».)*
- [x] D1.7 Confirmer sur le dessin coté WIMA l'encombrement du film `C325`. *(PASS, et **la réserve était fondée deux fois : la cote supposée était fausse, et la référence l'était aussi.** Catalogue WIMA extrait et lu : au pas de 5 mm, la cote constante de la série MKS2 est la **longueur** `L` = 7,2 mm, jamais l'épaisseur ; les tableaux donnent les boîtes en `W × H × L` et le 4,7 µF mesure `W` = 11 mm, `H` = 18 mm, `L` = 7,2 mm, sous 63 V comme sous 100 V. L'empreinte était donc trop étroite de 3,8 mm et le composant n'y serait pas entré. Le décodage de la clé de référence WIMA a de plus retourné la conclusion de D1.5 : **les champs 11-12 codent la boîte et le pas, le champ 15 seul code la tolérance.** Le `O` de `MKS2C044701O00KSSD` appartient au code de boîte `1O` et n'a jamais été un code de tolérance. La référence écartée en D1.5 comme inventée est donc la bonne, et c'est sa remplaçante qui n'existe pas : `MKS2B044701K00KSSD` cumule un `B0` qui n'est pas un code de tension WIMA — la MKS2 va de 63 à 630 VDC sans aucun calibre 50 V — et une boîte `1K` de 7,2 × 13 mm qui ne loge que 2,2 µF. `C325` reprend `MKS2C044701O00KSSD`, Value `4.7uF/63V`, empreinte `Capacitor_THT:C_Rect_L7.2mm_W11.0mm_P5.00mm_FKS2_FKP2_MKS2_MKP2` de la librairie standard — aucune empreinte locale nécessaire. Écriture MCP relue au fichier par le principal, position inchangée, ERC toujours à 15 warnings et 0 erreur. La marge de démarrage × 1,41 est conservée : le ± 10 % est catalogué, seule la tension nominale passe de 50 à 63 V. Détail : `docs/protection-48v.md`.)*
- [x] D1.9 Qualifier les quatre films `680 nF` du filtre de sortie, puis corriger leur empreinte. *(PASS. **Le pas de 5 mm était faux autant que l'épaisseur.** La nomenclature du `TPA3255EVM` (`SLOU441`, p. 14) tranche d'abord le diélectrique : l'EVM équipe ce poste d'un **`PHE426HB7100JR06` KEMET, film polypropylène, 1 µF / 250 V au pas de 17,5 mm**. Le catalogue WIMA `MKP4` a ensuite été extrait et lu : il **contredit l'idée qu'un MKP n'existerait pas sous 250 V** — la série se catalogue dès 100 VDC — et donne pour 0,68 µF **une boîte `4F` de 8 × 15 × 18 mm au pas de 15 mm, identique en 100 et en 250 VDC**, références `MKP4D036804F00` et `MKP4F036804F00`. Le calibre supérieur étant sans coût mécanique, **250 V retenu** : la marge sur les 100 V exigés par TI est gratuite. Les quatre condensateurs passent à `Capacitor_THT:C_Rect_L18.0mm_W8.0mm_P15.00mm_FKS3_FKP3` — empreinte de la librairie standard dont la description renvoie elle-même au catalogue WIMA — et leur `Value` à `680nF/250V MKP`. Écriture MCP relue au fichier par le principal, positions inchangées, ERC toujours à 15 violations et 0 erreur, jeu de messages identique. **Le choix de 680 nF face au 1 µF de l'EVM est confirmé sain** : avec 15 µH il donne 49,8 kHz de coupure contre 50,3 kHz pour le couple 10 µH / 1 µF de l'EVM, pour une impédance caractéristique de 4,7 Ω au lieu de 3,16 — mieux adaptée à des charges de 4 à 8 Ω. Les quatre derniers caractères de la référence WIMA, tolérance et conditionnement, restent à figer en H2 : la clé n'a pas été relue pour cette série. **L'empreinte locale `CF_Film_Box_P5.00mm_7.2x3.5mm`, devenue sans utilisateur, est supprimée** ; la librairie locale ne contient plus que le fusible et son entrée `fp-lib-table` reste justifiée. **Contrainte Phase E : la boîte passe de 7,2 × 3,5 à 18 × 8 mm sur 15 de haut, courtyard de 18,5 × 8,5 mm, quatre fois.**)* **Défaut prouvé, même mode de défaillance que D1.7 : une cote supposée, jamais lue.** `C321` à `C324` portent `CF_Film_Box_P5.00mm_7.2x3.5mm`, soit 3,5 mm d'épaisseur de corps. Or le condensateur de filtre est référencé à `GND` en aval de chaque demi-pont et voit donc tout le rail, et TI exige explicitement un calibre 100 V pour une alimentation de 51 V. Au pas de 5 mm, une boîte de 3,5 mm ne loge que 0,15 à 0,22 µF sous 100 V ; le 680 nF sous 100 V mesure 5 × 10 × 7,2 mm. L'empreinte est donc impossible quelle que soit la référence finalement retenue. **Ne pas la remplacer par une nouvelle supposition** : la référence reste `NEEDS_DATA`, et le diélectrique est le premier point à trancher — un filtre de sortie Class-D relève du polypropylène (MKP) et non du polyester (MKS), pour le `dV/dt` et les pertes, ce qui peut aussi déplacer le pas. Une fois la référence figée, prendre le membre correspondant de la famille standard `Capacitor_THT:C_Rect_L7.2mm_W*_P5.00mm_FKS2_FKP2_MKS2_MKP2`, qui couvre 2,5 à 11 mm d'épaisseur.
- [ ] D1.8 Corriger les graphiques de l'empreinte locale du fusible. **Volet film clos en D1.9** : l'empreinte locale redondante a été supprimée, il ne reste que le défaut cosmétique du fusible. **Périmètre réduit, et le point ne demande plus d'arbitrage utilisateur pour que le projet avance.**
  - `CF_Film_Box_P5.00mm_7.2x3.5mm` **sort du périmètre**. La librairie KiCad standard contient déjà `C_Rect_L7.2mm_W3.5mm_P5.00mm_FKS2_FKP2_MKS2_MKP2` — même corps 7,2 × 3,5 mm au pas de 5 mm, courtyard correct de 7,7 × 4,0 mm là où le local n'en déclare que 7,6 × 2,6. L'empreinte locale était donc **redondante dès sa création**, et le courtyard sous-dimensionné n'a jamais eu à être corrigé, seulement abandonné. Elle n'a plus d'utilisateur légitime (D1.9) et sera supprimée à la fermeture de D1.9, sans aucune édition de graphiques.
  - `Fuse_Schurter_UMT-H_5.3x16mm` reste locale et justifiée : le seul candidat standard, `Fuse:Fuse_Schurter_UMT250`, vise un corps de 3 × 10,1 mm avec pastilles à ± 4,25 mm, sans rapport avec les ± 6,875 mm du UMT-H. Son défaut résiduel est **purement cosmétique** — repère de broche 1 sur un composant non polarisé, cercle de sérigraphie hors courtyard à `x = −9,3`, sérigraphie à 0,15 mm des pastilles au lieu de 0,2 — alors que cuivre, pâte, masque et courtyard sont exacts. Sans effet sur le DRC ni sur la fabrication. Reporté sans blocage, à traiter si une version du MCP expose l'édition des graphiques.
- [x] D1.11 Requalifier les six condensateurs de bulk. *(PASS. **Défaut prouvé et corrigé sur `C312` à `C315`, empreinte validée sur `C316`/`C317`.** Source primaire : la nomenclature du kit d'évaluation TI `TPA3255EVM` (`SLOU441`, p. 14), qui est l'ancre déclarée du bulk dans `docs/architecture.md`.*
  - *`C312` à `C315`, 1500 µF/63 V : la BOM donne pour ce poste `EEU-FC1J152` Panasonic, boîtier **diamètre 18 mm**. L'empreinte portée était `CP_Radial_D16.0mm_P7.50mm`, soit **2 mm de trop peu en diamètre**, quatrième empreinte posée sur une cote supposée après D1.7, D1.9 et D1.10. Corrigée en `Capacitor_THT:CP_Radial_D18.0mm_P7.50mm`, membre de la librairie standard, pas inchangé à 7,5 mm. Écriture MCP relue au fichier par le principal, positions inchangées, ERC inchangé à 15 violations et 0 erreur. Le courtyard passe de 16,16 à 18,16 mm : **+2 mm sur chacun des quatre**, à reporter en Phase E. Hauteur nominale 35 mm.*
  - *`C316`/`C317`, 4700 µF/80 V : empreinte `CP_Radial_D35.0mm_P10.00mm_SnapIn` **confirmée juste**, par trois preuves indépendantes. La BOM donne `SLPX472M080H3P3` Cornell Dubilier en « D35 mm × L30 » ; la clé de référence CDE type SLP, décodée sur le catalogue `SLP.pdf` p. 2 (lettre de diamètre H = 35 mm, chiffre de longueur 3 = 30 mm), redonne bien ø35 × 30 ; et le diagramme « PC Board Mounting Holes » du même catalogue impose **pas 10,0 mm et deux perçages ø2,0 ± 0,1**, identiques au diagramme Vishay 058 PLL-SI. **Le perçage de 2,0 mm de l'empreinte KiCad n'est donc pas un jeu nul sur une broche de 2,0 mm : c'est la cote de perçage prescrite par les deux fabricants**, et toute la famille `CP_Radial_D*_P10.00mm_SnapIn` la reprend.*
  - *Deux réserves consignées, aucune bloquante. Le pas de 7,5 mm du ø18 n'a pas pu être relu chez Panasonic : `industrial.panasonic.com` ne répond pas du tout à `curl` (timeout, pas un refus). Il est **inchangé** par rapport à l'empreinte précédente, donc la correction ne substitue pas une supposition à une autre, mais il reste à confirmer. Et `SLPX472M080H3P3` n'apparaît pas dans le catalogue SLP courant, qui donne à 80 V `SLP472M080E4P3` en 30 × 45 et `SLP472M080H5P3` en 35 × 35 : `SLPX` est une série voisine ou antérieure. **Les deux MPN restent des candidats d'ancrage et ne sont pas inscrits au schéma** tant que la clé du fabricant n'est pas décodée, conformément à la leçon de D1.7.*

- [x] D1.10 Requalifier l'empreinte des quatre inductances de sortie `L301` à `L304`. *(PASS, **et la réserve énergétique était fondée**. La BOM du `TPA3255EVM` donne `MA5172-AE` Coilcraft pour ce poste, et sa datasheet — document Coilcraft 943, `https://www.coilcraft.com/pdfs/ma5172.pdf` — catalogue dans la même famille **`PA6331-AE` : 15 µH, DCR 31 mΩ, `I_sat` 20 A, `I_rms` 9,8 A à 20 °C d'échauffement et 14,2 A à 40 °C**, soit exactement le cahier des charges de `docs/architecture.md`, sur une pièce conçue pour les étages Class-D TI. **Retenue sur arbitrage utilisateur.** Elle confirme au passage que le `HCI-1350` était hors de cause : là où le boîtier supposé mesurait 12,8 × 12,8 × 4,7 mm, la pièce réelle est un **tore traversant debout de ø28,6 × 12,3 mm**. **Le tracé de la datasheet n'est pas à l'échelle** : mesuré sur les vecteurs du PDF, la vue de face donne 2,836 pt/mm et la vue de profil 2,463, 15 % d'écart ; les étiquettes font foi, comme chez Schurter. Elles donnent un entraxe de **10,0 ± 0,5 mm** et des broches de 0,96 à 1,07 mm. **Aucune empreinte standard ne convient** : `L_Toroid_Vertical_L28.6mm_W14.3mm_P11.43mm_Bourns_5700` a la bonne longueur mais un entraxe de 11,43 mm, hors tolérance de 1,43 mm ; `L_Toroid_Vertical_L26.7mm_W14.0mm_P10.16mm_Pulse_D` a le bon pas mais un courtyard trop court de 1,9 mm. **Deuxième empreinte locale justifiée du projet** : `L_Toroid_Vertical_L28.6mm_W12.3mm_P10.00mm_Coilcraft_PA6331`, créée par le MCP et **relue au fichier par le principal**. Contrairement à `CF_Film_Box`, les graphiques imposés par `create_footprint` sont **corrects** : courtyard à ± 14,55 en X et − 1,55 à 11,55 en Y, soit 0,25 mm autour de l'élément le plus extérieur — les pastilles, non le corps — et sérigraphie à 0,15 mm hors du corps, sans recouvrir les pastilles. Ce sont les conventions KLC, et elles valent mieux que les cotes plus grossières que le principal avait commandées. **La leçon de `CF_Film_Box` se précise : le générateur n'était pas en cause, les cotes qu'on lui donnait l'étaient.** `Value` portée à `15uH/20A`, ERC toujours à 15 violations et 0 erreur. Le `NEEDS_DATA` sur l'inductance tombe. Reste à vérifier en H2 que la référence est toujours approvisionnable.)* Elles portent `Inductor_SMD:L_Wuerth_HCI-1350` **sans MPN**, alors que l'inductance 15 µH est `NEEDS_DATA` : troisième empreinte posée sur une supposition, après D1.7 et D1.9. Le corps HCI-1350 ne fait que 12,8 × 12,8 × 4,7 mm. Point de mesure lu à la source, sur la référence même que cite l'empreinte KiCad (`744355019`) : dans ce corps, la série donne 0,19 µH à `I_SAT,10%` = 60 A, soit une énergie stockée de 342 µJ. Or le cahier des charges — 15 µH avec `I_sat` ≥ 10 A — en réclame 750 µJ, plus du double. L'énergie étant d'abord une propriété du volume et du matériau de noyau, **le boîtier 1350 est très probablement trop petit** ; c'est une déduction sur un seul point de mesure, à confirmer quand la référence sera figée, mais elle suffit à ne pas traiter cette empreinte comme acquise. Prévoir un corps plus grand et son impact sur le placement en Phase E.
- [x] D1.12 Rendre `fp-lib-table` portable. *(PASS. La librairie locale y était déclarée par un **chemin absolu Windows**, `C:\Users\FlowUP\Documents\...` : un clone du dépôt sur une autre machine, ou un simple déplacement du projet, aurait cassé la résolution des deux empreintes locales sans avertissement. Remplacé par `${KIPRJMOD}/HifiAmp_TPA3255_Local.pretty`. **`sym-lib-table` utilisait déjà `${KIPRJMOD}`** : la convention du projet était donc établie et l'incohérence isolée à ce seul fichier. Que KiCad résolve bien cette variable dans ce projet est prouvé par l'ERC lui-même, qui charge le symbole local `LM5069` déclaré de la même façon. Sauvegarde de l'original conservée le temps de la session.)*
- [x] D1.3 Vérifier particulièrement HTSSOP TPA3255, PowerPAD et stratégie de vias thermiques. *(PASS, **et la tâche se retourne : il n'y a pas de vias thermiques à concevoir sous le TPA3255**. `U6` portait `HTSSOP-44-1EP_6.1x14mm_P0.635mm_EP5.2x14mm_Mask4.31x8.26mm`, **empreinte d'un autre boîtier** : elle est taguée `Texas_DDW0044B` et pose une 45ᵉ pastille de 5,2 × 14 mm **sous** le composant. Or le TPA3255 est en `DDV0044D`, et la datasheet (`SLASEA8A`, p. 3) est explicite : *« The package type contains a PowerPAD that is located **on the top side of the device** for convenient thermal coupling to the heat sink »*. Le tableau des broches ajoute, pour le PowerPAD : *« Ground, connect to grounded heat sink »*. Le plan cothé (4218830/A, p. 44) donne un pad exposé de **6,72 à 7,30 mm sur 3,85 à 4,43 mm**, soit 7,01 × 4,14 nominal — et non 5,2 × 14 — tandis que le land pattern TI (p. 45) ne montre que **44 pastilles**, sans rien à braser dessous. Corrigé en `Package_SO:HTSSOP-44_6.1x14mm_P0.635mm_TopEP4.14x7.01mm`, taguée `Texas_DDV0044D`, 44 pastilles, pad thermique dessiné sur le dessus aux cotes nominales exactes. Écriture MCP relue au fichier par le principal, ERC toujours à 15 violations et 0 erreur. **La preuve chiffrée vient du tableau thermique 7.4 : `RθJC(bot)` est donné `n/a`** — TI ne caractérise même pas la voie par le dessous — alors que `RθJA` tombe de **50,7 à 2,4 °C/W** dès qu'un dissipateur est fixé sur le dessus, facteur 21, la note précisant que *« only path for dissipation is to the heatsink »*. **Le `NEEDS_DATA` annonçant un EP de 5,2 × 14 mm à confirmer est donc infirmé, pas confirmé**, et le `NEEDS_DATA` dissipateur devient le point dimensionnant unique du thermique du TPA3255. **Écart connu et voulu à reporter en Phase E** : le symbole compte 45 broches, la broche 45 étant `EP_45` câblée à `GND` — ce qui est correct au sens de TI — alors que l'empreinte n'a que 44 pastilles. L'import PCB signalera une broche sans pastille : c'est **attendu**, la mise à la masse du PowerPAD passant par le dissipateur, hors PCB. Au passage, le tableau des conditions recommandées confirme `L_OUT(BTL)` ≥ 5 µH, que les 15 µH de D1.10 satisfont largement.)*

- [ ] D1.13 Trancher la fenêtre de tension où le TPA3255 travaille hors spécification. **Relevé en D1.3, non instruit.** Le tableau des conditions recommandées distingue deux plafonds selon la charge : `PVDD` va jusqu'à **51 V typique et 53,5 V maximum sous 4 Ω**, mais jusqu'à 53,5 V typique et 56,5 V maximum sous ≥ 6 Ω, et seulement *« provided a reduced over-current threshold is set »*. Or le projet vise **4 à 8 Ω**, et le seuil de coupure haute du `LM5069` est réglé à **56,4 V** (`docs/protection-48v.md`). **Il existe donc une fenêtre de 53,5 à 56,4 V dans laquelle une charge de 4 Ω est alimentée au-dessus du maximum absolu du TPA3255 sans que la protection ne coupe.** Le raisonnement actuel — le plafond n'est pas fixé par le TPA3255 puisque `Q302` l'isole en surtension — ne couvre pas cette bande. Trois issues à peser : abaisser `V_OVH` vers 52 V, restreindre la charge admissible à ≥ 6 Ω, ou démontrer que l'alimentation externe retenue ne peut pas atteindre 53,5 V. **Dépend du `NEEDS_DATA` alimentation externe 48 V.** Arbitrage utilisateur probable.
- [ ] D1.14 Instruire les cinq `lib_symbol_mismatch` et les symboles reliquats. **Relevé en D1.3.** Les cinq avertissements sont un tiers de la baseline C2 et portent tous sur des symboles de la librairie locale : `LM5010ASD`, `LM2940IMP_12_FIXED`, `TPS3802K33`, `LM5069`, `TPA3255B`. Un `lib_symbol_mismatch` signifie que la copie embarquée dans le schéma diverge de la librairie ; **l'écart peut être anodin, un champ de propriété, ou grave, un brochage** — ce n'est pas tranché, donc la baseline repose sur cinq inconnues. À diagnostiquer symbole par symbole, en comparant broche à broche. Deux reliquats à traiter dans le même mouvement : le cache `lib_symbols` du schéma contient encore `TPA3255DDV`, `TPA3255DDV_TEST` — deux broches, manifestement un essai — et `TPA3255`, **qui n'existent plus dans le `.kicad_sym`** et que plus aucun composant n'utilise ; et la librairie garde `LM2940IMP-12` alors que `U2` est passé à `LM2940IMP_12_FIXED`. Ne rien supprimer avant d'avoir montré qu'aucun symbole n'y renvoie.

### Validation

Toutes les empreintes sont attribuées et revues contre leurs sources fabricant.

# Phase E — PCB 4 couches

## E1 — Définir stack-up, règles et placement

### Dépendances

D1 validée.

### Tâches

- [ ] E1.1 Créer le PCB 4 couches et documenter stack-up/règles/classes de nets.
- [ ] E1.2 Placer puissance Class-D, bootstrap/découplages/bulk et thermique.
- [ ] E1.3 Placer filtres LC, sorties, alimentation et boucles de retour.
- [ ] E1.4 Placer analogique faible bruit, volume et contrôles avec séparation fonctionnelle.
- [ ] E1.5 Revoir symétrie, retour des courants, masses, clearances et manufacturabilité.

### Validation

Placement guidé par contraintes TI et revue de design via workflow disponible.

# Phase F — Routage

## F1 — Router et créer les plans

### Dépendances

E1 validée.

### Tâches

- [ ] F1.1 Router boucles de commutation, alimentation Class-D et découplages.
- [ ] F1.2 Router sorties vers filtres LC et connecteurs avec largeurs justifiées.
- [ ] F1.3 Router analogique, contrôle puis signaux non critiques.
- [ ] F1.4 Créer plans/zones et vias thermiques ; remplir les zones.
- [ ] F1.5 Vérifier l’absence de ratsnest et les longueurs/vias inutiles critiques.

### Validation

Routage terminé, zones remplies, contraintes critiques inspectées.

# Phase G — Validation PCB

## G1 — DRC et design review

### Dépendances

F1 validée.

### Tâches

- [ ] G1.1 Exécuter DRC et archiver le résultat.
- [ ] G1.2 Revoir courts-circuits, alimentations, masses, découplages, puissance et thermique.
- [ ] G1.3 Corriger via MCP, relancer DRC et le workflow de design review.
- [ ] G1.4 Établir la revue manufacturabilité et les écarts résiduels.

### Validation

DRC final et design review documentés après corrections.

# Phase H — Revue électronique et livrables

## H1 — Revue audio/électronique indépendante

### Dépendances

G1 validée ou état final PCB explicitement PARTIAL.

### Tâches

- [ ] H1.1 Vérifier par calcul gain, headroom, impédances, coupures, LC, courants et dissipation.
- [ ] H1.2 Classer preuves : KiCad, documentation, calcul/simulation, non vérifié physiquement.
- [ ] H1.3 Documenter bruit/EMI/stabilité/thermique non mesurés et toute divergence TI.

### Validation

Rapport honnête, traçable, distinct d’ERC/DRC.

## H2 — Produire BOM, rapports et benchmark MCP

### Dépendances

H1.

### Tâches

- [ ] H2.1 Générer BOM et livrables exploitables disponibles.
- [ ] H2.2 Collecter métriques MCP observables, erreurs, retries, limitations et `NEEDS_DATA`.
- [ ] H2.3 Vérifier la présence de tous les livrables demandés.
- [ ] H2.4 Émettre les huit statuts finaux et `PRÊT À FABRIQUER` avec preuves.

### Validation

Livrables présents dans le workspace et rapport final sans métrique ni performance inventée.
