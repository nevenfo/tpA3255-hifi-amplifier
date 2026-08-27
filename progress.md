# PROGRESS

## Phase actuelle

Phase B — Schéma KiCad.

## Tâche actuelle

B1.1 — Valider le squelette de projet via MCP et préparer le premier bloc schématique.

## Dernière tâche validée

A1.6 — Architecture initiale revue et inconnues critiques enregistrées.

Validation :
- Sources TI primaires référencées dans `docs/architecture.md`.
- BTL stéréo, niveaux, rails, contrôle, LC, stack-up et protections initiales définis.
- Interface corrigée selon l’EVM : deux inverseurs `−1` en cascade par canal, `+12V-OA`, `VMID=6 V`.
- Huit `NEEDS_DATA` explicites interdisent encore un statut fabricable mais pas une capture marquée.
- `launch_kicad_ui`, `open_project` et `check_kicad_ui` ont validé l’UI et l’IPC live.

## Décisions actives

- Toutes les opérations KiCad directes passent exclusivement par `kicad-control` et le MCP privé.
- Aucun PCB avant gate ERC explicitement validé.
- Les composants non figés restent marqués et sans prétention de fabricabilité.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`
- `HifiAmp_TPA3255.kicad_sch`
- `HifiAmp_TPA3255.kicad_pcb`
- `docs/architecture.md`
- `reports/calculs-initiaux.md`
- `reports/mcp-initial-state.md`

## NEXT ACTION

B1.1 — Inspecter le squelette ouvert via MCP, confirmer schéma/PCB vides et enregistrables, puis préparer B1.2 alimentation/connectique sans commencer le PCB.
