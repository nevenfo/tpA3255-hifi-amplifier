# PROGRESS

## Phase actuelle

Phase D. **GATE C2 = PASS**, revérifié après chaque écriture : ERC à 15 violations et 0 erreur, jeu identique à la baseline C2 (10 `endpoint_off_grid`, 5 `lib_symbol_mismatch`).

## Tâche actuelle

D1.2 — Vérifier boîtiers fabricant, orientations, courants et contraintes d'assemblage.

## Dernière tâche validée

**D1.11 = PASS.** Les six bulks sont instruits : un défaut corrigé, une empreinte confirmée juste.

- **`C312` à `C315`** portaient `CP_Radial_D16.0mm_P7.50mm`, soit **2 mm de trop peu en diamètre**. La nomenclature du kit TI `TPA3255EVM` (`SLOU441`, p. 14) — l'ancre déclarée du bulk — donne pour ce poste `EEU-FC1J152` Panasonic en **ø18 mm**. Corrigées en `Capacitor_THT:CP_Radial_D18.0mm_P7.50mm`, librairie standard, pas inchangé. Écriture MCP relue au fichier par le principal, positions inchangées.
- **`C316`/`C317`** : `CP_Radial_D35.0mm_P10.00mm_SnapIn` **validée**. La BOM donne `SLPX472M080H3P3` Cornell Dubilier, « D35 × L30 » ; la clé CDE type SLP décodée au catalogue redonne bien H = ø35 et 3 = 30 mm ; le diagramme « PC Board Mounting Holes » du même catalogue impose **pas 10,0 mm et deux trous ø2,0 ± 0,1**, identiques au Vishay 058. **Le perçage de 2,0 mm n'est donc pas un jeu nul sur une broche de 2,0 mm : c'est la cote prescrite par les deux fabricants.**
- **Méthode qui a payé : remonter à la nomenclature du kit d'évaluation servant d'ancre au design, plutôt que chercher une série au hasard.** Elle donne le boîtier réel de chaque poste et sert de point d'entrée pour décoder la clé du fabricant.

## Défauts ouverts

- **D1.9 — `C321` à `C324` portent une empreinte impossible.** 3,5 mm d'épaisseur supposée, jamais lue. Le filtre étant référencé à `GND` en aval de chaque demi-pont, il voit tout le rail et TI impose un calibre 100 V ; or sous 100 V une boîte de 3,5 mm au pas de 5 mm ne loge que 0,15 à 0,22 µF, et le 680 nF mesure 5 × 10 × 7,2 mm. Ne pas re-supposer : la référence est `NEEDS_DATA`, et le diélectrique est à trancher d'abord (MKP polypropylène attendu sur un filtre Class-D, pas MKS polyester), ce qui peut déplacer le pas. **`SLOU441` contient la BOM de l'EVM et donnera très probablement la référence de ce poste : à exploiter en premier.**
- **D1.10 — `L301` à `L304` portent `L_Wuerth_HCI-1350` sans MPN**, alors que l'inductance 15 µH est `NEEDS_DATA`. Le boîtier 1350 est très probablement trop petit : 342 µJ stockés contre 750 µJ exigés. **Même remarque : exploiter la BOM `SLOU441` avant toute recherche de série.**
- **D1.8 déverrouillé, plus aucun arbitrage requis.** `CF_Film_Box_P5.00mm_7.2x3.5mm` était redondante dès sa création : la librairie standard contient `C_Rect_L7.2mm_W3.5mm_P5.00mm_FKS2_FKP2_MKS2_MKP2`, courtyard correct de 7,7 × 4,0 mm contre 7,6 × 2,6 au local. Suppression à la fermeture de D1.9. Le fusible `Fuse_Schurter_UMT-H_5.3x16mm` reste justifié, son défaut purement cosmétique.
- **`J1`, bornier MaiXu MX126-5.0 : calibre en courant toujours non instruit.** 4,6 A continus et 10 A transitoires attendus. Sa seule datasheet connue est celle que cite l'empreinte KiCad, hébergée par LCSC, **et LCSC refuse `curl`**.

## Blocage actif

Aucun.

## Contraintes portées en Phase E

