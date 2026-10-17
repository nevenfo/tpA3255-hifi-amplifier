# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1 à F1.3 et F1.6 closes : tout est routé sauf `GND`.** 598 pistes et vias.
DRC courant : **123 violations toutes de sérigraphie** (93 `silk_overlap`, 30 `silk_over_copper`),
**78 non-connectés, tous `GND`** (plan de F1.4), `schematic_parity` = **3**, aucune `clearance`,
`shorting_items`, `track_dangling` ni `courtyards_overlap`. **126 empreintes, 124 blocs `(units`,
`pad_prop_heatsink` = 1.** ERC : 16 (10 `endpoint_off_grid`, 6 `lib_symbol_mismatch`), baseline.

## Tâche actuelle

**F1.4 — plans, zones et vias thermiques.**

## Dernière tâche validée

**F1.6 = PASS — découplage d'entrée du buck ajouté selon SNVS376F** : `C327` 0,1 µF et `C328`
2,2 µF 100 V au plus près de `U1`, masse directe sur l'EP.

Validation :

- **Vérification du principal** : DRC parité 3, 78 non-connectés tous `GND`, aucune
  `clearance`/`shorting_items`/`courtyards_overlap` ; ERC identique (16) ; par UUID seule
  `e6052374` retirée, aucune empreinte déplacée, `H1`/`H2` présentes.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Bande `x` 100..155 réservée à l'analogique bas niveau** ; zone d'interdiction composants
  `x` [280 ; 291], `y` [160 ; 190] (pistes autorisées).
- **Sérigraphie à traiter en bloc** ; **masses au plan de F1.4** ; `+3V3` long en `B.Cu` à
  reprendre si F1.4 réserve `B.Cu` au `GND`.
- **Écart de 40 mm entre voies sur `INPUT` accepté** (0,27 ns, négligeable en audio).
- **Courant OPA1612 supposé (20 mA/boîtier)**, non tiré de la fiche technique.
- **Retour `COUT`↔EP de `U1` (≈ 20 mm) et `AVDD`/`DVDD` longs (F1.1) à reprendre par le plan
  `GND` de F1.4.**
- **Exceptions `.kicad_dru` à 0,15 mm limitées aux cours de `U6`, `U8`, `U1`** ; `PVDD` en
  `B.Cu` 7 mm, à doubler par zone en F1.4.

## Blocage actif

Aucun. L'éditeur de PCB est fermé : le rouvrir (geste GUI, `desktop-control`) avant toute
écriture MCP.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `d1c92ea037bde88a81731b002599e651`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md` — décisions E1, E1.5, F1.2-b, F1.2-c, F1.3-a à -e

## NEXT ACTION

**F1.4 — définir la stratégie de plans au fichier** (affectation `In1.Cu`/`In2.Cu`/`F.Cu`/`B.Cu`
aux zones `GND`, `PVDD`, `PVDD_FUSED` de `Q301`, séparation analogique/puissance, vias thermiques
sous `U6`/`U1`/`Q301`) puis la découper en unités F1.4-a… dans `plan.md`. Validation : chaque
unité porte objectif, nets, couches et critère DRC (non-connectés `GND` → 0).
