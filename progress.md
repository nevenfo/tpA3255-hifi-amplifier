# PROGRESS

## Phase actuelle

**Phase F — Routage.** Phase E close. La carte vient de recevoir ses **premières pistes** : les
quatre boucles de bootstrap. DRC courant : **112 violations toutes de sérigraphie** (90
`silk_overlap`, 22 `silk_over_copper`), **250 non-connectés**, `schematic_parity` = **3**, aucune
`clearance`, aucun `shorting_items`, aucun `track_dangling`, **zéro via sur la carte**.

## Tâche actuelle

**F1.1 — router les boucles de commutation, l'alimentation Class-D et les découplages.** La
tranche « bootstraps » est validée. **Restent** : la commutation de puissance, l'alimentation
Class-D et les découplages.

## Dernière tâche validée

**F1.1, tranche « bootstraps » = PASS.** `/BST_A`–`/BST_D` routées sur `F.Cu`, 12 segments,
largeur **0,35 mm**, **zéro via**.

Validation :

- **DRC identique à la référence** : 112 violations, exactement les mêmes types et comptes ;
  **0 `clearance`, 0 `shorting_items`, 0 `track_dangling`** ; parité toujours **3** ; 124
  empreintes inchangées.
- **Non-connectés 254 → 250** : la preuve que les quatre liaisons sont réellement faites, et non
  posées à côté des pastilles.
- **Vérification indépendante du principal** : connexité pad à pad sans segment orphelin, **miroir
  `y` = 175 exact sommet par sommet** (A↔D sur 3, B↔C sur 5), aucun croisement sur les 6 paires,
  zéro via au fichier. Longueurs 7,806 mm (A, D) et 10,118 mm (B, C).

**Avant elle** : E1.11 (permutation propagée au PCB, parité 7 → 3) et E1.12, qui closent la
Phase E.

## Décisions actives

Placement, budgets thermiques et pilotage KiCad sont dans `docs/architecture.md` et
`docs/kicad-operations.md`. Restent ici celles qui gouvernent la prochaine action :

- **La largeur de classe n'est pas toujours tenable en sortie de `U6`.** Pastilles 1,575 × 0,4 mm
  au pas de 0,635 : pour 0,25 mm d'isolation, **0,5 mm et 0,4 mm violent, 0,35 mm passe**.
  Recalculer la marge à chaque sortie de boîtier fin plutôt que d'appliquer la classe.
- **`C310` et `C311` barrent les tracés directs vers `C307`/`C308`** ; le couloir libre entre
  leurs pastilles fait 1,8 mm, centré sur `y` = 178,3 et 171,7.
- **La symétrie miroir autour de `y` = 175 est exacte dans le placement** et doit le rester dans
  le routage : le vérifier **numériquement, sommet par sommet**, jamais à l'œil.
- **Contrainte F1** : apparier les sorties **par paire de pont**, A avec B et C avec D.
- **La séparation des masses sera purement géométrique** : un seul net `/GND` (83 pastilles,
  70 composants), aucune zone de cuivre encore dessinée.
- **Zones interdites par la barre de liaison, et elles se composent** : `x` ∈ [280, 291],
  `y` ∈ [160, 190] ; et `x` ∈ [280, 300], `y` ∈ [169, 181]. Contour `(100,100)`–`(300,250)`.
- **Un compteur nul ne prouve rien tant qu'on n'a pas vérifié que la mesure a eu lieu.** Vaut pour
  `schematic_parity` (exige `--schematic-parity`, se compare à **3**), pour les tests
  d'intersection et pour toute extraction du `.kicad_pcb`. **Calibrer d'abord le contrôle sur un
  état qu'il doit rejeter.**
- **Un déplacement IPC ou une mise à jour depuis le schéma peut perdre `pad_prop_heatsink`** —
  `U1` en est la seule occurrence. À vérifier après chacune, avec `lib_footprint_mismatch`.
- **Un DRC de comparaison se lance dans le répertoire du projet.** Binaire :
  `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; les fichiers de
  configuration s'éditent directement.

## Blocage actif

**Une décision utilisateur est requise avant de router l'alimentation Class-D.**

- **Symptôme** : `C310` et `C311`, les deux découplages `PVDD` de `U6`, sont **tous deux à
  rotation 90°**. Le miroir autour de `y` = 175 en exige d'opposées, comme `C306`/`C307` à 90° et
  `C308`/`C309` à −90° depuis E1.5.
- **Cause** : `docs/architecture.md` n'a vérifié le miroir de `C310`/`C311` que sur la **position**
  (178,3 contre 171,7), **jamais sur l'orientation** — l'angle mort exact de E1.5.
- **Conséquence mesurée** : `C311` est correct (7,71 mm de boucle, aucun croisement). `C310`
  présente son `PVDD` face au `GND` de `U6` : **9,04 mm et croisement `PVDD`/`GND`, donc une via
  obligatoire** dans la boucle la plus critique du Class-D. `C310` tourné à −90° retombe sur
  **7,71 mm sans croisement**, identique à `C311`.
- **Faits exclus** : la rotation ne déplace rien — un 1210 tourné de 180° garde son emprise, seules
  les pastilles échangent leurs nets, et le couloir où passe `/BST_B` reste libre.
- **Décision attendue** : tourner `C310` à −90° avant de router, ou router en l'état.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes placées, **12 segments routés** (bootstraps
  seulement) ; `.kicad_dru` porte la règle de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation** ; contient notamment la convention de rotation des pads et le format des segments
- `docs/architecture.md` — placement, symétrie, retour des courants et masses, budgets thermiques

## NEXT ACTION

**Trancher l'orientation de `C310`** (voir « Blocage actif »), puis router l'alimentation Class-D
de `U6` : `PVDD` pins 29–31 et 36–38, `GND` pins 25–26, 33–34, 41–42, vers `C310`/`C311`, au plus
court et à surface de boucle minimale, en respectant le miroir `y` = 175. Valider par : DRC sans
`clearance` ni `shorting_items` ni `track_dangling` ; `schematic_parity` toujours **3** ;
non-connectés en baisse du nombre exact de liaisons faites ; `pad_prop_heatsink` de `U1` toujours
à 1 ; 124 empreintes intactes hormis la rotation éventuellement décidée ; miroir vérifié
numériquement ; **zéro via sur le découplage**.
