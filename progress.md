# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a close, F1.2-b ouverte.** **58 segments posés, zéro via** : les quatre
boucles de bootstrap, `PVDD`, les découplages auxiliaires (F1.1), et les quatre sorties filtrées
`self → condensateur → bornier` à 3,00 mm (F1.2-a). DRC courant : **113 violations toutes de
sérigraphie** (91 `silk_overlap`, 22 `silk_over_copper`), **230 non-connectés**,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`.

## Tâche actuelle

**F1.2-b — faire sortir `/OUT_A` à `/OUT_D` de `U6` vers les quatre selfs. BLOQUÉE sur décision.**
La mesure établit qu'**aucune des quatre sorties ne peut quitter `U6` sur `F.Cu` à une largeur
utilisable** avec le placement et le routage F1.1 en vigueur. Détail dans « Blocage actif ».

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
- Toute édition PCB passe par `kicad-control`/MCP, qui **exige l'éditeur ouvert** — `pcbnew.exe
  <fichier>` suffit, aucun geste GUI.

## Blocage actif

**F1.2-b : les quatre sorties de `U6` sont enfermées.** Mesure et tableaux dans
`docs/architecture.md`, section **F1.2-b**. La seule fenêtre large fait **2,50 mm** et doit être
partagée par `OUT_C` et `OUT_B` — 0,50 mm chacune ; `OUT_A` et `OUT_D` n'ont qu'un jour de
**0,725 mm**, largeur **négative** à l'isolation `PWR_OUT`. Il en faut 3,00 mm pour 5 A. Causes :
`C310`/`C311` posés en face du flanc de puissance, et `BST_B` traversant structurellement la
rangée `OUT_A` (`BST_C` de même sur `OUT_D`). Contournements haut, bas et droite déjà exclus.

**Prochaine tentative : aucune sans arbitrage.** Trois voies à coût réel — déplacer `C310`/`C311`
en rallongeant la boucle `PVDD` ; descendre les `OUT` sur couche interne par vias ; reprendre le
voisinage complet de `U6`, selfs comprises.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, **58 segments routés**, 0 via ; `.kicad_pro` porte
  les classes de net ; `.kicad_dru` les deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation**
- `docs/architecture.md` — placement, symétrie, retour des courants, budgets thermiques ; section
  **F1.2-a** pour l'état en vigueur des sorties

## NEXT ACTION

**F1.2-b — obtenir l'arbitrage sur la libération du champ d'échappement de `U6`, puis router.**
Dès la réponse : appliquer le choix, puis router `/OUT_A` à `/OUT_D` des broches de `U6` vers les
pastilles d'entrée des selfs, en passant par la pastille `OUT` de chaque bootstrap — `C306` pad 1
(273,5 ; 179,625), `C307` pad 1 (273,5 ; 175,725), `C308` pad 1 (273,5 ; 174,275), `C309` pad 1
(273,5 ; 170,375). Valider par : DRC sans `clearance`, `shorting_items` ni `track_dangling` ;
parité **3** ; non-connectés **230 → 220** ; `pad_prop_heatsink` à 1 ; 124 empreintes et 119 blocs
`(units` ; **sélectivité de la règle DRU prouvée**.
