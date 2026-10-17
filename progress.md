# PROGRESS

## Phase actuelle

Phase F — Routage. F1.1 à F1.3, F1.6 et F1.4-a validées ; F1.4-b à démarrer.
Branche `ai/f1.4-plane-strategy`.

## Tâche actuelle

F1.4-b — appliquer O3 : transfert `B.Cu` → `In2.Cu`, plans `/GND` continus sur
`In1.Cu` et `B.Cu`, zones, couture et vias thermiques.

## Dernière tâche validée

F1.4-a — stratégie des plans : O3 retenue par l'utilisateur le 2026-10-17.

Validation :
- Décision consignée dans `docs/architecture.md` (section F1.4-a) et `plan.md`.
- Baseline PCB inchangée : MD5 `d1c92ea037bde88a81731b002599e651`, DRC 123 violations de
  sérigraphie, 78 non-connectés tous `GND`, parité 3 ; ERC 16 violations.

## Décisions actives

- Règles durables et décisions de placement : `docs/kicad-operations.md`, `docs/architecture.md`.
- `OUT` à 5 A sur `F.Cu`, sans via ; bande analogique `x` 100..155 réservée.
- O3 : `F.Cu` composants/signaux, `In1.Cu` `/GND` continu, `In2.Cu` mixte (pistes
  ex-`B.Cu`), `B.Cu` `/GND` continu ; aucune fente sous `INPUT_A`–`D`, couture `GND` à la
  frontière fonctionnelle et près de `U6`, retours MKP `C321`–`C324` vers `U6` hors aire analogique.
- Zone `PVDD` sur `In2.Cu` ou `F.Cu` (plus sur `In1.Cu`) ; reprendre aussi les retours
  `COUT`–EP de `U1` et `AVDD`/`DVDD`. Aucun via sous `U6`.
- Matrices thermiques `U1`/`Q301` seulement après données fabricant (NEEDS_DATA).
- Sérigraphie à traiter en bloc. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Aucun. Vérifier l'état de l'éditeur PCB avant toute écriture IPC.

## Fichiers / zones utiles

- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/architecture.md` — décision O3.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

F1.4-b — réunir les recommandations thermiques PCB fabricant de `LM5010ASD` et
`IPP330P10NM` (vias, cuivre, puissance pire cas) et les consigner dans
`docs/plane-strategy-f1.4-a.md`, avant toute écriture KiCad.
