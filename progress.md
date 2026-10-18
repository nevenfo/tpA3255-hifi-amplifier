# PROGRESS

## Phase actuelle

Phase F — Routage. F1.1 à F1.3, F1.6, F1.7 et F1.4-a validées ; F1.4-b débloquée.
Branche `ai/f1.4-plane-strategy`.

## Tâche actuelle

F1.4-b — appliquer O3 : transfert `B.Cu` → `In2.Cu`, zones `/GND` `In1.Cu`/`B.Cu`, `PVDD`,
couture, vias thermiques `U1`/`Q301`, remplissage.

## Dernière tâche validée

F1.7 : `U1`/`L1` avec les bonnes empreintes, buck auxiliaire replacé et re-routé.

Validation :
- DRC sans `clearance`/`shorting_items`/`track_dangling`/`courtyards_overlap`/`hole_clearance`/`solder_mask_bridge`.
- Non-connectés 76, tous `/GND` ; parité 3 ; `pad_prop_heatsink` = 1 ; 124 `(units`.
- Pistes hors zone identiques par UUID. PCB MD5 `050bbce302a6f410e3a3378507d0cff3`.

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

Décision utilisateur attendue pour F1.4-b4 seulement : cuivre de drain `Q301` à 4,6 cm² par couche
au lieu des 9 cm² de l'étude (Tj ≈ 121 °C estimée). Options : accepter et mesurer en H2, agrandir
la zone, ou ajouter un dissipateur. F1.4-b1 à b3 n'en dépendent pas. Éditeur PCB fermé (seul le gestionnaire de projet est ouvert) : le rouvrir seul avant toute
écriture IPC, recompter les `(units` (124) après chaque déplacement MCP.

## Fichiers / zones utiles

- `plan.md` F1.4-b et F1.7 (`+15V` en `In2.Cu` à relire).
- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

F1.4-b1 — transférer 112 segments `B.Cu` en `In2.Cu` (garder les 4 de `C310`/`C311` et les 56 vias),
puis valider selon `plan.md` F1.4-b1.
