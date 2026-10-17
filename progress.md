# PROGRESS

## Phase actuelle

Phase F — Routage. F1.1 à F1.3, F1.6 et F1.4-a validées ; F1.4-b à démarrer.
Branche `ai/f1.4-plane-strategy`.

## Tâche actuelle

F1.7 — corriger les empreintes de `U1` (`DPR0010A` 4 × 4 mm au pas de 0,8 mm, au lieu d'un
WSON 2,5 mm au pas de 0,5 mm) et de `L1` (WE-PD 1050 de 10 × 10 mm, au lieu d'un WE-PD2 d'environ 6 mm).
Diagnostic `U1` fait (voir `plan.md` F1.7) ; impact de `L1` à mesurer. F1.4-b bloquée jusque-là.

## Dernière tâche validée

F1.4-b, préconditions thermiques : données fabricant `U1`/`Q301` établies dans
`docs/plane-strategy-f1.4-a.md`. Résultats : `U1` large marge ; `Q301` à 101 °C dans le pire cas
avec cuivre de drain, 160 °C sans.

Validation :
- Sources lues : SNVS376F (rév. mai 2016) et Infineon rév. 2.0 du 2021-05-10.
- Calculs consignés avec leurs formules. PCB inchangé, MD5 `d1c92ea037bde88a81731b002599e651`.

## Décisions actives

- Règles durables et décisions de placement : `docs/kicad-operations.md`, `docs/architecture.md`.
- `OUT` à 5 A sur `F.Cu`, sans via ; bande analogique `x` 100..155 réservée.
- O3 : `F.Cu` composants/signaux, `In1.Cu` `/GND` continu, `In2.Cu` mixte (pistes
  ex-`B.Cu`), `B.Cu` `/GND` continu ; aucune fente sous `INPUT_A`–`D`, couture `GND` à la
  frontière fonctionnelle et près de `U6`, retours MKP `C321`–`C324` vers `U6` hors aire analogique.
- Zone `PVDD` sur `In2.Cu` ou `F.Cu` (plus sur `In1.Cu`) ; reprendre aussi les retours
  `COUT`–EP de `U1` et `AVDD`/`DVDD`. Aucun via sous `U6`.
- Cuivre de drain `Q301` sur `F.Cu` + `In2.Cu` (plus `B.Cu`) ; `U1` : vias 2 × 3 de 0,6/0,3 mm
  dans le pad exposé, bouchés côté `B.Cu`.
- Sérigraphie à traiter en bloc. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Aucun. Vérifier l'état de l'éditeur PCB avant toute écriture IPC.

## Fichiers / zones utiles

- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/architecture.md` — décision O3.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

F1.7 — par `kicad-control`, en lecture seule : impact de l'empreinte WE-PD 1050 de `L1`
(courtyards, pistes, vias sous le composant) et proposition de placement commune `U1`/`L1`.
