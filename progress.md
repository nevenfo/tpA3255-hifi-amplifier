# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a close, F1.2-b débloquée et en cours.** **58 segments posés, zéro
via** : les quatre boucles de bootstrap, `PVDD`, les découplages auxiliaires (F1.1), et les
quatre sorties filtrées `self → condensateur → bornier` à 3,00 mm (F1.2-a). DRC courant :
**113 violations toutes de sérigraphie** (91 `silk_overlap`, 22 `silk_over_copper`),
**230 non-connectés**, `schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`,
aucun `track_dangling`.

## Tâche actuelle

**F1.2-b1 — basculer `C310`/`C311` sur `B.Cu` et rerouter `/PVDD` par vias.** Premier des trois
gestes que l'arbitrage a ouverts, à la suite desquels F1.2-b2 puis F1.2-b3 s'enchaînent.

## Dernière tâche validée

**F1.2-a = PASS.** Les quatre nets `/OUT_x_F` sont routés, **23 segments à 3,00 mm, zéro via**,
après deux échanges d'emplacement entre composants identiques — `L302` ↔ `L303` et `C322` ↔ `C323`
— qui alignent l'ordre des condensateurs sur celui des borniers et rendent la topologie planaire.

Validation :

- **Non-connectés 238 → 230**, les huit chevelus prédits ; **0 `clearance`, 0 `shorting_items`,
  0 `track_dangling`, 0 `solder_mask_bridge`, 0 `copper_edge_clearance`** ; `schematic_parity`
  toujours **3** ; sérigraphie **113 inchangée** ; 124 empreintes, `pad_prop_heatsink` de `U1` à 1.
- **Vérification indépendante du principal** : DRC relancé en `kicad-cli --schematic-parity` ;
  **58 segments tous sur `F.Cu`**, **0 via** ; **zéro intersection** entre les 23 segments, testée
  segment à segment ; diff Git ne montrant **que les quatre `(at)` échangés et les 23 segments**.

## Décisions actives

Les règles durables sont dans `docs/kicad-operations.md`, le placement et les mesures dans
`docs/architecture.md`, sections **F1.2-a** et **F1.2-b**. Restent ici :

- **Arbitrage F1.2-b rendu, en deux gestes indépendants.** `C310`/`C311` **passent au dos sur
  `B.Cu`**, vias sous les broches `PVDD`/`GND` — la colonne `x` ∈ [276 ; 279] se libère et la
  boucle de découplage **raccourcit** au lieu de s'allonger. Le croisement structurel se dénoue
  par **une via sur chaque `BST`**, jamais sur un `OUT` à 5 A. **Coût assumé : la carte cesse
  d'être simple-face**, second passage d'assemblage pour deux composants. Voies écartées et
  motifs dans `docs/architecture.md`.
- **La classe n'est jamais tenable en sortie de `U6`** — 0,35 mm aux bootstraps, 0,30 mm aux
  auxiliaires — mais elle le redevient hors du pas fin : `PWR_OUT` passe à ses 3,00 mm dans la
  région des sorties. Recalculer dans les deux sens.
- **Règle DRU d'échappement posée et prouvée inerte** ; **sa sélectivité reste à prouver** en
  retirant `U6` de sa condition, quand une piste l'exercera.
- **`OUT_B` (35) et `OUT_C` (32) n'ont qu'une pastille**, `OUT_A` et `OUT_D` en ont deux.
  Conforme à SLASEA8, vérifié : ce n'est pas un défaut de symbole.
- **Le déplacement par MCP retire le bloc `(units …)`** de l'empreinte déplacée, invisiblement au
  DRC. Compter les blocs `(units` avant et après — il y en a 119. Le `.kicad_pcb` est en CRLF.
- **113 violations de sérigraphie restent à traiter en bloc**, aucune n'ayant d'effet cuivre.
- **Masses locales et rails auxiliaires trop longs partent au plan de F1.4**, où `AVDD` et `DVDD`
  — 28,95 et 34,82 mm — seront réexaminés. Un seul net `/GND`, aucune zone dessinée.
- **Les réservations de la barre de liaison interdisent les composants, pas le cuivre** :
  `x` ∈ [280, 291], `y` ∈ [160, 190] et `x` ∈ [280, 300], `y` ∈ [169, 181]. Contour
  `(100,100)`–`(300,250)`.
- Toute édition PCB passe par `kicad-control`/MCP, qui **exige l'éditeur de PCB ouvert depuis le
  gestionnaire de projet** — un `pcbnew.exe` isolé ne partage pas l'IPC. Ouvrir le gestionnaire
  par `explorer.exe <projet>.kicad_pro`, jamais depuis le shell d'un agent.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, **58 segments routés**, 0 via ; `.kicad_pro` porte
  les classes de net ; `.kicad_dru` les deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation**
- `docs/architecture.md` — placement, symétrie, retour des courants, budgets thermiques ; section
  **F1.2-b** pour la mesure du champ d'échappement et l'arbitrage rendu

## NEXT ACTION

**F1.2-b1 — basculer `C310` et `C311` sur `B.Cu`, puis rerouter `/PVDD` par vias sous les broches
`PVDD`/`GND` de `U6`.** Valider par : trajet équivalent de la boucle de découplage **≤ 7,71 mm** ;
colonne `x` ∈ [276,15 ; 278,85] libre de tout cuivre `F.Cu` ; DRC sans `clearance`,
`shorting_items` ni `track_dangling` ; parité **3** ; non-connectés **inchangés à 230** ;
`pad_prop_heatsink` à 1 ; **124 empreintes et 119 blocs `(units`**.
