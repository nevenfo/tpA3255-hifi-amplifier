# PROGRESS

## Phase actuelle

**Phase F — Routage.** Phase E close. **16 segments posés, zéro via** : les quatre boucles de
bootstrap et l'alimentation `PVDD`. DRC courant : **112 violations toutes de sérigraphie** (90
`silk_overlap`, 22 `silk_over_copper`), **244 non-connectés**, `schematic_parity` = **3**, aucune
`clearance`, aucun `shorting_items`, aucun `track_dangling`.

## Tâche actuelle

**F1.1 — router les boucles de commutation, l'alimentation Class-D et les découplages.** Les
tranches « bootstraps » et « alimentation Class-D » sont validées. **Reste** : le découplage des
alimentations auxiliaires (`AVDD`, `DVDD`, `+12V`, `VBG`). Le retour `GND` est **renvoyé au plan
de masse de F1.4**, faute de passage sur `F.Cu`.

## Dernière tâche validée

**F1.1, tranche « alimentation Class-D » = PASS**, avec la rotation de `C310` décidée par
l'utilisateur. `/PVDD` routé sur `F.Cu`, **4 segments, zéro via**, largeurs 0,8 puis 1,1 mm.

Validation :

- **`C310` tourné de 90° à −90° sans bouger** (277,5 ; 178,3) : ses pastilles ont échangé leurs
  nets, `PVDD` est désormais en 176,825, face aux pins 36–38 de `U6`. Boucle ramenée de 9,04 mm
  avec croisement à **7,71 mm sans croisement**, identique à `C311`.
- **Audit d'orientation de tout le bloc** : sur les 8 paires en miroir, **3 sont discriminantes**
  (rotation hors 0/180) et **`C310`/`C311` était le seul défaut** ; `C306`/`C309` et `C307`/`C308`
  sont correctes.
- **DRC 112 violations, strictement inchangées** ; 0 `clearance`, 0 `shorting_items`,
  0 `track_dangling` ; parité **3** ; **non-connectés 250 → 244** ; **124 empreintes dont `C310`
  seule modifiée**, et seulement en rotation ; `pad_prop_heatsink` de `U1` à 1.
- **Vérification indépendante du principal** : orientations miroir, `PVDD` plus près de l'axe que
  `GND` sur les deux condensateurs, 0 via, miroir des segments exact, aucun croisement avec les
  12 segments de bootstrap — qui sont eux-mêmes revérifiés intacts.

**Avant elle** : F1.1 tranche « bootstraps », puis E1.11 et E1.12 qui closent la Phase E.

## Décisions actives

Placement, budgets thermiques et pilotage KiCad sont dans `docs/architecture.md` et
`docs/kicad-operations.md`. Restent ici celles qui gouvernent la prochaine action :

- **La largeur de classe n'est pas toujours tenable en sortie de `U6`.** Pastilles 1,575 × 0,4 mm
  au pas de 0,635 : pour 0,25 mm d'isolation, **0,5 mm et 0,4 mm violent, 0,35 mm passe**.
  Recalculer la marge à chaque sortie de boîtier fin plutôt que d'appliquer la classe.
- **L'isolation dépend de la classe du net : `PWR_48V` exige 0,5 mm, `GATE_DRIVE` et `GND`
  0,25 mm, `PWR_OUT` 0,5 mm.** Reprendre la valeur d'un net voisin a coûté un tracé refait.
  **Vérifier la classe avant de calculer un couloir.**
- **Le retour `GND` local ne passe pas sur `F.Cu` près de `U6`** : `/BST_B` traverse le couloir
  haut (`x` ≈ 279,72 à `y` ≈ 180) et `/BST_C` le bas. Les masses partent au **plan de F1.4**.
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

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes placées, **16 segments routés** (bootstraps et
  `PVDD`) ; `.kicad_dru` porte la règle de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation** ; contient notamment la convention de rotation des pads et le format des segments
- `docs/architecture.md` — placement, symétrie, retour des courants et masses, budgets thermiques

## NEXT ACTION

**F1.1, dernière tranche : router les découplages des alimentations auxiliaires de `U6`** —
`AVDD` (pin 14), `DVDD` (pin 11), `+12V` (pins 1, 2, 22) et `VBG` (pin 20), vers leurs
condensateurs respectifs. Établir d'abord **par mesure au fichier** quel condensateur sert quelle
broche, et **relever la classe de chaque net avant de calculer un couloir**. Router sur `F.Cu`
sans via si la géométrie le permet ; sinon le dire et s'arrêter. Valider par : DRC sans
`clearance` ni `shorting_items` ni `track_dangling` ; `schematic_parity` toujours **3** ;
non-connectés en baisse du nombre exact de chevelus résolus ; `pad_prop_heatsink` de `U1` à 1 ;
124 empreintes intactes ; aucun croisement avec les 16 segments déjà posés.
