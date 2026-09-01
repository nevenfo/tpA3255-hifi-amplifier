# PROGRESS

## Phase actuelle

Phase D. **GATE C2 = PASS**, revérifié après chaque écriture : ERC à 15 violations et 0 erreur, jeu identique à la baseline C2 (10 `endpoint_off_grid`, 5 `lib_symbol_mismatch`).

## Tâche actuelle

D1.10 — Requalifier l'empreinte des quatre inductances de sortie `L301` à `L304`.

## Dernière tâche validée

**D1.9 = PASS.** Sur `C321` à `C324`, **le pas de 5 mm était faux autant que l'épaisseur**.

- La BOM du `TPA3255EVM` (`SLOU441`, p. 14) tranche le diélectrique : l'EVM équipe ce poste d'un `PHE426HB7100JR06` KEMET, **film polypropylène**, 1 µF/250 V au pas de 17,5 mm.
- Le catalogue WIMA `MKP4`, extrait et lu, **contredit l'idée qu'un MKP n'existerait pas sous 250 V** : la série se catalogue dès 100 VDC. Pour 0,68 µF elle donne une boîte `4F` de **8 × 15 × 18 mm au pas de 15 mm, identique en 100 et en 250 VDC**. Le calibre supérieur étant sans coût mécanique, **250 V retenu**.
- Empreinte : `Capacitor_THT:C_Rect_L18.0mm_W8.0mm_P15.00mm_FKS3_FKP3`, librairie standard, description renvoyant elle-même au catalogue WIMA. `Value` passée à `680nF/250V MKP`. Écriture MCP relue au fichier, positions inchangées, ERC inchangé.
- **Le 680 nF du projet, face au 1 µF de l'EVM, est confirmé sain** : avec 15 µH il donne 49,8 kHz de coupure contre 50,3 kHz pour le couple 10 µH / 1 µF, pour une impédance caractéristique de 4,7 Ω au lieu de 3,16 — mieux adaptée à 4-8 Ω.
- `CF_Film_Box_P5.00mm_7.2x3.5mm` n'ayant plus d'utilisateur, **elle est supprimée**. La librairie locale ne garde que le fusible, son entrée `fp-lib-table` reste justifiée.

## Défauts ouverts

- **D1.10 — `L301` à `L304` portent `Inductor_SMD:L_Wuerth_HCI-1350` sans MPN**, alors que l'inductance 15 µH est `NEEDS_DATA`. Le boîtier 1350 (12,8 × 12,8 × 4,7 mm) est trop petit : 342 µJ stockés contre 750 µJ exigés. **Piste forte trouvée et déjà lue à la source** : la BOM de l'EVM donne `MA5172-AE` Coilcraft pour ce poste, et sa datasheet (Coilcraft document 943, `https://www.coilcraft.com/pdfs/ma5172.pdf`, déjà téléchargée) catalogue dans la même famille **`PA6331-AE` : 15 µH, DCR 31 mΩ, `I_sat` 20 A, `I_rms` 9,8 A à 20 °C d'échauffement et 14,2 A à 40 °C** — soit exactement le cahier des charges, sur une pièce conçue pour les étages Class-D TI. **C'est un composant traversant**, corps 28,6 × 12,3 mm : aucune empreinte standard ne conviendra, une empreinte locale sera nécessaire et devra être justifiée. Reste à lire précisément l'entraxe des broches, leur diamètre et la hauteur sur le dessin coté.
- **D1.8, reliquat cosmétique.** Le volet film est clos ; ne reste que les graphiques de `Fuse_Schurter_UMT-H_5.3x16mm`, sans effet DRC ni fabrication. L'empreinte locale reste justifiée : le standard `Fuse_Schurter_UMT250` vise un corps 3 × 10,1 mm, pastilles à ± 4,25 contre ± 6,875 mm.
- **`J1`, bornier MaiXu MX126-5.0 : calibre en courant toujours non instruit.** 4,6 A continus et 10 A transitoires attendus. Sa seule datasheet connue est celle que cite l'empreinte KiCad, hébergée par LCSC, **et LCSC refuse `curl`**.

## Blocage actif

Aucun.

## Contraintes portées en Phase E

- **Films de sortie : la boîte passe de 7,2 × 3,5 à 18 × 8 mm sur 15 de haut**, courtyard de 18,5 × 8,5 mm, quatre fois. C'est le plus gros changement d'encombrement de la phase.
- **Bulk : `C312` à `C315` gagnent 2 mm de courtyard chacun** (16,16 → 18,16 mm), hauteur nominale 35 mm. `C316`/`C317` : ø35 mais **30 mm de haut seulement**, et non les 50 mm que suggère la description de l'empreinte KiCad.
- `C325` culmine à **18 mm**.
- **`R306` est un shunt à deux bornes, pas Kelvin.** Les liaisons vers `VIN` et `SENSE` doivent partir des **bords intérieurs** des pastilles, le courant de puissance entrant par les bords extérieurs. Repli : `WSK25122L000FEA`, quatre bornes, même boîtier 2512.
- `Q302` doit être monté sur radiateur : sa SOA suppose le boîtier à 75 °C.
- `Q301` : courant continu plafonné à 6,9 A avec la surface de cuivre de référence de sa datasheet, 6 cm² en 70 µm.
- Rail d'entrée : les calibres UMT-H supposent des pistes de 7,5 mm en cuivre 140 µm. Déclassement sinon.
- `C110`/`C210` imposent un établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec la temporisation de mute en Phase F.
- Connectique déportée en JST XH : deuxième famille à approvisionner à côté des MaiXu MX126-5.0, et pince à sertir nécessaire.

