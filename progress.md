# PROGRESS

## Phase actuelle

Phase G — Validation PCB. Phase F validée, PR #1 ouverte vers `main` (non fusionnée).
Branche `ai/g1-drc-review`, créée depuis `ai/f1.4-plane-strategy`.

## Tâche actuelle

G1.3 — clôture : rejouer le DRC, l'archiver et mettre à jour les statuts C1 à C6 dans la revue.

## Dernière tâche validée

G1.3-b : `C13` à 1 µF en `B.Cu` près de `U1.9`, `C329` 1 µF/100 V près du drain de `Q302`.

Validation :
- DRC : seuls `silk_overlap` 89 et `silk_over_copper` 25 restent ; 0 non-connecté, parité 3.
- 127 empreintes, 125 `(units`, ERC 16 avertissements connus. PCB MD5 `ea6ea4b3695946c52b2deca67f622fd4`.

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
- F1.4-b, choix validés par l'utilisateur : plans en retrait de 0,5 mm, pas de zone `PVDD` en `In2.Cu`, règle DRU interdisant les vias sous `L1`/`U6`, couture par pad. Drain `Q301` agrandi (b4) ; Tj à mesurer en H2.
- Vias dans les pads `C303.2`/`C206.2` et dans l'EP de `U1` : à faire boucher et capper (G1).
- G1.3 (utilisateur) : C1 en option (b) ; `C13` à 1 µF en `B.Cu` ; ajout d'un 1 µF/100 V sur `/PVDD_SENSE` ; pas de 10 µF sur `U3`.
- Sérigraphie à traiter en bloc. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Aucun. Éditeur PCB fermé : travail au fichier, remplissage par `kicad-cli pcb drc --refill-zones --save-board` ; recompter les 125 `(units`.

## Fichiers / zones utiles

- `docs/drc/design-review-g1.2.md` — constats B1, C1 à C6, A1 à A11 et `NEEDS_DATA`.
- `docs/drc/README.md` — DRC archivé.
- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

G1.3 — rejouer le DRC (`docs/drc/drc-g1.3.json`), mettre à jour `docs/drc/README.md` et les statuts C1 à C6 de
`docs/drc/design-review-g1.2.md`, puis cocher G1.3.
