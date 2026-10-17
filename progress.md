# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1, F1.2, F1.3-a à -d closes ; F1.3-e à ouvrir.** 472 segments, **50 vias**.
DRC courant : **107 violations toutes de sérigraphie** (86 `silk_overlap`, 21 `silk_over_copper`),
**87 non-connectés** (78 `GND` + contrôle 9),
`schematic_parity` = **3**, aucune `clearance`, `shorting_items`, `track_dangling` ni
`courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.3-e — contrôle (9 chevelus), dernière unité de F1.3.**

## Dernière tâche validée

**F1.3-d = PASS — signal analogique routé, `±VSE` reclassés en `ANALOG`** (longueurs et écarts :
`docs/architecture.md`, § F1.3-d).

Validation :

- **Vérification du principal** (`kicad-cli pcb drc --schematic-parity`) : 113 → 87, restent
  `GND` 78 + contrôle 9 ; violations 86 + 21 silk uniquement ; parité 3 ; 124 / 119.
- Les 365 segments et vias antérieurs sont identiques par UUID ; `.kicad_pro` : 4 affectations.
- Paires `INPUT` appariées à 0,45 et 0,00 mm.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Bande `x` 100..155 réservée à l'analogique bas niveau** ; zone d'interdiction composants
  `x` [280 ; 291], `y` [160 ; 190] (pistes autorisées).
- **Sérigraphie à traiter en bloc** ; **masses au plan de F1.4** ; `+3V3` long en `B.Cu` à
  reprendre si F1.4 réserve `B.Cu` au `GND`.
- **Écart de 40 mm entre voies sur `INPUT` accepté** (0,27 ns, négligeable en audio).
- **Courant OPA1612 supposé (20 mA/boîtier)**, non tiré de la fiche technique.
- **Découplage `BUCK_VIN` absent après `D3`** : à trancher en F1.6, sans bloquer F1.3.
- **Exceptions `.kicad_dru` à 0,15 mm limitées aux cours de `U6`, `U8`, `U1`** ; `PVDD` en
  `B.Cu` 7 mm, à doubler par zone en F1.4.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `a6e263d7fe8bacedd7bfaaa237b4cf88`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md` — décisions E1, E1.5, F1.2-b, F1.2-c, F1.3-a à -d

## NEXT ACTION

**F1.3-e — router le contrôle** (`RESET`, `RESET-SW`, `RESET_RC`, `FAULT`, `CLIP_OTW`, `OC_ADJ`,
`FREQ_ADJ`, `C_START`), sorties de `U6` resserrées par les vias `INPUT` (`x` 290–294,3).
Validation : 9 résolus (87 → 78, il ne reste que `GND`), DRC sans
`clearance`/`shorting_items`/`track_dangling`, parité 3, 124 empreintes, 119 `(units`.
