# PROGRESS

## Phase actuelle

Phase H. H1.1–H1.3 faites (`docs/review/h1-electrical.md`). PR #1 et #2 ouvertes, non fusionnées.
Branche `ai/h1-electrical-review`.

## Tâche actuelle

H1.4 — corriger le défaut bloquant B1 (chaîne d'entrée couplée en DC) et trancher A1 (`C7`) et A2 (réseau RC de sortie).

## Dernière tâche validée

H1.1–H1.3 : revue électrique par le calcul.

Validation :
- Rapport `docs/review/h1-electrical.md`, avec classement des preuves et NEEDS_DATA.
- B1 confirmé sur la netlist (`kicad-cli`) : `VOL_L` = `J4.2`, `R101.1` ; `J4.3` sur `/GND` ; `VMID` sur `U4.3`/`U4.5`.

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
- G1.3 (utilisateur) : C1 en option (b) ; `C13` à 1 µF en `B.Cu` ; ajout d'un 1 µF/100 V sur `/PVDD_SENSE` ; pas de 10 µF sur `U3`.
- Fabrication (utilisateur) : PCBWay, IPC-4761 type VII pour les vias dans une pastille, 1 oz sur toutes les couches. Notes : `docs/drc/dfm-g1.4.md` §5.
- Sérigraphie traitée en G1.4-a ; seuls restent les 4 repères B2 de `L301`–`L304`. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Aucun.

Notes : Éditeur PCB fermé : travail au fichier, remplissage par `kicad-cli pcb drc --refill-zones --save-board` ; recompter les 125 `(units`.

## Fichiers / zones utiles

- `docs/drc/design-review-g1.2.md` — constats B1, C1 à C6, A1 à A11 et `NEEDS_DATA`.
- `docs/drc/README.md` — DRC archivé.
- `docs/plane-strategy-f1.4-a.md` — mesures, préconditions F1.4-b.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb`.

## NEXT ACTION

H1.4 — F8 par GUI (ajout de `C111`, `C211`, `C7`, `C330`–`C337`, `R311`–`R314`, posés hors carte), puis placement et routage
dans le fichier, remplissage et DRC (validation : `plan.md` H1.4).
