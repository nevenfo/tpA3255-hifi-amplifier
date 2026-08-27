# État initial KiCad MCP — 2026-08-27

## État live

- Serveur MCP Konnect : joignable.
- Application KiCad : `kicad_ui_running=false`.
- IPC KiCad : `ipc_responsive=false`.
- Erreur IPC exacte : `KiCAD IPC is not reachable...`.
- Projet KiCad dans le workspace : aucun `.kicad_pro`, `.kicad_sch` ou `.kicad_pcb`.

Le serveur MCP répond donc réellement, mais aucune connexion à une instance KiCad active n’est encore prouvée.

## Outils exposés directement

`changes_since`, `create_project`, `find_capabilities`, `get_active_toolsets`, `get_effective_config`, `get_project_info`, `get_recent_calls`, `kicad_agent`, `kicad_agent_verify`, `kicad_describe`, `kicad_invoke`, `list_toolboxes`, `load_tools`, `load_toolset`, `load_user_config`, `open_project`, `open_schematic_viewer`, `save_project`, `server_stats`, `snapshot_project`, `unload_toolset`.

Catalogue observé : 202 outils, 21 toolsets ; toolset actif lors du probe : `project` (6 outils).

## Capacités pertinentes trouvées

- `run_design_review(schematic|board)`
- `audit_power_rails(schematic)`
- `audit_decoupling(schematic, board?, max_distance_mm=5)`
- `get_recent_calls`
- `server_stats`

## Probes

- `open_project` : appel accepté, mais aucune UI/IPC active.
- `kicad_invoke` → `check_kicad_ui` : succès ; résultat `running=false, ipc_responsive=false`.
- `kicad_invoke` → `run_design_review({})` : échec attendu `invalid_argument` avec `a review needs something to review: pass 'schematic', 'board', or both` ; aucun fichier touché.

## Observabilité au jalon

`server_stats` observait 17 appels et 1 erreur. `get_recent_calls` confirmait les identifiants et statuts. Cette valeur est un instantané initial, pas le total final du benchmark.

## Limitation initiale

Sans projet ouvert et sans instance KiCad avec IPC actif, les capacités réelles d’édition, ERC, PCB et DRC restent non validées. Le prochain probe doit créer puis ouvrir le projet via MCP, lancer/joindre KiCad, et répéter `check_kicad_ui`.

## Reprise live après création du squelette

- Les fichiers `HifiAmp_TPA3255.kicad_pro`, `HifiAmp_TPA3255.kicad_sch` et `HifiAmp_TPA3255.kicad_pcb` sont présents et non suivis par Git au jalon.
- `launch_kicad_ui` : succès.
- `open_project` : succès sur `HifiAmp_TPA3255.kicad_pro`.
- `check_kicad_ui` : succès, `running=true`, `ipc_responsive=true`.

La limitation initiale UI/IPC est donc levée. Le contenu du schéma et du PCB reste à construire et à valider via MCP.
