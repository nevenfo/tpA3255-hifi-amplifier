# PROGRESS

## Phase actuelle

Phase F — Routage. F1.1 à F1.3, F1.6 et F1.4-a validées ; F1.4-b bloquée par F1.7.
Branche `ai/f1.4-plane-strategy`.

## Tâche actuelle

F1.7 — empreintes de `U1` (`WSON-10-1EP_4x4mm_P0.8mm_EP2.6x3mm`) et `L1`
(`L_Wuerth_WE-PD-1050_10x10x5mm`) propagées au PCB. Reste : replacer, puis re-router la zone du buck
auxiliaire (positions cibles et pistes à refaire : `plan.md` F1.7).

## Dernière tâche validée

F1.7, propagation F8 : seules `U1`/`L1` changent de lib_id ; `U1.11` garde `pad_prop_heatsink`.

Validation :
- Géométrie : `U1` pas de 0,8 mm, pad exposé de 2,6 × 3 mm ; `L1` pads 2,0 × 3,5 mm à ± 4,5 mm.
- 544 segments et 54 vias identiques par UUID ; parité = 3 écarts connus.
- PCB MD5 `84eea39742fab9ee92628179f1048da5`.
- DRC transitoire attendu (anciennes pistes sous les nouveaux pads) : `shorting_items` 11, `clearance` 4,
  `hole_clearance` 1, `courtyards_overlap` 4, `solder_mask_bridge` 15, `track_dangling` 5,
  `silk_over_copper` 32, non-connectés 83.

## Décisions actives

- Règles durables et décisions de placement : `docs/kicad-operations.md`, `docs/architecture.md`.
- `OUT` à 5 A sur `F.Cu`, sans via ; bande analogique `x` 100..155 réservée.
- O3 : `F.Cu` composants/signaux, `In1.Cu` `/GND` continu, `In2.Cu` mixte (pistes
  ex-`B.Cu`), `B.Cu` `/GND` continu ; aucune fente sous `INPUT_A`–`D`, couture `GND` à la
  frontière fonctionnelle et près de `U6`, retours MKP `C321`–`C324` vers `U6` hors aire analogique.
- Zone `PVDD` sur `In2.Cu` ou `F.Cu` (plus sur `In1.Cu`) ; reprendre aussi les retours
  `COUT`–EP de `U1` et `AVDD`/`DVDD`. Aucun via sous `U6` ni sous `L1`.
- Cuivre de drain `Q301` sur `F.Cu` + `In2.Cu` (plus `B.Cu`) ; `U1` : vias 2 × 3 de 0,6/0,3 mm
  dans le pad exposé, bouchés côté `B.Cu`.
- Sérigraphie à traiter en bloc. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Aucun. L'éditeur de schéma est ouvert à côté de l'éditeur PCB : l'IPC voit 2 instances. Fermer
l'éditeur de schéma avant toute écriture IPC.

## Fichiers / zones utiles

- `plan.md` F1.7 — positions cibles, pistes/vias à refaire, validation.
- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

F1.7 — fermer l'éditeur de schéma, replacer `L1`/`U1`/`C327`/`C328`/`D1`/`D3` aux positions cibles,
re-router la zone (33 segments, 2 vias, `BST`) ; valider selon `plan.md` F1.7.
