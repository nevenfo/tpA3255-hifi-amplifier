# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1 close.** **35 segments posés, zéro via** : les quatre boucles de
bootstrap, `PVDD`, et les découplages auxiliaires `AVDD`/`DVDD`/`+12V`/`VBG`. DRC courant :
**113 violations toutes de sérigraphie** (91 `silk_overlap`, 22 `silk_over_copper`), **238
non-connectés**, `schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun
`track_dangling`.

## Tâche actuelle

**F1.2 — router les sorties vers les filtres LC et les connecteurs, avec des largeurs justifiées.**
Pas encore commencée. Classe `PWR_OUT` : piste nominale **3,00 mm**, isolation **0,50 mm**.

## Dernière tâche validée

**F1.1 = PASS**, dernière tranche incluse. `VBG` a été résolu **en corrigeant la cause plutôt qu'en
la contournant** : `C303` et `C304` ont échangé leurs positions (`y` 163 ↔ 166), ce qui aligne
l'ordre des extrémités et fait passer `VBG` et `+12V` pin 22 en direct, sans via.

Validation :

- **Non-connectés 239 → 238**, le chevelu prédit ; **0 `clearance`, 0 `shorting_items`,
  0 `track_dangling`** ; parité **3** ; `pad_prop_heatsink` de `U1` à 1, contrôlé au premier
  déplacement de la session ; **124 empreintes, exactement deux modifiées, seulement en position**.
- **Sérigraphie 112 → 113**, seul compteur à bouger : `C303` entre en conflit avec les champs
  référence de `C206`/`C207` et libère celui de `C318`. Cosmétique, aucun effet cuivre.
- **Vérification indépendante du principal** : DRC relancé ; **35 segments tous sur `F.Cu`**,
  **0 via** ; largeurs 12 × 0,35 + 2 × 0,8 + 2 × 1,1 + 15 × 0,30 + 4 × 0,25 ; diff Git ne montrant
  **que les deux `(at)` d'empreinte échangés**.

## Décisions actives

Les règles durables de routage (classes de net, largeurs tenables en sortie de `U6`, exhaustivité
des obstacles, calibration des contrôles, ouverture de l'éditeur, normalisation des angles) sont
dans `docs/kicad-operations.md` ; placement et budgets thermiques dans `docs/architecture.md`.
Restent ici celles qui gouvernent la prochaine action :

- **`AVDD` et `DVDD` sont longs — 28,95 et 34,82 mm** pour une distance directe d'environ 13,8 mm,
  le corridor central étant saturé. Acceptable pour des rails auxiliaires filtrés, mais **à
  réexaminer en F1.4** quand les plans offriront un retour et des vias.
- **Le déplacement IPC laisse derrière lui les pistes qu'il traîne** : trois segments obsolètes ont
  subsisté après le déplacement de `C304`. **Requêter les pistes d'un net après avoir déplacé l'un
  de ses composants**, sinon un `track_dangling` s'installe.
- **113 violations de sérigraphie restent à traiter en bloc**, aucune n'ayant d'effet cuivre.
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

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, **35 segments routés**, 0 via ; `.kicad_pro` porte
  les classes de net ; `.kicad_dru` la règle de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation** ; convention de rotation des pads et format des segments
- `docs/architecture.md` — placement, symétrie, retour des courants et masses, budgets thermiques

## NEXT ACTION

**F1.2 — router les sorties de puissance vers les filtres LC et les connecteurs.** Établir d'abord
**par mesure au fichier** quelles sorties de `U6` vont vers quelles selfs et quels connecteurs, et
**apparier par paire de pont, A avec B et C avec D**, conformément à la contrainte F1. Classe
`PWR_OUT` : nominal **3,00 mm**, isolation **0,50 mm** — mais **recalculer la largeur tenable en
sortie de `U6`**, où la classe n'a jamais été applicable. Router sur `F.Cu` sans via si la
géométrie le permet ; sinon le dire avant d'en poser une. Valider par : DRC sans `clearance`,
`shorting_items` ni `track_dangling` ; `schematic_parity` toujours **3** ; non-connectés en baisse
du nombre exact de chevelus résolus ; `pad_prop_heatsink` de `U1` à 1 ; 124 empreintes intactes ;
aucun croisement avec les 35 segments déjà posés.
