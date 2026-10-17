# PROGRESS

## Phase actuelle

Phase F — Routage. F1.1 à F1.3 et F1.6 validées ; F1.4 en étude.
Aucune modification KiCad pendant F1.4-a. Branche `ai/f1.4-plane-strategy`.

## Tâche actuelle

F1.4-a — étude réalisée ; arbitrage utilisateur indispensable avant application.
Rapport et proposition dans `docs/plane-strategy-f1.4-a.md` et `docs/architecture.md`.

## Dernière tâche validée

F1.6 — découplage d'entrée de `U1` ajouté ; baseline recontrôlée le 2026-10-17.

Validation :
- `kicad-cli 10.0.6` : 123 violations de sérigraphie, 78 non-connectés tous `GND`,
  parité 3 ; aucune `clearance`, `shorting_items`, `track_dangling` ni `courtyards_overlap`.
- ERC : 16 violations (baseline). PCB MD5 `d1c92ea037bde88a81731b002599e651`.
- Continuité structurelle conforme ; dernier état initial `7ab12e8` présent sur `origin/main`.

## Décisions actives

- Règles durables et décisions de placement : `docs/kicad-operations.md`, `docs/architecture.md`.
- `OUT` à 5 A sur `F.Cu`, sans via ; bande analogique `x` 100..155 réservée.
- E1.5 n'est pas révisée tant que l'utilisateur n'a pas choisi la stratégie des plans.
- O3 recommandée : transférer `B.Cu` vers `In2.Cu`, deux plans de masse continus ;
  chiffrage et limites dans l'étude. Recommandation, pas décision acquise.
- Sérigraphie à traiter en bloc ; dimensionnement thermique `U1`/`Q301` à établir avant
  les matrices de vias de F1.4-b. Les retours MKP doivent rester près de `U6`.
- F1.4-b doit aussi reprendre les retours `COUT`–EP de `U1`, `AVDD`/`DVDD`
  et doubler `PVDD` par zone. Interdiction composants `x` [280 ; 291], `y` [160 ; 190].

## Blocage actif

Arbitrage produit : adopter O3 implique de remplacer la découpe de masse E1.5 par
une masse continue. Ne pas appliquer F1.4-b avant décision. L'éditeur PCB était fermé
à la reprise ; vérifier son état avant toute écriture IPC.

## Fichiers / zones utiles

- `docs/plane-strategy-f1.4-a.md` — options, mesures et préconditions.
- `docs/architecture.md` — proposition et état de la décision.
- `docs/kicad-operations.md` — à lire avant manipulation.
- `HifiAmp_TPA3255.kicad_pcb` — PCB inchangé.

## NEXT ACTION

F1.4-a — obtenir l'arbitrage utilisateur sur O3 et le consigner dans
`docs/architecture.md` ; ensuite seulement clore l'étude et commencer F1.4-b.
