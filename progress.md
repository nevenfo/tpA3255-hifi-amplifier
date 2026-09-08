# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a close, F1.2-b ouverte.** **58 segments posés, zéro via** : les quatre
boucles de bootstrap, `PVDD`, les découplages auxiliaires (F1.1), et les quatre sorties filtrées
`self → condensateur → bornier` à 3,00 mm (F1.2-a). DRC courant : **113 violations toutes de
sérigraphie** (91 `silk_overlap`, 22 `silk_over_copper`), **230 non-connectés**,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`.

## Tâche actuelle

**F1.2-b — faire sortir `/OUT_A` à `/OUT_D` de `U6` vers les quatre selfs.** Pas encore commencée.
C'est la moitié difficile de F1.2 : quatre nœuds de commutation à 5 A, contraints d'être **courts**,
qui doivent quitter un boîtier au pas de 0,635 mm.

## Dernière tâche validée

**F1.2-a = PASS.** Les quatre nets `/OUT_x_F` sont routés, **23 segments à 3,00 mm, zéro via**,
après deux échanges d'emplacement entre composants identiques — `L302` ↔ `L303` et `C322` ↔ `C323`
— qui alignent l'ordre des condensateurs sur celui des borniers et rendent la topologie planaire.

Validation :

- **Non-connectés 238 → 230**, les huit chevelus prédits, deux par net ; **0 `clearance`,
  0 `shorting_items`, 0 `track_dangling`, 0 `solder_mask_bridge`, 0 `copper_edge_clearance`** ;
  `schematic_parity` toujours **3** ; 124 empreintes, `pad_prop_heatsink` de `U1` à 1.
- **Sérigraphie 113, strictement inchangée** malgré les quatre déplacements.
- **Vérification indépendante du principal** : DRC relancé en `kicad-cli --schematic-parity` ;
  **58 segments tous sur `F.Cu`**, **0 via** ; largeurs 23 × 3,00 + les 35 de F1.1 intactes ;
  **zéro intersection** entre les 23 segments, testée segment à segment ; diff Git ne montrant
  **que les quatre `(at)` échangés et les 23 segments**.

## Décisions actives

Les règles durables sont dans `docs/kicad-operations.md`, le placement et la symétrie dans
`docs/architecture.md`, section **F1.2-a**. Restent ici celles qui gouvernent la prochaine action :

- **La classe n'est jamais tenable en sortie de `U6`** — 0,35 mm aux bootstraps, 0,30 mm aux
  auxiliaires — mais elle le redevient dès qu'on quitte le pas fin : `PWR_OUT` passe à ses
  3,00 mm dans la région des sorties. Recalculer dans les deux sens.
- **Sans la règle DRU d'échappement, aucune piste ne sort de `OUT_B` ni de `OUT_C`** : à 0,50 mm
  d'isolation la largeur maximale y est **négative**. La règle est posée et **prouvée inerte** ;
  **sa sélectivité reste à prouver** en retirant `U6` de sa condition.
- **`OUT_B` (35) et `OUT_C` (32) n'ont qu'une pastille**, `OUT_A` (39-40) et `OUT_D` (27-28) en ont
  deux. Conforme à SLASEA8, vérifié : ce n'est pas un défaut de symbole.
- **`C310` et `C311` barrent les quatre échappements en direct** : `C311` couvre `y` 168,88–174,52
  donc `OUT_D` et `OUT_C`, `C310` couvre 175,48–181,12 donc `OUT_B` et `OUT_A`. Le jour entre les
  deux ne fait que **0,96 mm** ; le couloir entre eux et les pastilles de `U6`, **1,65 mm**, est
  déjà partiellement pris par `/PVDD`.
- **Le champ `x` ∈ [195, 272], `y` ∈ [184, 193] est entièrement libre** : c'est là que les
  échappements ont de la place, une fois `C310`/`C311` contournés.
- **Le déplacement par MCP retire le bloc `(units …)`** de l'empreinte déplacée, invisiblement au
  DRC. Compter les blocs `(units` avant et après — il y en a 119.
- **113 violations de sérigraphie restent à traiter en bloc**, aucune n'ayant d'effet cuivre.
- **Les masses locales et les rails auxiliaires trop longs partent au plan de F1.4**, où `AVDD`
  et `DVDD` — 28,95 et 34,82 mm — seront réexaminés. Un seul net `/GND`, aucune zone dessinée.
- **Zones interdites par la barre de liaison, et elles se composent** : `x` ∈ [280, 291],
  `y` ∈ [160, 190] ; et `x` ∈ [280, 300], `y` ∈ [169, 181]. Contour `(100,100)`–`(300,250)`.
- Toute édition PCB passe par `kicad-control`/MCP, qui **exige l'éditeur ouvert** — `pcbnew.exe
  <fichier>` suffit, aucun geste GUI. Les fichiers de configuration s'éditent directement.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, **58 segments routés**, 0 via ; `.kicad_pro` porte
  les classes de net ; `.kicad_dru` les deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation**
- `docs/architecture.md` — placement, symétrie, retour des courants, budgets thermiques ; section
  **F1.2-a** pour l'état en vigueur des sorties

## NEXT ACTION

**F1.2-b — router `/OUT_A` à `/OUT_D` de `U6` vers les pastilles d'entrée des selfs** `L301`
(276 ; 195), `L302` (276 ; 212), `L303` (240 ; 195), `L304` (240 ; 212). Établir d'abord **par
calcul, avant tout tracé**, la largeur d'échappement tenable pour chacune des quatre sorties, en
distinguant les pastilles simples (`OUT_B`, `OUT_C`) des doubles (`OUT_A`, `OUT_D`), puis le
contournement de `C310`/`C311` — les quatre trajets directs sont barrés. Reprendre la largeur de
classe **3,00 mm dès la sortie du courtyard**, le neck d'échappement restant court. Router sur
`F.Cu` sans via si la géométrie le permet ; sinon le dire avant d'en poser une. Valider par : DRC
sans `clearance`, `shorting_items` ni `track_dangling` ; `schematic_parity` toujours **3** ;
non-connectés en baisse du nombre exact de chevelus résolus ; `pad_prop_heatsink` de `U1` à 1 ;
124 empreintes et 119 blocs `(units` intacts ; **sélectivité de la règle DRU d'échappement prouvée**
en retirant `U6` de sa condition.
