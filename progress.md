# PROGRESS

## Phase actuelle

Phase F — Routage. F1.1 à F1.3, F1.6, F1.7 et F1.4-a validées ; F1.4-b débloquée.
Branche `ai/f1.4-plane-strategy`.

## Tâche actuelle

F1.4-b — appliquer O3 : transfert `B.Cu` → `In2.Cu`, zones `/GND` `In1.Cu`/`B.Cu`, `PVDD`,
couture, vias thermiques `U1`/`Q301`, remplissage.

## Dernière tâche validée

F1.4-b1 : 112 segments `B.Cu` passés en `In2.Cu`, avec 2 copies `In2.Cu` en plus pour `PVDD` vers `C310`/`C311`.

Validation :
- DRC sans `clearance`/`shorting_items`/`track_dangling`/`courtyards_overlap`/`hole_clearance`.
- Non-connectés 76, tous `/GND` ; parité 3 ; 124 `(units` ; `pad_prop_heatsink` = 1.
- Couches : `F.Cu` 421, `In2.Cu` 116, `B.Cu` 4 segments. PCB MD5 `fe6ee0dd2738c8de3cab2819ea089624`.

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
- F1.4-b, choix validés par l'utilisateur : plans en retrait de 0,5 mm, pas de zone `PVDD` en `In2.Cu`, règle DRU interdisant les vias sous `L1`/`U6`, couture par pad. Drain `Q301` : agrandir à ≥ 6 cm² par couche.
- Sérigraphie à traiter en bloc. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Aucun. Éditeur PCB fermé : travail au fichier ou réouverture seule avant IPC ; recompter les 124 `(units`.

## Fichiers / zones utiles

- `plan.md` F1.4-b et F1.7 (`+15V` en `In2.Cu` à relire).
- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

F1.4-b2 — pour les 6 pads `/GND` sans via (`U6.3/4/33/34`, `C303.2`, `C206.2`), chiffrer les options
(reprise locale de pistes, via dans le pad, liaison vers un pad `GND` voisin), puis appliquer la moins risquée.
