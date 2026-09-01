# PROGRESS

## Phase actuelle

Phase D. **GATE C2 = PASS**, revérifié après chaque écriture : ERC à 15 warnings et 0 erreur, jeu de messages identique à la baseline C2.

## Tâche actuelle

D1.2 — Vérifier boîtiers fabricant, orientations, courants et contraintes d'assemblage.

## Dernière tâche validée

**D1.7 = PASS.** La réserve sur `C325` était fondée deux fois : la cote supposée était fausse, et la référence aussi.

- Au pas de 5 mm, la cote constante de la série WIMA MKS2 est la **longueur** `L` = 7,2 mm, jamais l'épaisseur. Le 4,7 µF mesure **11 × 18 × 7,2 mm**. L'empreinte était trop étroite de 3,8 mm.
- Clé de référence WIMA : **champs 11-12 = boîte et pas, champ 15 seul = tolérance.** Le `O` de `MKS2C044701O00KSSD` est le second caractère du code de boîte `1O`. La référence écartée en D1.5 comme inventée était donc la bonne ; sa remplaçante `MKS2B044701K00KSSD` est celle qui n'existe pas (`B0` n'est pas un code de tension, la MKS2 n'a aucun calibre 50 V ; boîte `1K` = 7,2 × 13 mm, qui ne loge que 2,2 µF).
- `C325` : MPN `MKS2C044701O00KSSD`, Value `4.7uF/63V`, empreinte `Capacitor_THT:C_Rect_L7.2mm_W11.0mm_P5.00mm_FKS2_FKP2_MKS2_MKP2` (librairie standard). Écriture MCP **relue au fichier par le principal**, position inchangée, ERC inchangé. Marge de démarrage × 1,41 conservée.

**Leçon transposable, à appliquer à tout passif restant : une référence ne se valide pas sur son aspect, mais en la décodant champ par champ contre la clé du fabricant, puis en recoupant la boîte obtenue avec le tableau de la valeur visée.**

## Défauts ouverts

- **D1.9 — `C321` à `C324` portent une empreinte impossible.** Même mode de défaillance que D1.7 : 3,5 mm d'épaisseur supposée, jamais lue. Le filtre étant référencé à `GND` en aval de chaque demi-pont, il voit tout le rail et TI impose un calibre 100 V ; or sous 100 V une boîte de 3,5 mm au pas de 5 mm ne loge que 0,15 à 0,22 µF, et le 680 nF mesure 5 × 10 × 7,2 mm. Ne pas re-supposer : la référence est `NEEDS_DATA`, et le diélectrique est à trancher d'abord (MKP polypropylène attendu sur un filtre Class-D, pas MKS polyester), ce qui peut déplacer le pas.
- **D1.8 déverrouillé, plus aucun arbitrage requis.** `CF_Film_Box_P5.00mm_7.2x3.5mm` était **redondante dès sa création** : la librairie standard contient `C_Rect_L7.2mm_W3.5mm_P5.00mm_FKS2_FKP2_MKS2_MKP2`, courtyard correct de 7,7 × 4,0 mm contre 7,6 × 2,6 mm au local. Son courtyard cassé n'est donc pas à corriger, mais à abandonner ; suppression à la fermeture de D1.9. Reste seulement le fusible : `Fuse_Schurter_UMT-H_5.3x16mm` demeure justifiée (le standard `Fuse_Schurter_UMT250` vise un corps 3 × 10,1 mm, pastilles à ± 4,25 contre ± 6,875 mm), et son défaut est **purement cosmétique**, sans effet DRC ni fabrication.
- **Risque de même classe non encore instruit** : `L301` à `L304` portent `L_Wuerth_HCI-1350` sans MPN, alors que l'inductance 15 µH est `NEEDS_DATA`. L'empreinte est donc, elle aussi, une supposition.

## Blocage actif

Aucun.

## Contraintes portées en Phase E

- `C325` culmine à **18 mm**, contre 13 mm pour la boîte supposée : composant film le plus encombrant, à croiser avec le dégagement sous capot.
- **`R306` est un shunt à deux bornes, pas Kelvin.** Les liaisons vers `VIN` et `SENSE` doivent partir des **bords intérieurs** des pastilles, le courant de puissance entrant par les bords extérieurs. Repli : `WSK25122L000FEA`, quatre bornes, même boîtier 2512.
- `Q302` doit être monté sur radiateur : sa SOA suppose le boîtier à 75 °C.
- `Q301` : courant continu plafonné à 6,9 A avec la surface de cuivre de référence de sa datasheet, 6 cm² en 70 µm.
- Rail d'entrée : les calibres UMT-H supposent des pistes de 7,5 mm en cuivre 140 µm. Déclassement sinon.
- `C110`/`C210` imposent un établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec la temporisation de mute en Phase F.
- Connectique déportée en JST XH : deuxième famille à approvisionner à côté des MaiXu MX126-5.0, et pince à sertir nécessaire.

