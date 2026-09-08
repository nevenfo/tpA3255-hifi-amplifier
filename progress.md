# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches : close.** Toutes les tâches de E1 sont validées, y compris les deux
ajoutées en cours de phase (E1.11, E1.12). Baseline ERC 16 violations / 0 erreur (`GATE C2`).
DRC courant : **112 violations toutes de sérigraphie** (90 `silk_overlap`, 22 `silk_over_copper`),
254 non-connectés, **`schematic_parity` = 3**, aucune `clearance`, aucun `courtyards_overlap`,
aucun `lib_footprint_mismatch`. 124 empreintes toutes placées, **aucune piste routée**.

## Tâche actuelle

**F1.1 — router les boucles de commutation, l'alimentation Class-D et les découplages.** Rien
n'est encore routé ; c'est la première tâche de la phase F.

## Dernière tâche validée

**E1.11 = PASS**, schéma et PCB. La permutation des broches des borniers est propagée à la carte.

Validation :

- **`schematic_parity` ramené de 7 à 3**, mesuré avec `--schematic-parity`, et les 3 sont
  exactement la baseline : `U6` pin 45 sans pastille, `H1` et `H2` en `extra_footprint`.
- `J301` pad 1 = `/OUT_B_F`, pad 2 = `/OUT_A_F` ; `J302` pad 1 = `/OUT_D_F`, pad 2 = `/OUT_C_F`.
- **Test d'intersection : aucun croisement** sur les deux borniers, contre **deux avant** — le
  test a été calibré pour distinguer les deux états avant d'être cru.
- Aucune régression : **124 empreintes intactes au micron**, aucune disparue ni apparue,
  `pad_prop_heatsink` de `U1` à 1, `lib_footprint_mismatch` à 0, 254 non-connectés.

**Avant elle** : E1.12 (condensateurs d'entrée devant les broches de `U6`), E1.5 en cinq tranches,
E1.4, E1.10, E1.3, E1.9, E1.2, E1.8, E1.7, E1.6, E1.1.

## Décisions actives

Placement, budgets thermiques et pilotage KiCad sont dans `docs/architecture.md` et
`docs/kicad-operations.md`. Restent ici celles qui gouvernent la prochaine action :

- **Router les quatre boucles de bootstrap d'abord et sans via.** E1.5 a retiré le dernier
  croisement qui en aurait imposé un : router autrement serait perdre ce gain.
- **Contrainte F1** : apparier les sorties **par paire de pont**, A avec B et C avec D.
- **La séparation des masses sera purement géométrique** : il n'existe qu'un seul net `/GND`
  (83 pastilles, 70 composants) et aucune zone de cuivre n'est encore dessinée.
- **Zones interdites par la barre de liaison, et elles se composent** : `x` ∈ [280, 291],
  `y` ∈ [160, 190] ; et `x` ∈ [280, 300], `y` ∈ [169, 181]. Contour `(100,100)`–`(300,250)`.
- **Un croisement se prouve par intersection de segments, jamais par une aire** — et le test se
  calibre sur un état qu'il doit rejeter avant de servir de preuve.
- **`schematic_parity` exige `--schematic-parity`**, sinon le champ vaut `0` sans que le test ait
  tourné. **Comparer à 3, jamais à 0.**
- **Un déplacement IPC ou une mise à jour depuis le schéma peut perdre `pad_prop_heatsink`** —
  `U1` en est la seule occurrence. À vérifier après chacune, avec `lib_footprint_mismatch`.
- **Une contrainte d'encombrement se valide au DRC, pas au calcul de courtyard.**
- **Un DRC de comparaison se lance dans le répertoire du projet.** Binaire :
  `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; les fichiers de
  configuration s'éditent directement.

## Blocage actif

Aucun. Le blocage GUI qui retenait E1.11 venait d'une **session RDP déconnectée**, pas de KiCad :
reconnectée, le geste est passé par automatisation. La procédure et son piège principal — la case
« Supprimer les empreintes sans symbole associé », **cochée par défaut**, qui détruirait `H1` et
`H2` — sont consignés dans `docs/kicad-operations.md`.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes placées, **rien de routé** ; `.kicad_dru` porte la
  règle de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation**
- `docs/architecture.md` — placement, symétrie, retour des courants et masses, budgets thermiques

## NEXT ACTION

**F1.1 — router les quatre boucles de bootstrap de `U6`, sur `F.Cu` et sans aucune via.** Les
router en premier, avant toute autre piste, pour que rien ne vienne occuper leur passage. Valider
par : les quatre nets de bootstrap routés et **zéro via sur ces nets** ; DRC sans `clearance` ni
`shorting_items` ; `schematic_parity` toujours à **3** ; `pad_prop_heatsink` de `U1` toujours à 1 ;
124 empreintes toujours intactes au micron.