## NEEDS_DATA ouverts

`RV1` (mécanique du potentiomètre), EP du TPA3255DDV à 5,2 × 14 mm à confirmer sur dessin mécanique TI, alimentation externe 48 V, **inductances 15 µH (bloque D1.10, candidat sérieux `PA6331-AE`)**, dissipateur et thermique, réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33, calibre en courant du bornier `J1`.

Candidats d'ancrage **non inscrits au schéma** tant que la clé fabricant n'est pas décodée jusqu'au bout : `EEU-FC1J152` (Panasonic, ø18) pour `C312` à `C315` ; `SLPX472M080H3P3` (CDE) pour `C316`/`C317` ; `MKP4F036804F00` + quatre caractères de tolérance et conditionnement pour `C321` à `C324`.

Assumés : stabilité de la boucle de limitation de puissance du LM5069 face aux 540 nC de grille de `Q302` ; `V_C` de la `SMDJ58CA` sous `I_PP` ; pas de 7,5 mm du boîtier ø18, inchangé par la correction mais non relu chez Panasonic.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.**
- **`kicad-cli.exe` est utilisable directement** (`sch erc`, `sch export netlist`) et fournit une preuve indépendante du MCP, sans GUI. Chemin : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **La nomenclature du kit d'évaluation qui sert d'ancre au design est la première source à ouvrir** pour tout poste dont la référence manque : elle donne le boîtier réel et le diélectrique, et sert de point d'entrée pour décoder la clé du fabricant. Elle a résolu D1.9, D1.11 et débloqué D1.10 en une seule lecture.
- **Avant de créer une empreinte locale, épuiser la librairie standard.** Sur les deux locales créées jusqu'ici, une seule était nécessaire.
- **Lire les cotes par extraction du PDF fabricant, pas par recherche web ni par listing distributeur.** `pymupdf` installé, `pdftotext` dans `/mingw64/bin`. **Toujours contrôler si un tracé est à l'échelle** : celui de Schurter ne l'est pas. **Et toujours vérifier à quelle figure appartient une cote** : le « ø2 ± 0,1 » des snap-in est une cote de perçage, pas de broche.
- **Une référence ne se valide pas sur son aspect, mais en la décodant champ par champ contre la clé du fabricant, puis en recoupant la boîte obtenue avec le tableau de la valeur visée.**
- **Serveurs qui servent le PDF à `curl`** : ti.com/lit, vishay.com/docs, content.kemet.com, cde.com, coilcraft.com/pdfs, wima.de, tdk-electronics.tdk.com, Infineon. **Serveurs qui refusent ou ne répondent pas** : Littelfuse, DigiKey, LCSC, nichicon.co.jp, rubycon.co.jp, industrial.panasonic.com (timeout complet), coilcraft.com hors `/pdfs` (403).
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
- **`create_footprint` impose ses propres graphiques** et aucun outil ne les édite après coup : point dur pour D1.10, qui exigera une empreinte locale.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb` (**vide, aucune empreinte placée** : les changements d'empreinte n'imposent aucune resynchronisation), `.kicad_sym`, `sym-lib-table`, `fp-lib-table`
- `HifiAmp_TPA3255_Local.pretty/` : ne contient plus que `Fuse_Schurter_UMT-H_5.3x16mm`
- `docs/architecture.md`, `docs/power-block.md`, `docs/protection-48v.md`
- Librairies KiCad : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/share/kicad/footprints`
- PDF déjà téléchargés, dans le scratchpad de session : `slou441.pdf` (BOM EVM TPA3255, p. 14-15), `ma5172.pdf` (Coilcraft doc 943), `e_WIMA_MKP_4.pdf`, `SLP.pdf` (CDE), `058059pll-si.pdf` (Vishay), `KEM_A4082_ALC80.pdf`.

## NEXT ACTION

D1.10 — sur le dessin coté de `ma5172.pdf` (Coilcraft doc 943, déjà dans le scratchpad), relever pour `PA6331-AE` l'entraxe des broches, leur diamètre et la hauteur du corps, en contrôlant d'abord si le tracé est à l'échelle. Vérifier ensuite qu'aucune empreinte de la librairie standard ne couvre cette géométrie ; si aucune ne convient, créer l'empreinte locale via le MCP en sachant que `create_footprint` impose ses graphiques et qu'aucun outil ne les édite après coup. Assigner `L301` à `L304`, porter `Value` à `15uH` avec le calibre en courant, relire au fichier, revérifier l'ERC à 15 violations et 0 erreur. Le `NEEDS_DATA` sur l'inductance tombe si `PA6331-AE` est retenue.
