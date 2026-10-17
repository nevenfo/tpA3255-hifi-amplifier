# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1, F1.2 et F1.3-a closes ; F1.3-b à ouvrir.** 159 segments, **14 vias**.
DRC courant : **107 violations toutes de sérigraphie** (86 `silk_overlap`, 21 `silk_over_copper`),
**186 non-connectés** (78 `GND` + entrée/protection 39 + analogique 34 + signal 26 + contrôle 9),
`schematic_parity` = **3**, aucune `clearance`, `shorting_items`, `track_dangling` ni
`courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.3-b — entrée de puissance et protection `LM5069` (39 chevelus).**

## Dernière tâche validée

**F1.3-a = PASS — buck `U1` et rails `+12V`/`+15V`/`+3V3` routés** (détail et largeurs :
`docs/architecture.md`, § F1.3-a).

Validation :

- **Vérification du principal** (`kicad-cli pcb drc --schematic-parity`) : non-connectés 216 → 186,
  aucun restant sur `SW`, `BUCK_VIN`, `FB`, `BST`, `VCC`, `SS`, `RON_SD`, `+12V`, `+15V`, `+3V3` ;
  violations 86 + 21 silk uniquement ; parité 3 ; 124 / 119 / 1.
- Diff `.kicad_pcb` purement additif (672 insertions, 0 suppression) : aucune empreinte déplacée.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Bande `x` 100..155 réservée à l'analogique bas niveau** ; zone d'interdiction composants
  `x` [280 ; 291], `y` [160 ; 190] (pistes autorisées).
- **Sérigraphie à traiter en bloc** ; **masses au plan de F1.4** ; `+3V3` long en `B.Cu` à
  reprendre si F1.4 réserve `B.Cu` au `GND`.
- **Découplage `BUCK_VIN` absent après `D3`** : à trancher en F1.6, sans bloquer F1.3.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `4ecca372ee549a15e6c5ef0c692b1fad`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md` — décisions E1, E1.5, F1.2-b, F1.2-c, F1.3-a

## NEXT ACTION

**F1.3-b — router l'entrée de puissance et la protection `LM5069`** : shunt `R306` en Kelvin sur
les bords intérieurs, chemin de puissance dimensionné au courant total, 6 cm² réservés au drain de
`Q301`. Validation : 39 résolus (186 → 147), DRC sans `clearance`/`shorting_items`/`track_dangling`,
parité 3, largeurs de puissance justifiées, 124 empreintes, 119 `(units`.
