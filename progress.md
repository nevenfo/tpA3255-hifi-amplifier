# PROGRESS

## Phase actuelle

**Phase F — Routage.** **31 segments posés, zéro via** : les quatre boucles de bootstrap, `PVDD`,
et les découplages auxiliaires `AVDD`/`DVDD`/`+12V`. DRC courant : **112 violations toutes de
sérigraphie** (90 `silk_overlap`, 22 `silk_over_copper`), **239 non-connectés**,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`.

## Tâche actuelle

**F1.1 — router les boucles de commutation, l'alimentation Class-D et les découplages.** Les trois
tranches sont routées. **Reste un seul point ouvert : `VBG` → `C303`, dont l'impossibilité
mono-couche est démontrée et qui attend un arbitrage utilisateur.** Le retour `GND` est renvoyé au
plan de masse de F1.4.

## Dernière tâche validée

**F1.1, tranche « découplages auxiliaires » = PASS sauf `VBG`.** `AVDD` (pin 14 → `C318`), `DVDD`
(pin 11 → `C319`) et `/+12V` (pins 1, 2, 22 → `C302`, `C304`) sur `F.Cu`, **15 segments, zéro via**,
largeur 0,30 mm.

Validation :

- **DRC 112 violations, identiques à la baseline** ; 0 `clearance`, 0 `shorting_items`,
  0 `track_dangling` ; parité **3**, les trois mêmes écarts ; `pad_prop_heatsink` de `U1` à 1.
- **Non-connectés 244 → 239**, exactement les cinq chevelus prédits (`AVDD`, `DVDD`, et les trois
  de `+12V`).
- **Vérification indépendante du principal** : DRC relancé ; **31 segments tous sur `F.Cu`**,
  **0 via** au fichier ; largeurs 12 × 0,35 + 2 × 0,8 + 2 × 1,1 + 15 × 0,30 ; **diff Git sans
  aucune suppression de segment**, les 6 seules lignes retirées étant une normalisation d'écriture
  de KiCad (`-90` → `270`) sur les champs texte de `C310`, dont le placement est inchangé.
- **Un premier lot avait produit 6 `shorting_items`** : les découplages d'entrée `C106`/`C107`/
  `C206`/`C207` manquaient à la liste d'obstacles. Segments retirés, géométrie recalculée, reposée.

**Avant elle** : les tranches « bootstraps » et « alimentation Class-D » de F1.1.

## Décisions actives

Les règles durables de routage (classes de net, largeurs tenables en sortie de `U6`, exhaustivité
des obstacles, calibration des contrôles, ouverture de l'éditeur, normalisation des angles) sont
dans `docs/kicad-operations.md` ; placement et budgets thermiques dans `docs/architecture.md`.
Restent ici celles qui gouvernent la prochaine action :

- **`VBG` → `C303` exige une via, et c'est prouvé, pas supposé.** Côté `U6`, la pin 22 (`y` =
  168,33) précède `VBG` (169,6) ; côté condensateurs, `C303` (163) précède `C304` (166) :
  **l'ordre s'inverse entre source et cible**, donc toute liaison mono-couche croise. Détour
  externe, contournement local et inversion de priorité ont été testés et écartés par calcul.
- **`AVDD` et `DVDD` sont longs — 28,95 et 34,82 mm** pour une distance directe d'environ 13,8 mm,
  le corridor central étant saturé. Acceptable pour des rails auxiliaires filtrés, mais **à
  réexaminer en F1.4** quand les plans offriront un retour et des vias.
- **Le retour `GND` local ne passe pas sur `F.Cu` près de `U6`** : les masses partent au plan de
  F1.4.
- **La symétrie miroir autour de `y` = 175 est exacte dans le placement** et doit le rester dans le
  routage : le vérifier **numériquement, sommet par sommet**, jamais à l'œil.
- **Contrainte F1** : apparier les sorties **par paire de pont**, A avec B et C avec D.
- **Zones interdites par la barre de liaison, et elles se composent** : `x` ∈ [280, 291],
  `y` ∈ [160, 190] ; et `x` ∈ [280, 300], `y` ∈ [169, 181]. Contour `(100,100)`–`(300,250)`.
- **La séparation des masses sera purement géométrique** : un seul net `/GND`, aucune zone encore
  dessinée. `pad_prop_heatsink` de `U1` est la seule occurrence de la carte et se perd sur un
  déplacement IPC : le vérifier après chacun.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; les fichiers de configuration
  s'éditent directement.

## Blocage actif

**Arbitrage utilisateur en attente sur `VBG` → `C303`** — seule liaison de F1.1 non routée.
Symptôme : croisement inévitable avec `/+12V` pin 22. Cause établie : inversion de l'ordre des
extrémités entre `U6` et la rangée de condensateurs. Écarté par calcul : détour externe (croise
`AVDD`), contournement local (0,75 mm requis contre 0,65 mm disponible), inversion de priorité.
Prochaine tentative : appliquer l'option retenue par l'utilisateur.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, **31 segments routés**, 0 via ; `.kicad_pro` porte
  les classes de net ; `.kicad_dru` la règle de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation** ; convention de rotation des pads et format des segments
- `docs/architecture.md` — placement, symétrie, retour des courants et masses, budgets thermiques

## NEXT ACTION

**F1.1 — trancher `VBG` → `C303` avec l'utilisateur, puis clore F1.1.** Trois options : poser une
via sur `VBG` (rompt le « zéro via » mais route la liaison proprement) ; déplacer `C303` et `C304`
pour rétablir l'ordre des extrémités (rouvre le placement, clos depuis la Phase E) ; ou reporter
`VBG` au plan de masse et aux couloirs de F1.4. Une fois tranché, appliquer, revalider par DRC
(0 `clearance`, 0 `shorting_items`, 0 `track_dangling` ; parité 3 ; non-connectés 239 → 238 si
routé) et cocher F1.1.
