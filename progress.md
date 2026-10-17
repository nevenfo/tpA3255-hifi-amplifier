# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1, F1.2, F1.3-a, -b et -c closes ; F1.3-d à ouvrir.** 332 segments, **33 vias**.
DRC courant : **107 violations toutes de sérigraphie** (86 `silk_overlap`, 21 `silk_over_copper`),
**113 non-connectés** (78 `GND` + signal 26 + contrôle 9),
`schematic_parity` = **3**, aucune `clearance`, `shorting_items`, `track_dangling` ni
`courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.3-d — signal analogique (26 chevelus), précédé du reclassement `±VSE` en `ANALOG`.**

## Dernière tâche validée

**F1.3-c = PASS — alimentations analogiques routées**, `VMID` égal au centième entre `U4` et
`U5` (`docs/architecture.md`, § F1.3-c).

Validation :

- **Vérification du principal** (`kicad-cli pcb drc --schematic-parity`) : 147 → 113, restent
  `GND` 78 + signal + contrôle ; violations 86 + 21 silk uniquement ; parité 3 ; 124 / 119.
- Diff `.kicad_pcb` purement additif ; aucune exception `.kicad_dru` ajoutée.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Bande `x` 100..155 réservée à l'analogique bas niveau** ; zone d'interdiction composants
  `x` [280 ; 291], `y` [160 ; 190] (pistes autorisées).
- **Sérigraphie à traiter en bloc** ; **masses au plan de F1.4** ; `+3V3` long en `B.Cu` à
  reprendre si F1.4 réserve `B.Cu` au `GND`.
- **`±VSE`/`±VSE_R` sont des sorties d'AOP** : leur classe `PWR_OUT` est erronée, reclassement
  en `ANALOG` en tête de F1.3-d.
- **Courant OPA1612 supposé (20 mA/boîtier)**, non tiré de la fiche technique.
- **Découplage `BUCK_VIN` absent après `D3`** : à trancher en F1.6, sans bloquer F1.3.
- **Exceptions `.kicad_dru` à 0,15 mm limitées aux cours de `U6`, `U8`, `U1`** ; `PVDD` en
  `B.Cu` 7 mm, à doubler par zone en F1.4.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `a303a88f9c90e32c0935da4bdf4da3e0`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md` — décisions E1, E1.5, F1.2-b, F1.2-c, F1.3-a, -b, -c

## NEXT ACTION

**F1.3-d — reclasser `±VSE`/`±VSE_R` en `ANALOG` (DRC inchangé), puis router le signal
analogique** (`INPUT_A`–`D`, `INV1`–`4_NEG`, `RCA_L/R`, `VOL_L/R(_IN)`) dans la bande `x` 100..155,
symétrie (0 ; +40) conservée, traversées nord-sud en `B.Cu`. Validation : 26 résolus (113 → 87),
écart de longueur entre voies chiffré, DRC sans `clearance`/`shorting_items`/`track_dangling`,
parité 3, 124 empreintes, 119 `(units`.
