# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** Baseline ERC 16 violations / 0 erreur (`GATE C2`). DRC courant :
**112 violations toutes de sérigraphie**, 254 non-connectés, aucune `clearance`, aucun
`courtyards_overlap`, aucun `lib_footprint_mismatch`. Parité **7** — 3 de baseline plus les 4
pastilles des borniers en attente de propagation.

## Tâche actuelle

**E1.11 — permuter les broches de `J301`/`J302`.** Schéma corrigé et validé ; **la propagation au
PCB est bloquée** et exige une action de l'utilisateur. **E1.12 est close.** C'est la seule tâche
ouverte de la phase.

## Dernière tâche validée

**E1.12 = PASS.** Les quatre condensateurs d'entrée sont devant les broches de `U6` : `C106`
(291,5 ; 182,0), `C107` (295,0 ; 182,0), `C206` (295,0 ; 168,63), `C207` (291,5 ; 168,64), au
micron et sans rotation. **3,50 à 6,54 mm** de leur broche au lieu de 160.

Validation :

- DRC **112 violations toutes de sérigraphie**, **aucune `clearance` ni `courtyards_overlap`** —
  le seul risque réel, les cibles étant bordées par `C302`, `R301`, `C304`, `R302`.
- 124 empreintes, **seules ces quatre déplacées** ; `pad_prop_heatsink` à 1 ; 254 non-connectés.
- **Symétrie redéfinie et vérifiée** : `C106`+`C207` = 350,64 et `C107`+`C206` = 350,63, le miroir
  du brochage. Les quatre hors des deux zones interdites, par test explicite.

**Avant elle** : E1.11 tranche « schéma » (ERC 16 / 0 inchangé, et la découverte que
`schematic_parity` n'avait jamais tourné) ; **E1 close** — E1.5 en cinq tranches, E1.4, E1.10,
E1.3, E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

## Décisions actives

Placement, budgets thermiques et pilotage KiCad sont dans `docs/architecture.md` et
`docs/kicad-operations.md`. Restent ici celles qui gouvernent la prochaine action :

- **Le brochage de `U6` est un miroir autour de `y` = 175**, qui inverse l'ordre des nets d'une
  moitié à l'autre. **Une position mise en miroir ne suffit pas : l'orientation doit l'être
  aussi.**
- **Un croisement se prouve par intersection de segments, jamais par une aire.**
- **La symétrie n'est pas un vecteur unique** : (0 ; +40) en analogique, (0 ; +17) aux selfs,
  (−22 ; 0) aux sorties, miroir autour de 175,32 pour les entrées après E1.12. **Contrainte F1** :
  apparier les sorties **par paire de pont**, A avec B et C avec D.
- **Zones interdites par la barre de liaison, et elles se composent** : `x` ∈ [280, 291],
  `y` ∈ [160, 190] ; et `x` ∈ [280, 300], `y` ∈ [169, 181]. Contour `(100,100)`–`(300,250)`.
- **Un déplacement IPC, ou une mise à jour depuis le schéma, peut perdre `pad_prop_heatsink`** —
  `U1` en est la seule occurrence. À vérifier après chacune, avec `lib_footprint_mismatch`.
- **Une contrainte d'encombrement se valide au DRC, pas au calcul de courtyard.**
- **`schematic_parity` exige `--schematic-parity`**, sinon le test ne tourne pas et le champ vaut
  `0`. **Vraie baseline = 3** (`U6` pin 45 sans pastille, PowerPAD sur le dessus ; `H1`/`H2` sans
  symbole). **Comparer à 3, jamais à 0** — un compteur à zéro ne prouve rien tant qu'on n'a pas
  vérifié que la mesure a eu lieu.
- **Un DRC de comparaison se lance dans le répertoire du projet.** Binaire :
  `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; les fichiers de
  configuration s'éditent directement.

## Blocage actif

**La propagation du schéma vers le PCB n'est pas automatisable, et E1.11 l'attend.**

- **Symptôme** : le schéma porte la permutation, la carte non. `schematic_parity` = **7** au lieu
  de **3**, les quatre écarts en trop étant exactement les pastilles des borniers.
- **Cause établie** : aucune commande `kicad-cli` ni aucun outil MCP ne réalise « Mettre à jour le
  PCB depuis le schéma ».
- **Faits exclus** : ni l'IPC, qui fonctionne, ni le schéma, dont l'ERC reste à 16 / 0.
- **Prochaine tentative** : **l'utilisateur lance `Outils → Mettre à jour le PCB depuis le
  schéma`** dans l'éditeur de PCB, déjà ouvert.
- **État préservé et commité** : `.kicad_sch` `b0e93635ea39e76597a7d20b9ce3aa97`, `.kicad_pcb`
  intact à `eb5273ac4252b64fd97f4bd90b058fd9`.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes **toutes placées** ; `.kicad_dru` porte la règle
  de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation**
- `docs/architecture.md` — placement, symétrie, retour des courants et masses, budgets thermiques

## NEXT ACTION

**E1.11, propagation au PCB.** Elle demande **une action de l'utilisateur**, décrite au blocage :
`Outils → Mettre à jour le PCB depuis le schéma`, dans l'éditeur déjà ouvert. Dès qu'elle est
faite, reprendre sans rien redemander et valider :

- `kicad-cli pcb drc --schematic-parity` depuis le répertoire du projet : parité ramenée à **3**.
- `J301` pad 1 = `/OUT_B_F`, pad 2 = `/OUT_A_F` ; `J302` pad 1 = `/OUT_D_F`, pad 2 = `/OUT_C_F` ;
  **test d'intersection rejoué = aucun croisement**.
- **Les deux régressions déjà causées par cette opération** : `pad_prop_heatsink` de `U1` à **1**,
  `lib_footprint_mismatch` à **0**.
- 124 empreintes intactes au micron, 254 non-connectés, aucune `clearance`.

Ensuite **F1.1** — router les boucles de commutation, l'alimentation Class-D et les découplages.
Les quatre boucles de bootstrap **d'abord et sans via** : E1.5 a retiré le dernier croisement qui
en aurait imposé un, ce serait le perdre que de router autrement.
