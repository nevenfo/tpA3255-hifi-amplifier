# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1 à F1.3 closes : tout est routé sauf `GND`.** 540 segments, **54 vias**.
DRC courant : **107 violations toutes de sérigraphie** (86 `silk_overlap`, 21 `silk_over_copper`),
**78 non-connectés, tous `GND`** (plan de F1.4), `schematic_parity` = **3**, aucune `clearance`,
`shorting_items`, `track_dangling` ni `courtyards_overlap`. **124 empreintes, 119 blocs `(units`,
`pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.6 — découplage d'entrée du buck `U1`, avant les zones de F1.4.**

## Dernière tâche validée

**F1.3-e = PASS — contrôle routé, F1.3 close** (`docs/architecture.md`, § F1.3-e).

Validation :

- **Vérification du principal** (`kicad-cli pcb drc --schematic-parity`) : 87 → 78, tous `GND` ;
  violations 86 + 21 silk uniquement ; parité 3 ; 124 / 119.
- Comparaison par UUID : seuls les 5 éléments `INPUT` déclarés (3 vias 0,8/0,4, 2 amorces) sont
  retirés, aucun autre modifié ; écart `INPUT_C`/`D` 0,10 mm.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Bande `x` 100..155 réservée à l'analogique bas niveau** ; zone d'interdiction composants
  `x` [280 ; 291], `y` [160 ; 190] (pistes autorisées).
- **Sérigraphie à traiter en bloc** ; **masses au plan de F1.4** ; `+3V3` long en `B.Cu` à
  reprendre si F1.4 réserve `B.Cu` au `GND`.
- **Écart de 40 mm entre voies sur `INPUT` accepté** (0,27 ns, négligeable en audio).
- **Courant OPA1612 supposé (20 mA/boîtier)**, non tiré de la fiche technique.
- **F1.6 avant F1.4** : un éventuel condensateur `BUCK_VIN` se place avant le remplissage des zones.
- **Exceptions `.kicad_dru` à 0,15 mm limitées aux cours de `U6`, `U8`, `U1`** ; `PVDD` en
  `B.Cu` 7 mm, à doubler par zone en F1.4.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `285764818e9cd9d068d2bb36fefd8976`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md` — décisions E1, E1.5, F1.2-b, F1.2-c, F1.3-a à -e

## NEXT ACTION

**F1.6 — vérifier au schéma le découplage d'entrée du buck `U1` (LM5010)** : capacité présente
entre `D3` et `U1.10` (`BUCK_VIN`) ou non, exigence de la fiche technique. Validation : capacité
locale au schéma et au PCB, ou absence justifiée par la fiche ; si un ajout est requis, décision
de placement avant F1.4.
