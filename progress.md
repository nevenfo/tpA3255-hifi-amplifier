# PROGRESS

## Phase actuelle

Phase B — Schéma KiCad.

## Tâche actuelle

B1.1 — Valider placement et persistance par MCP.

## Dernière tâche validée

A1.6 — Architecture initiale revue et inconnues critiques enregistrées.

Validation :
- Architecture TI documentée dans `docs/architecture.md` et calculs dans `reports/calculs-initiaux.md`.
- Chaîne auxiliaire et MPN vérifiés dans `docs/power-block.md`.
- KiCad 10.0.3 installé ; `api.enable_server=true`.
- Symbole custom `LM5010ASD` créé par MCP avec 11 connexions TI et bibliothèque enregistrée comme `HifiAmp_TPA3255_Local`.

## Décisions actives

- Toute édition schéma/PCB reste réservée à `kicad-control`/MCP ; aucun fichier `.kicad_sch` ou `.kicad_pcb` ne sera modifié directement.
- Aucun PCB avant gate ERC PASS.
- Sept `NEEDS_DATA` restent centralisés dans `docs/architecture.md`.

## Blocage actif

Le placeur MCP ne résout pas la bibliothèque projet après trois cycles complets. Erreurs observées selon le contexte IPC : `no_project_path_found`, `Library 'HifiAmp_TPA3255_Local' not found`; `save_project` a aussi produit `GetOpenDocuments (AS_UNHANDLED)`. KiCad voit la bibliothèque après enregistrement GUI, mais la stack ne permet pas encore de placer/persister U1. Ne pas contourner par édition directe ni modifier le MCP dans ce projet.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym`, `sym-lib-table`
- `docs/architecture.md`, `docs/power-block.md`
- `reports/mcp-initial-state.md`, `reports/benchmark-mcp.md`

## NEXT ACTION

B1.1 — Après correction externe du résolveur/attachement IPC de kicad-control, ouvrir le projet, placer `HifiAmp_TPA3255_Local:LM5010ASD` comme U1 via MCP, relire puis prouver sa persistance avant B1.2.
