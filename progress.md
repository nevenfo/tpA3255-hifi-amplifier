# PROGRESS

## Phase actuelle

Phase F — Routage validée (F1.1–F1.8), PR vers `main` à ouvrir. Phase G — Validation PCB à démarrer.
Branche `ai/f1.4-plane-strategy`.

## Tâche actuelle

G1.1 — DRC final archivé.

## Dernière tâche validée

F1.8 : nettoyage du routage, ce qui clôt la phase F.

Validation :
- DRC : seuls `silk_overlap` 90 et `silk_over_copper` 26 restent ; 0 non-connecté, 0 `via_dangling`, parité 3.
- Longueurs `OUT` inchangées, `INPUT` à Δ ≤ 0,14 mm ; zones identiques. PCB MD5 `70e315fad1997ce7b3616006aac5cd1b`.

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
- Sérigraphie à traiter en bloc. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Aucun. Éditeur PCB fermé : travail au fichier, remplissage par `kicad-cli pcb drc --refill-zones --save-board` ; recompter les 124 `(units`.

## Fichiers / zones utiles

- `plan.md` G1 ; points reportés en G1 : vias dans les pads `C303.2`/`C206.2`/EP `U1`, broches `GND` du TPA3255 (SLASEG), écart de 40 mm entre les paires `INPUT` A–B et C–D, dog-bones `U6` à la limite, sérigraphie.
- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

G1.1 — exécuter le DRC final (avec parité et `--severity-all`) et archiver le rapport dans `docs/`,
après ouverture de la PR de la phase F vers `main`.
