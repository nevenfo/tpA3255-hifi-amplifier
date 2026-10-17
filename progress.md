# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1, F1.2, F1.3-a et F1.3-b closes ; F1.3-c à ouvrir.** 253 segments, **20 vias**.
DRC courant : **107 violations toutes de sérigraphie** (86 `silk_overlap`, 21 `silk_over_copper`),
**147 non-connectés** (78 `GND` + analogique 34 + signal 26 + contrôle 9),
`schematic_parity` = **3**, aucune `clearance`, `shorting_items`, `track_dangling` ni
`courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.3-c — alimentations analogiques (34 chevelus).**

## Dernière tâche validée

**F1.3-b = PASS — entrée de puissance, protection `LM5069` et `ISEN` routées** (largeurs et
zone réservée de `Q301` : `docs/architecture.md`, § F1.3-b).

Validation :

- **Vérification du principal** (`kicad-cli pcb drc --schematic-parity`) : 186 → 147, aucun net
  F1.3-b restant ; violations 86 + 21 silk uniquement ; parité 3 ; 124 / 119.
- Règles `.kicad_dru` d'échappement `U8` et `U1` (0,15 mm) prouvées sélectives par l'agent.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Bande `x` 100..155 réservée à l'analogique bas niveau** ; zone d'interdiction composants
  `x` [280 ; 291], `y` [160 ; 190] (pistes autorisées).
- **Sérigraphie à traiter en bloc** ; **masses au plan de F1.4** ; `+3V3` long en `B.Cu` à
  reprendre si F1.4 réserve `B.Cu` au `GND`.
- **Découplage `BUCK_VIN` absent après `D3`** : à trancher en F1.6, sans bloquer F1.3.
- **Exceptions `.kicad_dru` à 0,15 mm limitées aux cours de `U6`, `U8`, `U1`** ; `PVDD` en
  `B.Cu` 7 mm, à doubler par zone en F1.4.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `adba0c270371ba0f9cba6cb283413d0b`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md` — décisions E1, E1.5, F1.2-b, F1.2-c, F1.3-a, F1.3-b

## NEXT ACTION

**F1.3-c — router les alimentations analogiques** (`±VSE`, `±VSE_R`, `L/R_POS/NEG_AC`,
`+12V-OA`, `VMID`), `VMID` équidistant de `U4`/`U5`. Validation : 34 résolus (147 → 113), DRC sans
`clearance`/`shorting_items`/`track_dangling`, parité 3, 124 empreintes, 119 `(units`.