- **Bulk : `C312` à `C315` gagnent 2 mm de courtyard chacun** (16,16 → 18,16 mm), hauteur nominale 35 mm. `C316`/`C317` : ø35 mais **30 mm de haut seulement**, et non les 50 mm que suggère la description de l'empreinte KiCad — plus favorable qu'attendu sous capot.
- `C325` culmine à **18 mm** : composant film le plus encombrant.
- **`R306` est un shunt à deux bornes, pas Kelvin.** Les liaisons vers `VIN` et `SENSE` doivent partir des **bords intérieurs** des pastilles, le courant de puissance entrant par les bords extérieurs. Repli : `WSK25122L000FEA`, quatre bornes, même boîtier 2512.
- `Q302` doit être monté sur radiateur : sa SOA suppose le boîtier à 75 °C.
- `Q301` : courant continu plafonné à 6,9 A avec la surface de cuivre de référence de sa datasheet, 6 cm² en 70 µm.
- Rail d'entrée : les calibres UMT-H supposent des pistes de 7,5 mm en cuivre 140 µm. Déclassement sinon.
- `C110`/`C210` imposent un établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec la temporisation de mute en Phase F.
- Connectique déportée en JST XH : deuxième famille à approvisionner à côté des MaiXu MX126-5.0, et pince à sertir nécessaire.

## NEEDS_DATA ouverts

`RV1` (mécanique du potentiomètre), EP du TPA3255DDV à 5,2 × 14 mm à confirmer sur dessin mécanique TI, alimentation externe 48 V, **films 680 nF (bloque D1.9) et inductances 15 µH (bloque D1.10)**, dissipateur et thermique, réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33, calibre en courant du bornier `J1`.

Candidats d'ancrage **non inscrits au schéma** tant que la clé fabricant n'est pas décodée : `EEU-FC1J152` (Panasonic, ø18) pour `C312` à `C315` ; `SLPX472M080H3P3` (CDE) pour `C316`/`C317`, sachant que le catalogue SLP courant ne liste pas ce code et donne à 80 V `SLP472M080E4P3` en 30 × 45 et `SLP472M080H5P3` en 35 × 35.

Assumés : stabilité de la boucle de limitation de puissance du LM5069 face aux 540 nC de grille de `Q302` ; `V_C` de la `SMDJ58CA` sous `I_PP` ; pas de 7,5 mm du boîtier ø18, inchangé par la correction mais non relu chez Panasonic.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.**
- **`kicad-cli.exe` est utilisable directement** (`sch erc`, `sch export netlist`) et fournit une preuve indépendante du MCP, sans GUI. Chemin : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **Avant de créer une empreinte locale, épuiser la librairie standard.** Sur les deux locales créées jusqu'ici, une seule était nécessaire.
- **Lire les cotes par extraction du PDF fabricant, pas par recherche web ni par listing distributeur.** `pymupdf` installé, `pdftotext` dans `/mingw64/bin`. **Toujours contrôler si un tracé est à l'échelle** : celui de Schurter ne l'est pas. **Et toujours vérifier à quelle figure appartient une cote** : le « ø2 ± 0,1 » des snap-in est une cote de perçage, pas de broche.
- **Une référence ne se valide pas sur son aspect, mais en la décodant champ par champ contre la clé du fabricant, puis en recoupant la boîte obtenue avec le tableau de la valeur visée.**
- **Serveurs qui servent le PDF à `curl`** : ti.com/lit, vishay.com/docs, content.kemet.com, cde.com, tdk-electronics.tdk.com, Infineon. **Serveurs qui refusent ou ne répondent pas** : Littelfuse, DigiKey, LCSC, nichicon.co.jp, rubycon.co.jp, industrial.panasonic.com (timeout complet).
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
- **`create_footprint` impose ses propres graphiques** et aucun outil ne les édite après coup.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`, `.kicad_sym` (symbole `LM5069` local), `sym-lib-table`, `fp-lib-table`
- `HifiAmp_TPA3255_Local.pretty/` : `Fuse_Schurter_UMT-H_5.3x16mm` (justifiée), `CF_Film_Box_P5.00mm_7.2x3.5mm` (à supprimer, D1.9)
- `docs/architecture.md`, `docs/power-block.md`, `docs/protection-48v.md`
- Librairies KiCad : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/share/kicad/footprints`
- PDF déjà téléchargés, dans le scratchpad de session : `slou441.pdf` (BOM EVM TPA3255, p. 14), `SLP.pdf` (CDE), `058059pll-si.pdf` (Vishay), `KEM_A4082_ALC80.pdf`.

## NEXT ACTION

D1.9 — exploiter la BOM du `TPA3255EVM` (`SLOU441`, p. 14, PDF déjà présent dans le scratchpad) pour lire la référence réelle des quatre films `680 nF` du filtre de sortie et leur diélectrique, puis décoder cette référence contre la clé de son fabricant pour en tirer l'épaisseur de boîte et le pas. Corriger ensuite `C321` à `C324` avec le membre correspondant de la famille standard `Capacitor_THT:C_Rect_L7.2mm_W*_P5.00mm_FKS2_FKP2_MKS2_MKP2`, relire au fichier, revérifier l'ERC à 15 violations et 0 erreur, puis supprimer l'empreinte locale `CF_Film_Box_P5.00mm_7.2x3.5mm` devenue sans utilisateur. Enchaîner sur D1.10 par la même voie.
