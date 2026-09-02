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
- [x] B2.4 Dimensionner l'étage `LM5069` sur équations de datasheet : seuils de sous-tension et de surtension avec marge sous le maximum absolu du TPA3255, résistance de shunt, limitation de puissance et temporisateur de défaut. *(Le « 65 V » d'origine était faux : `SLASEA8A` § 7.1 donne **69 V** pour `PVDD_X to GND`. Rectifié en D1.13. L'erreur était conservatrice et n'invalide aucun choix de B2.4 — la marge pire cas passe de 5,1 à 9,2 V.)*
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
- [x] D1.8 Corriger les graphiques de l'empreinte locale du fusible. **Volet film clos en D1.9** : l'empreinte locale redondante a été supprimée, il ne reste que le défaut cosmétique du fusible. **Périmètre réduit, et le point ne demande plus d'arbitrage utilisateur pour que le projet avance.**
  - `CF_Film_Box_P5.00mm_7.2x3.5mm` **sort du périmètre**. La librairie KiCad standard contient déjà `C_Rect_L7.2mm_W3.5mm_P5.00mm_FKS2_FKP2_MKS2_MKP2` — même corps 7,2 × 3,5 mm au pas de 5 mm, courtyard correct de 7,7 × 4,0 mm là où le local n'en déclare que 7,6 × 2,6. L'empreinte locale était donc **redondante dès sa création**, et le courtyard sous-dimensionné n'a jamais eu à être corrigé, seulement abandonné. Elle n'a plus d'utilisateur légitime (D1.9) et sera supprimée à la fermeture de D1.9, sans aucune édition de graphiques.
  - `Fuse_Schurter_UMT-H_5.3x16mm` reste locale et justifiée : le seul candidat standard, `Fuse:Fuse_Schurter_UMT250`, vise un corps de 3 × 10,1 mm avec pastilles à ± 4,25 mm, sans rapport avec les ± 6,875 mm du UMT-H. Son défaut résiduel est **purement cosmétique** — repère de broche 1 sur un composant non polarisé, cercle de sérigraphie hors courtyard à `x = −9,3`, sérigraphie à 0,15 mm des pastilles au lieu de 0,2 — alors que cuivre, pâte, masque et courtyard sont exacts. Sans effet sur le DRC ni sur la fabrication. Reporté sans blocage, à traiter si une version du MCP expose l'édition des graphiques. — **PASS.** Le report n'avait plus lieu d'être : **l'outil `set_footprint_graphics` existe bel et bien** dans le MCP, et la note « aucun outil n'édite les graphiques après coup » était fausse. Deux défauts corrigés : le repère de broche 1, supprimé, qui n'avait pas de sens sur un composant non polarisé et tombait de surcroît hors courtyard à `x = −9,3` ; et la sérigraphie, qui encadrait l'**enveloppe des pastilles** au lieu du **corps**, ramenée à deux segments horizontaux `x ∈ [−4,70 ; 4,70]` à `y = ±2,675`. Le `F.Fab` suit désormais le corps réel `±7,70 × ±2,675`, sans le chanfrein de broche 1. **Vérifié au fichier par le principal, pas sur rapport d'agent** : dégagement sérigraphie-pastille de **0,240 mm** contre 0,20 exigés par les KLC, aucun élément hors courtyard `±9,00 × ±3,05`, pastilles / pâte / masque / courtyard / `descr` rigoureusement inchangés, champ Footprint de `F301` intact. **Preuve indépendante du MCP** : `kicad-cli fp export svg` trace les deux empreintes locales sans avertissement.

  - **Nuance sur la leçon D1.10, qui ne se généralise pas.** Donner au générateur les cotes justes du corps a corrigé le `F.Fab` mais **pas** la sérigraphie : `create_footprint` la dérive de l'enveloppe des pastilles, laquelle déborde ici le corps de 1,05 mm de chaque côté puisque les pastilles dépassent en bout pour le congé. Sur une empreinte dont les pastilles sont plus larges que le corps, le générateur produit donc une sérigraphie non conforme quelles que soient les cotes fournies, et il faut la reprendre par `set_footprint_graphics`.