## NEEDS_DATA ouverts

`RV1` (mécanique du potentiomètre), EP du TPA3255DDV à 5,2 × 14 mm à confirmer sur dessin mécanique TI, alimentation externe 48 V, **inductances 15 µH et films 680 nF (bloque D1.9)**, dissipateur et thermique, réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33.

Assumés : stabilité de la boucle de limitation de puissance du LM5069 face aux 540 nC de grille de `Q302` ; `V_C` de la `SMDJ58CA` sous `I_PP`.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.** Confirmé trois fois : une référence inventée passée dans le fichier, une assignation rapportée mais non faite, et une référence saine écartée à tort.
- **`kicad-cli.exe` est utilisable directement** (`sch erc`, `sch export netlist`) et fournit une preuve indépendante du MCP, sans GUI. Chemin : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **Avant de créer une empreinte locale, épuiser la librairie standard.** Sur les deux locales créées jusqu'ici, une seule était nécessaire.
- **Lire les cotes par extraction du PDF fabricant, pas par recherche web ni par listing distributeur.** `pymupdf` installé, `pdftotext` dans `/mingw64/bin`. **Toujours contrôler si un tracé est à l'échelle** : celui de Schurter ne l'est pas.
- Les serveurs Littelfuse et DigiKey refusent les requêtes automatisées ; le serveur Infineon répond. L'identité de tout PDF récupéré doit être contrôlée sur son en-tête.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- TVS cantonnée aux transitoires rapides ; la protection en surtension est active, par `LM5069`.
- Bulk maintenu à 15 400 µF sur arbitrage utilisateur ; `Q302` choisi en conséquence.
- Connectique déportée : JST XH 2,5 mm, vertical par défaut, sur arbitrage utilisateur.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.
- Instantanés PDF automatiques du MCP (`*_pre_delete_*.pdf`) exclus par `.gitignore`.
- Réserve consignée non tranchée : à 100 V un P-canal reste environ trois fois moins bon qu'un N-canal ; l'alternative serait un contrôleur de diode idéale (`LM74700`, `LM5050`) pilotant un N-canal. Non retenue, la topologie P-MOS est tranchée et 0,70 W est acceptable.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Les écritures sont fichier et persistées ; prouver par relecture.
- Attributs `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées.
- **`edit_schematic_component` accepte `uuid` en plus de `reference`** : seul moyen d'adresser un symbole quand plusieurs partagent le même repère.
- **`create_footprint` impose ses propres graphiques** et aucun outil ne les édite après coup : `edit_footprint_pad` ne touche que les pastilles, les toolsets `pcb_*` n'opèrent que sur un `.kicad_pcb`.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Une recherche naïve de `(property "Reference" ...)` suivie d'une fenêtre de caractères déborde sur le symbole voisin. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`, `.kicad_sym` (symbole `LM5069` local), `sym-lib-table`, `fp-lib-table`
- `HifiAmp_TPA3255_Local.pretty/` : `Fuse_Schurter_UMT-H_5.3x16mm` (justifiée), `CF_Film_Box_P5.00mm_7.2x3.5mm` (à supprimer, D1.9)
- `docs/architecture.md`, `docs/power-block.md`, `docs/protection-48v.md`
- Librairies KiCad : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/share/kicad/footprints`

## NEXT ACTION

D1.2 — poursuivre la vérification des boîtiers sur les postes non encore instruits, en appliquant la méthode de décodage validée en D1.7 : `L301` à `L304` (empreinte `HCI-1350` posée sans MPN), les bulks `C312` à `C315` (`CP_Radial_D16.0mm_P7.50mm`) et `C316`/`C317` (`CP_Radial_D35.0mm_P10.00mm_SnapIn`), dont la hauteur de boîte conditionne le placement, puis l'orientation de `Q301` (TO-220-3) et `Q302` (TO-264-3), retenue verticale par défaut et dépendante du radiateur. `IPP330P10NM` est déjà confirmée réelle, datasheet servie par le serveur Infineon.