- [x] D1.11 Requalifier les six condensateurs de bulk. *(PASS. **Défaut prouvé et corrigé sur `C312` à `C315`, empreinte validée sur `C316`/`C317`.** Source primaire : la nomenclature du kit d'évaluation TI `TPA3255EVM` (`SLOU441`, p. 14), qui est l'ancre déclarée du bulk dans `docs/architecture.md`.*
  - *`C312` à `C315`, 1500 µF/63 V : la BOM donne pour ce poste `EEU-FC1J152` Panasonic, boîtier **diamètre 18 mm**. L'empreinte portée était `CP_Radial_D16.0mm_P7.50mm`, soit **2 mm de trop peu en diamètre**, quatrième empreinte posée sur une cote supposée après D1.7, D1.9 et D1.10. Corrigée en `Capacitor_THT:CP_Radial_D18.0mm_P7.50mm`, membre de la librairie standard, pas inchangé à 7,5 mm. Écriture MCP relue au fichier par le principal, positions inchangées, ERC inchangé à 15 violations et 0 erreur. Le courtyard passe de 16,16 à 18,16 mm : **+2 mm sur chacun des quatre**, à reporter en Phase E. Hauteur nominale 35 mm.*
  - *`C316`/`C317`, 4700 µF/80 V : empreinte `CP_Radial_D35.0mm_P10.00mm_SnapIn` **confirmée juste**, par trois preuves indépendantes. La BOM donne `SLPX472M080H3P3` Cornell Dubilier en « D35 mm × L30 » ; la clé de référence CDE type SLP, décodée sur le catalogue `SLP.pdf` p. 2 (lettre de diamètre H = 35 mm, chiffre de longueur 3 = 30 mm), redonne bien ø35 × 30 ; et le diagramme « PC Board Mounting Holes » du même catalogue impose **pas 10,0 mm et deux perçages ø2,0 ± 0,1**, identiques au diagramme Vishay 058 PLL-SI. **Le perçage de 2,0 mm de l'empreinte KiCad n'est donc pas un jeu nul sur une broche de 2,0 mm : c'est la cote de perçage prescrite par les deux fabricants**, et toute la famille `CP_Radial_D*_P10.00mm_SnapIn` la reprend.*
  - *Deux réserves consignées, aucune bloquante. Le pas de 7,5 mm du ø18 n'a pas pu être relu chez Panasonic : `industrial.panasonic.com` ne répond pas du tout à `curl` (timeout, pas un refus). Il est **inchangé** par rapport à l'empreinte précédente, donc la correction ne substitue pas une supposition à une autre, mais il reste à confirmer. Et `SLPX472M080H3P3` n'apparaît pas dans le catalogue SLP courant, qui donne à 80 V `SLP472M080E4P3` en 30 × 45 et `SLP472M080H5P3` en 35 × 35 : `SLPX` est une série voisine ou antérieure. **Les deux MPN restent des candidats d'ancrage et ne sont pas inscrits au schéma** tant que la clé du fabricant n'est pas décodée, conformément à la leçon de D1.7.*

- [x] D1.10 Requalifier l'empreinte des quatre inductances de sortie `L301` à `L304`. *(PASS, **et la réserve énergétique était fondée**. La BOM du `TPA3255EVM` donne `MA5172-AE` Coilcraft pour ce poste, et sa datasheet — document Coilcraft 943, `https://www.coilcraft.com/pdfs/ma5172.pdf` — catalogue dans la même famille **`PA6331-AE` : 15 µH, DCR 31 mΩ, `I_sat` 20 A, `I_rms` 9,8 A à 20 °C d'échauffement et 14,2 A à 40 °C**, soit exactement le cahier des charges de `docs/architecture.md`, sur une pièce conçue pour les étages Class-D TI. **Retenue sur arbitrage utilisateur.** Elle confirme au passage que le `HCI-1350` était hors de cause : là où le boîtier supposé mesurait 12,8 × 12,8 × 4,7 mm, la pièce réelle est un **tore traversant debout de ø28,6 × 12,3 mm**. **Le tracé de la datasheet n'est pas à l'échelle** : mesuré sur les vecteurs du PDF, la vue de face donne 2,836 pt/mm et la vue de profil 2,463, 15 % d'écart ; les étiquettes font foi, comme chez Schurter. Elles donnent un entraxe de **10,0 ± 0,5 mm** et des broches de 0,96 à 1,07 mm. **Aucune empreinte standard ne convient** : `L_Toroid_Vertical_L28.6mm_W14.3mm_P11.43mm_Bourns_5700` a la bonne longueur mais un entraxe de 11,43 mm, hors tolérance de 1,43 mm ; `L_Toroid_Vertical_L26.7mm_W14.0mm_P10.16mm_Pulse_D` a le bon pas mais un courtyard trop court de 1,9 mm. **Deuxième empreinte locale justifiée du projet** : `L_Toroid_Vertical_L28.6mm_W12.3mm_P10.00mm_Coilcraft_PA6331`, créée par le MCP et **relue au fichier par le principal**. Contrairement à `CF_Film_Box`, les graphiques imposés par `create_footprint` sont **corrects** : courtyard à ± 14,55 en X et − 1,55 à 11,55 en Y, soit 0,25 mm autour de l'élément le plus extérieur — les pastilles, non le corps — et sérigraphie à 0,15 mm hors du corps, sans recouvrir les pastilles. Ce sont les conventions KLC, et elles valent mieux que les cotes plus grossières que le principal avait commandées. **La leçon de `CF_Film_Box` se précise : le générateur n'était pas en cause, les cotes qu'on lui donnait l'étaient.** `Value` portée à `15uH/20A`, ERC toujours à 15 violations et 0 erreur. Le `NEEDS_DATA` sur l'inductance tombe. Reste à vérifier en H2 que la référence est toujours approvisionnable.)* Elles portent `Inductor_SMD:L_Wuerth_HCI-1350` **sans MPN**, alors que l'inductance 15 µH est `NEEDS_DATA` : troisième empreinte posée sur une supposition, après D1.7 et D1.9. Le corps HCI-1350 ne fait que 12,8 × 12,8 × 4,7 mm. Point de mesure lu à la source, sur la référence même que cite l'empreinte KiCad (`744355019`) : dans ce corps, la série donne 0,19 µH à `I_SAT,10%` = 60 A, soit une énergie stockée de 342 µJ. Or le cahier des charges — 15 µH avec `I_sat` ≥ 10 A — en réclame 750 µJ, plus du double. L'énergie étant d'abord une propriété du volume et du matériau de noyau, **le boîtier 1350 est très probablement trop petit** ; c'est une déduction sur un seul point de mesure, à confirmer quand la référence sera figée, mais elle suffit à ne pas traiter cette empreinte comme acquise. Prévoir un corps plus grand et son impact sur le placement en Phase E.
- [x] D1.12 Rendre `fp-lib-table` portable. *(PASS. La librairie locale y était déclarée par un **chemin absolu Windows**, `C:\Users\FlowUP\Documents\...` : un clone du dépôt sur une autre machine, ou un simple déplacement du projet, aurait cassé la résolution des deux empreintes locales sans avertissement. Remplacé par `${KIPRJMOD}/HifiAmp_TPA3255_Local.pretty`. **`sym-lib-table` utilisait déjà `${KIPRJMOD}`** : la convention du projet était donc établie et l'incohérence isolée à ce seul fichier. Que KiCad résolve bien cette variable dans ce projet est prouvé par l'ERC lui-même, qui charge le symbole local `LM5069` déclaré de la même façon. Sauvegarde de l'original conservée le temps de la session.)*
- [x] D1.3 Vérifier particulièrement HTSSOP TPA3255, PowerPAD et stratégie de vias thermiques. *(PASS, **et la tâche se retourne : il n'y a pas de vias thermiques à concevoir sous le TPA3255**. `U6` portait `HTSSOP-44-1EP_6.1x14mm_P0.635mm_EP5.2x14mm_Mask4.31x8.26mm`, **empreinte d'un autre boîtier** : elle est taguée `Texas_DDW0044B` et pose une 45ᵉ pastille de 5,2 × 14 mm **sous** le composant. Or le TPA3255 est en `DDV0044D`, et la datasheet (`SLASEA8A`, p. 3) est explicite : *« The package type contains a PowerPAD that is located **on the top side of the device** for convenient thermal coupling to the heat sink »*. Le tableau des broches ajoute, pour le PowerPAD : *« Ground, connect to grounded heat sink »*. Le plan cothé (4218830/A, p. 44) donne un pad exposé de **6,72 à 7,30 mm sur 3,85 à 4,43 mm**, soit 7,01 × 4,14 nominal — et non 5,2 × 14 — tandis que le land pattern TI (p. 45) ne montre que **44 pastilles**, sans rien à braser dessous. Corrigé en `Package_SO:HTSSOP-44_6.1x14mm_P0.635mm_TopEP4.14x7.01mm`, taguée `Texas_DDV0044D`, 44 pastilles, pad thermique dessiné sur le dessus aux cotes nominales exactes. Écriture MCP relue au fichier par le principal, ERC toujours à 15 violations et 0 erreur. **La preuve chiffrée vient du tableau thermique 7.4 : `RθJC(bot)` est donné `n/a`** — TI ne caractérise même pas la voie par le dessous — alors que `RθJA` tombe de **50,7 à 2,4 °C/W** dès qu'un dissipateur est fixé sur le dessus, facteur 21, la note précisant que *« only path for dissipation is to the heatsink »*. **Le `NEEDS_DATA` annonçant un EP de 5,2 × 14 mm à confirmer est donc infirmé, pas confirmé**, et le `NEEDS_DATA` dissipateur devient le point dimensionnant unique du thermique du TPA3255. **Écart connu et voulu à reporter en Phase E** : le symbole compte 45 broches, la broche 45 étant `EP_45` câblée à `GND` — ce qui est correct au sens de TI — alors que l'empreinte n'a que 44 pastilles. L'import PCB signalera une broche sans pastille : c'est **attendu**, la mise à la masse du PowerPAD passant par le dissipateur, hors PCB. Au passage, le tableau des conditions recommandées confirme `L_OUT(BTL)` ≥ 5 µH, que les 15 µH de D1.10 satisfont largement.)*

- [x] D1.13 Trancher la fenêtre de tension où le TPA3255 travaille hors spécification. **Relevé en D1.3, non instruit.** Le tableau des conditions recommandées distingue deux plafonds selon la charge : `PVDD` va jusqu'à **51 V typique et 53,5 V maximum sous 4 Ω**, mais jusqu'à 53,5 V typique et 56,5 V maximum sous ≥ 6 Ω, et seulement *« provided a reduced over-current threshold is set »*. Or le projet vise **4 à 8 Ω**, et le seuil de coupure haute du `LM5069` est réglé à **56,4 V** (`docs/protection-48v.md`). **Il existe donc une fenêtre de 53,5 à 56,4 V dans laquelle une charge de 4 Ω est alimentée au-dessus du maximum absolu du TPA3255 sans que la protection ne coupe.** Le raisonnement actuel — le plafond n'est pas fixé par le TPA3255 puisque `Q302` l'isole en surtension — ne couvre pas cette bande. Trois issues à peser : abaisser `V_OVH` vers 52 V, restreindre la charge admissible à ≥ 6 Ω, ou démontrer que l'alimentation externe retenue ne peut pas atteindre 53,5 V. **Dépend du `NEEDS_DATA` alimentation externe 48 V.** Arbitrage utilisateur probable. — **CLOS.** Deux rectifications d'abord : 53,5 V est une borne de *conditions recommandées* et non un maximum absolu, le maximum absolu `PVDD_X to GND` valant **69 V** (`SLASEA8A` § 7.1) et non les 65 V écrits dans `docs/protection-48v.md`, désormais corrigés ; et la bande incriminée se situe donc **12,6 V sous la destruction**, la limite franchie portant sur le courant de sortie, pas sur la tenue en tension. Ensuite, **l'issue « abaisser `V_OVH` » est arithmétiquement impossible** : garantir ≤ 53,5 V au pire cas impose un nominal ≤ 50,5 V, dont la borne basse tombe à 49,5 V — sous les 50,4 V d'un rail 48 V à +5 % — et dont la reprise, hystérésis déduite, tomberait entre 43,8 et 47,2 V, empêchant tout redémarrage ; la fenêtre à couvrir vaut ± 3 % contre ± 6 % de dispersion spécifiée du seuil. **Le `LM5069` ne peut donc pas faire respecter les conditions recommandées du TPA3255**, seulement protéger d'un défaut d'alimentation. **Issue retenue sur arbitrage utilisateur : borner l'alimentation par spécification, charge 4 Ω conservée, aucun composant modifié.** Exigence `REQ-PSU-1` inscrite dans `docs/architecture.md` : sortie ≤ 53,5 V en toutes conditions, soit 3,1 V de marge pour une alimentation régulée à ± 5 %. Résidu assumé et tracé : une panne d'alimentation dans la bande 53,5–56,4 V ne serait coupée par rien, l'`OCP` en CB3C et l'`OTW`/`OTSD` restant seuls actifs.
- [x] D1.14 Instruire les cinq `lib_symbol_mismatch` et les symboles reliquats. *(PASS. **Diagnostic livré, et il est rassurant : aucun écart de brochage.** Les deux copies de chacun des cinq symboles — celle du cache `lib_symbols` du schéma et celle du `.kicad_sym` — ont été comparées par **égalité d'arbre S-expression**, après neutralisation du seul nom racine, qui porte le préfixe de librairie dans le cache. Résultat : `LM5010ASD` 330 feuilles, `LM2940IMP_12_FIXED` 169, `TPS3802K33` 192, `LM5069` 307, `TPA3255B` 1112 — **identiques des deux côtés, feuille à feuille**. Numéros, noms, types électriques et positions de broches concordent. **Aucune vérification antérieure n'est donc remise en cause**, ce qui était la question de fond. L'avertissement ne porte sur aucune divergence de contenu entre les deux fichiers. **Piste de la version de format testée et infirmée** : le schéma est en `20250610` et la librairie en `20240108`, toutes deux écrites par `konnect` ; migrer une copie de la librairie par `kicad-cli sym upgrade --force` — KiCad y ajoute `exclude_from_sim`, `in_pos_files`, `show_name`, normalise `Datasheet` de `~` à vide, et la taille passe de 17 744 à 28 263 octets, les 6 symboles et 79 broches étant conservés — puis relancer l'ERC laisse **exactement les mêmes 15 violations**. La copie migrée a donc été abandonnée et la librairie restaurée par `git checkout`. **Conclusion : les cinq avertissements sont cosmétiques et restent dans la baseline C2**, qui demeure à 15 violations et 0 erreur. Une dernière piste existe — faire réécrire le cache du schéma lui-même par KiCad — mais elle fait passer un outil de migration sur le cœur du projet, dont la connectivité repose sur la coïncidence label/ancre, pour un gain purement cosmétique : **non retenue sans arbitrage utilisateur**. **Reliquats confirmés inutilisés** : `TPA3255DDV`, `TPA3255DDV_TEST` et `TPA3255` dans le cache, `LM2940IMP-12` dans la librairie, **zéro `lib_id` les référençant**, quand les cinq symboles utiles ont exactement une instance chacun. Ils ne sont pas supprimés : retirer des entrées du cache demanderait d'éditer directement le `.kicad_sch`, et KiCad les nettoie de lui-même à la première sauvegarde depuis l'interface.)* **Relevé en D1.3.** Les cinq avertissements sont un tiers de la baseline C2 et portent tous sur des symboles de la librairie locale : `LM5010ASD`, `LM2940IMP_12_FIXED`, `TPS3802K33`, `LM5069`, `TPA3255B`. Un `lib_symbol_mismatch` signifie que la copie embarquée dans le schéma diverge de la librairie ; **l'écart peut être anodin, un champ de propriété, ou grave, un brochage** — ce n'est pas tranché, donc la baseline repose sur cinq inconnues. À diagnostiquer symbole par symbole, en comparant broche à broche. Deux reliquats à traiter dans le même mouvement : le cache `lib_symbols` du schéma contient encore `TPA3255DDV`, `TPA3255DDV_TEST` — deux broches, manifestement un essai — et `TPA3255`, **qui n'existent plus dans le `.kicad_sym`** et que plus aucun composant n'utilise ; et la librairie garde `LM2940IMP-12` alors que `U2` est passé à `LM2940IMP_12_FIXED`. Ne rien supprimer avant d'avoir montré qu'aucun symbole n'y renvoie.

### Validation

Toutes les empreintes sont attribuées et revues contre leurs sources fabricant.

# Phase E — PCB 4 couches

## E1 — Définir stack-up, règles et placement

### Dépendances

D1 validée.

### Tâches

- [x] E1.1 Créer le PCB 4 couches et documenter stack-up/règles/classes de nets. *(PASS. Quatre couches cuivre — `F.Cu` signal, `In1.Cu` power, `In2.Cu` mixed, `B.Cu` signal — sur une carte de 1,6 mm. Huit classes de nets et **71 affectations**, sans doublon, dimensionnées sur IPC-2221 à 35 µm et 10 °C d'échauffement. Règles globales conservatrices, le fabricant n'étant pas choisi : 0,20 mm de piste et d'isolation, via ø0,60, perçage 0,30, anneau 0,15. **Contour de carte délibérément non tracé** : il dépend du dissipateur et du boîtier, encore en `NEEDS_DATA`. **Vérifié par le principal au fichier** : les quatre couches sont là, le bloc `setup` ne contient que `pad_to_mask_clearance`, zéro empreinte, `Edge.Cuts` vide, et `kicad-cli pcb drc` charge le fichier en ne remontant que `invalid_outline` — l'absence de contour, donc la condition voulue. Détail des classes : `docs/architecture.md`, section « Règles et classes de nets posées en E1.1 ».)*
  - **Incident MCP, consigné dans `reports/MCP_BUG-setup-tokens-kicad-pcb.md`.** `set_design_rules` écrit `min_clearance`, `min_track_width`, `min_via_drill` et `min_via_size` dans le bloc `(setup ...)` du `.kicad_pcb`, où ces jetons **n'existent pas** dans le format ; `set_active_layer` y ajoute `active_layer`, tout aussi invalide. Le fichier devient illisible par KiCad — *« Inattendu active_layer en ligne 32 »* — **sans qu'aucun retour d'outil ne signale l'échec**, et aucun des 203 outils du MCP ne sait nettoyer le bloc. Récupéré par `git checkout` du seul `.kicad_pcb`, sans risque puisqu'il était vide de toute connectivité. **Ces deux outils sont désormais proscrits sur ce projet** ; les règles globales se posent dans le `.kicad_pro`, sous `board.design_settings.rules`, où `set_design_rules` n'avait appliqué que deux des quatre valeurs demandées.
  - **Réserve reportée** : aucun outil MCP n'expose l'épaisseur de cuivre par couche. Le `.kicad_pcb` ne porte pas de bloc `stackup` explicite et KiCad applique son défaut. À rendre explicite dans Board Setup et sur la commande au fabricant avant les livrables de fabrication.
- [ ] E1.2 Placer puissance Class-D, bootstrap/découplages/bulk et thermique.
  - **Débloquée.** Les trois arbitrages de E1.7 sont rendus et consignés dans `docs/architecture.md`, section « Arbitrages rendus, et contour qui en découle » : coffret Modushop `03/300` 3U, carte à plat avec `U6` en bord de carte relié au flanc par une barre aluminium courte et massive, isolation électrique reportée à la jonction barre/flanc. Contour arrêté à **200 × 150 mm**.
  - **Étape 1 franchie — contour et import.** `Edge.Cuts` porte le rectangle 200 × 150 mm, coins `(100,100)`–`(300,250)`. Les **122 empreintes** sont importées avec leur connectivité : 122 références distinctes, aucun doublon, **74 nets**, 327 pastilles sur 332 portant un net. `RV1` reste volontairement absente, faute d'empreinte assignée. **Vérifié au fichier par le principal** et par `kicad-cli pcb drc` : `schematic_parity` vaut **0** — la carte est le miroir exact du schéma — et `invalid_outline` a disparu. Les 391 violations et 253 non-connectés restants sont l'état normal d'avant placement : empreintes empilées au point d'import, aucune piste tracée.
  - **Étape 2 franchie — bloc de puissance placé.** 21 empreintes posées aux coordonnées imposées : `U6` en `(285, 175)` à **rotation 180°**, ses quatre bootstrap `C306`–`C309`, les deux découplages `PVDD` `C310`/`C311` en 1210, les huit découplages du côté bas niveau `C301`–`C305`/`C318`–`C320`, et le bulk `C312`–`C317`. **Vérifié au fichier par le principal**, indépendamment du rapport d'agent : l'empreinte MD5 du `.kicad_pcb` a changé — donc l'enregistrement a bien eu lieu —, les 21 positions et angles relus sont **exacts au micron**, les 101 autres empreintes n'ont pas bougé d'un pas, le total reste à 122 sans doublon et le contour est intact. `kicad-cli pcb drc` : les violations tombent de **391 à 137**, `schematic_parity` reste à **0**, et les 253 non-connectés sont inchangés puisque rien n'est routé.
  - **Choix de rotation, et sa raison.** Les deux flancs du `HTSSOP-44` ne sont pas interchangeables : côté `x < 0` du symbole se trouve tout le bas niveau, côté `x > 0` toute la puissance — six `PVDD`, `OUT_A`–`D`, quatre `BST`, six `GND`. La rotation 180° tourne donc la puissance vers l'intérieur de la carte, où vivent le bulk et le futur filtre LC, et laisse le bas niveau échapper vers la lisière droite. Conséquence mécanique tirée au passage : **la barre de liaison est dressée**, 10 mm d'épaisseur dans le plan de la carte pour 60 de hauteur, ce qui préserve les 600 mm² de section mais réduit son ombre sur la carte à une bande de 10 mm sans aucun site de composant. Détail et réserve dans `docs/architecture.md`, section « E1.2 — le placement retourne la section de la barre ».
  - **Reste à faire pour clore E1.2** : les deux perçages M3 de fixation de la barre, de part et d'autre de `U6`. **Volontairement différés après E1.8** : ce sont des empreintes sans symbole au schéma, et la resynchronisation de E1.8 les effacerait si l'option « supprimer les empreintes sans symbole » restait cochée.
  - **Découverte d'outillage, à porter en H2.** **Aucun des 203 outils du MCP ne sait synchroniser le schéma vers le PCB** : ni import de netlist, ni « update PCB from schematic ». `kicad-cli` ne l'expose pas davantage. De plus, les outils MCP dépendants de l'IPC exigent non seulement que KiCad tourne, mais que **l'éditeur de PCB ait le fichier ouvert** — le gestionnaire de projet seul renvoie `KiCad does not handle kiapi.common.commands.GetOpenDocuments for this document type`. La synchronisation a donc dû passer par l'action **Outils → « Mise à jour du PCB à partir du Schéma »** de l'éditeur de PCB, **ouvert depuis le gestionnaire de projet** : ouvert en autonome, KiCad la refuse.
- [ ] E1.3 Placer filtres LC, sorties, alimentation et boucles de retour.
- [ ] E1.4 Placer analogique faible bruit, volume et contrôles avec séparation fonctionnelle.
- [ ] E1.5 Revoir symétrie, retour des courants, masses, clearances et manufacturabilité.
- [x] E1.6 Chiffrer l'exigence thermique du dissipateur de `U6`, pour transformer le `NEEDS_DATA` en critère d'achat. **Ajouté en cours de Phase E**, le dissipateur étant devenu le point dimensionnant unique après D1.3. *(PASS. Dissipation de `U6` lue sur la **figure 10 de `SLASEA8A`** par extraction vectorielle, tracé contrôlé à l'échelle sur deux paires de graduations par axe : **22,4 W** à 2 × 100 W sur 8 Ω, **37,4 W** sur 4 Ω. **Recoupement fort** : les 22,4 W donnent 89,9 % de rendement, soit exactement les 90 % que le projet postulait sans preuve depuis l'origine pour établir les 4,6 A du rail. **Découverte principale : le point dur n'est pas le dissipateur mais l'interface.** Le PowerPAD ne fait que **29,02 mm²**, si bien qu'un pad silicone standard vaudrait **8,61 °C/W** — à lui seul plus que la totalité du budget. D'où deux exigences inscrites dans `docs/architecture.md` : **`REQ-THERM-1`, `RθSA` ≤ 1,0 °C/W** en convection naturelle pour l'usage nominal 8 Ω à 40 °C d'ambiante, et **`REQ-THERM-2`, interface ≤ 0,6 °C/W**, pad silicone standard explicitement exclu. Critère retenu : `T_C` ≤ 75 °C, plus strict que le seuil d'`OTW` à 125 °C, parce qu'il préserve la validité de toutes les courbes TI utilisées pour dimensionner la carte. **Réserve portée à l'utilisateur** : le continu à pleine puissance sur 4 Ω exigerait `RθSA` ≤ 0,37 °C/W, hors d'atteinte en convection naturelle — limite physique, non défaut de conception.)*
  - **Ne lève pas le `NEEDS_DATA`, et ne débloque pas E1.2.** Le placement réclame les **cotes** du modèle retenu et celles du boîtier, que seul l'utilisateur peut fournir. E1.6 produit le critère de sélection, pas la pièce.
- [x] E1.7 Établir la short-list dissipateur + boîtier contre `REQ-THERM-1`/`REQ-THERM-2`, et en relever les cotes. *(PASS. **Trois découvertes, dont deux rectifient E1.6.** (1) Le dissipateur de l'EVM, `ATS-TI1OP-519-C1-R3`, **ne tient pas l'exigence** : sa fiche `qats.com` ne publie aucune valeur à vitesse d'air nulle et plafonne à 2,2 °C/W sous 1 m/s, ce qui donnerait `T_C` = 102 °C. Il en reste la **mécanique de fixation** : deux taraudages M3, entraxe 36,8 mm, boulonnage par le dessous à travers le PCB. (2) **La résistance d'étalement manquait au budget** : les 22,4 W entrent par 29 mm², ce qui coûte au mieux 0,411 °C/W en base aluminium et ramène la cible d'un `RθSA` de catalogue de 1,0 à **0,58 °C/W**. `REQ-THERM-1` est reformulé sous forme invariante : interface + étalement + dissipateur ≤ **1,5625 °C/W**. (3) **La solution est le coffret, pas un dissipateur rapporté.** Un flanc de coffret dissipe vers les 25 °C de la pièce et non vers les 40 °C internes, ce qui vaut 0,67 °C/W de budget. Données fabricant Modushop/HiFi 2000 *Pesante Dissipante* : 0,45 °C/W par flanc en 2U/300 jusqu'à 0,23 en 4U/400, largeur intérieure 360 mm, hauteurs 80/120/165/210 mm. **Même le plus petit modèle donne `T_C` = 57 °C contre 75 visés**, et laisse 0,80 °C/W pour la barre de liaison — assez pour 50 mm en section 10 × 60, pas pour une équerre mince. Détail, budgets et réserves : `docs/architecture.md`, section « E1.7 — la solution n'est pas un dissipateur, c'est le coffret ».)*
  - **Fischer Elektronik non vérifiable** : 403 sur toutes les pages produit même avec en-tête navigateur, miroirs distributeurs inexploitables. Le `SK 47/100/SA` n'est **pas** retenu, faute de source primaire.
  - **Trois arbitrages utilisateur restent ouverts et conditionnent E1.2** : modèle de coffret, architecture mécanique (carte à plat + barre courte, ou carte verticale contre le flanc), et traitement de la masse — le PowerPAD étant `GND`, le presser sur le châssis relie la masse signal à la terre. **Ajouté en cours de Phase E** sur arbitrage utilisateur : plutôt que de fournir les cotes, l'utilisateur demande que le projet propose les candidats. C'est la tâche qui débloque E1.2.
  - **`REQ-THERM-3` acquis d'emblée** : le 4 Ω passe en régime de crête, la cible reste donc `RθSA` ≤ 1,0 °C/W en convection naturelle et non 0,37. Inscrit dans `docs/architecture.md`.
  - Contrainte de forme dominante : le PowerPAD étant **sur le dessus**, le dissipateur se pose à plat au-dessus de `U6`, à l'intérieur du boîtier. Un flanc dissipant ne conviendrait qu'avec un caloduc. La hauteur libre du boîtier doit donc couvrir le PCB, les 35 mm des condensateurs bulk **et** la hauteur du dissipateur.
  - Cotes à relever **par extraction du PDF fabricant**, jamais par listing distributeur : hors-tout du dissipateur, entraxe et mode de fixation, dimensions **intérieures** du boîtier.
  - Validation : chaque candidat retenu porte un `RθSA` **lu chez le fabricant en convection naturelle**, avec sa condition de mesure, et des cotes traçables à un document identifié.
- [x] E1.8 Raccorder le tab de `U3` à sa sortie au schéma. *(PASS. Un symbole local `TLV1117_33_FIXED` a été créé dans `HifiAmp_TPA3255.kicad_sym`, copie du `Regulator_Linear:TLV1117-33` augmentée d'une **broche 4 `TAB`, type `passive`, superposée exactement sur la broche 2 `VO`** — mêmes `(at 7.62 0 180)` et `(length 2.54)`. `U3` y est repointé sans bouger : `(at 195.58 100.33 0)`, UUID, `Reference`, `Value` et `Footprint` inchangés. **Vérifié au fichier par le principal** : `pad 4` de `U3` porte `/+3V3`, et surtout **plus aucune pastille numérotée de la carte n'est sans équipotentielle**. Les 21 placements de E1.2 sont intacts au micron, 122 empreintes, 74 nets, contour intact, `schematic_parity` toujours à **0**. Les non-connectés passent de 253 à **254** : exactement la pastille nouvellement raccordée et pas encore routée — la preuve arithmétique que le raccordement a pris.)*
  - **Superposition plutôt qu'ancre distincte, et c'est délibéré.** Le précédent `LM2940IMP_12_FIXED` pose sa broche `TAB` sur une ancre séparée, raccordée par une étiquette placée exactement dessus. Ici le tab est électriquement **le même nœud que la sortie** : le superposer sur la broche 2 le raccorde sans créer ni fil ni étiquette, donc **sans calculer une coordonnée d'ancre** — la seule partie réellement faillible de l'opération, et celle où la convention KiCad s'était révélée non triviale à dériver. Le type `passive` évite le conflit `power_out` contre `power_out` à l'ERC.
  - **Coût assumé, à inscrire à la baseline** : l'ERC passe de 15 à **16 violations, toujours 0 erreur**. L'écart est **un `lib_symbol_mismatch` de plus**, qui rejoint les **cinq déjà présents** pour `LM5010ASD`, `LM2940IMP_12_FIXED`, `TPS3802K33`, `LM5069` et `TPA3255B` — un simple écart de formatage entre la copie en cache du `.kicad_sch` et celle de la librairie, sans incidence électrique. Les 10 `endpoint_off_grid` sont inchangés, aucune violation n'a disparu. **La baseline `GATE C2` devient donc 16 violations / 0 erreur.**
  - **Preuve du défaut.** Après import, `U3` est le seul composant dont une pastille numérotée reste sans équipotentielle : `pad 4` du SOT-223, c'est-à-dire le **tab**. Le rapport KiCad le dit explicitement — *« Pas d'équipotentielle pour composant 'U3' pad '4' (pas de pin 4 en symbole) »*. Le symbole `TLV1117-33IDCY` ne déclare que trois broches.
  - **Fait établi à la source primaire.** `SLVS561O`, figure 4-1 *« DCY Package, 4-Pin SOT (Top View) »* : le boîtier **DCY est un SOT-223 à quatre broches**, la quatrième — le tab — étant **`OUTPUT`**, donc le même nœud que la broche 2. Le tab de `U3` doit porter `/+3V3`.
  - **Contre-exemple interne qui confirme l'omission** : `U2`, `LM2940IMP-12` dans le **même boîtier SOT-223**, a bien son `pad 4` raccordé à `/+12V`, sa propre sortie. L'asymétrie est un oubli de symbole, pas un choix.
  - **Conséquences si on n'y touche pas** : le tab est le chemin thermique principal du régulateur, et il serait laissé flottant ; le cuivre sous `U3` deviendrait un îlot isolé, relié à `/+3V3` par le seul boîtier.
  - **Non bloquant pour le placement**, bloquant pour F1 : à corriger avant tout routage.
  - **Voie de correction, avec précédent dans le projet.** `U3` tire le symbole **standard** `Regulator_Linear:TLV1117-33`, à trois broches. `U2` en revanche tire un symbole **local**, `LM2940IMP_12_FIXED` de `HifiAmp_TPA3255.kicad_sym`, et c'est précisément pour cela qu'il a son `pad 4`. Le correctif consiste donc à refaire le même geste : créer un `TLV1117_33_FIXED` local portant la quatrième broche, puis y repointer `U3`. Aucune invention de méthode, seulement l'application d'un précédent déjà validé sur ce projet.
  - Validation : `pad 4` de `U3` porte `/+3V3` après resynchronisation, `schematic_parity` reste à 0, et l'ERC ne régresse pas — 15 violations, 0 erreur, jeu identique à la baseline `GATE C2`.
- [ ] E1.9 Poser une règle DRC pour l'isolation pastille-à-pastille interne aux boîtiers à pas fin. **Ajouté en cours de Phase E**, révélé par le DRC d'après placement.
  - **Preuve.** Les **42 violations `clearance`** du DRC ne sont pas des collisions de placement : elles opposent toutes **deux pastilles d'un même boîtier**, et se répartissent en `U6` (26), `U1` (8) et `U8` (8). Message type : *« Violation d'isolation (netclasse `PWR_AUX` isolation 0,2500 mm ; réel 0,2350 mm) »*. Leur nombre est **identique avant et après placement**, ce qui démontre qu'aucune translation ni rotation ne peut les faire disparaître.
  - **Cause.** L'écart est imposé par le pas des boîtiers retenus : `HTSSOP-44` à 0,635 mm de pas laisse 0,235 mm entre pastilles, `WSON-10` et `MSOP-10` sont à 0,5 mm. Les isolations de classes de nets posées en E1.1 — 0,25 mm en général, 0,50 mm sur les classes de puissance — sont donc **plus exigeantes que ce que la géométrie des composants permet**.
  - **Ce n'est pas un défaut de fabricabilité** : 0,235 mm reste très au-dessus des capacités courantes d'un fabricant de circuits imprimés. C'est la règle qui est mal cadrée, pas la carte.
  - Voie retenue : une règle personnalisée limitée aux pastilles **appartenant à la même empreinte**, dans `board.design_settings.rules` du `.kicad_pro` — jamais par `set_design_rules`, proscrit. Ne pas abaisser l'isolation générale de la carte pour autant.
  - Validation : les 42 violations `clearance` disparaissent, aucune nouvelle n'apparaît, et l'isolation entre empreintes distinctes reste inchangée.

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
