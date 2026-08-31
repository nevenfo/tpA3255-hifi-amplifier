# PROGRESS

## Phase actuelle

Phase B — Schéma KiCad.

## Tâche actuelle

B1.5 — Ajouter TPA3255, interface différentielle et contrôles puis inspecter les nets.

## Dernière tâche validée

B1.4 — Entrée/volume/analogique droite ajoutée et nets inspectés.

Preuves observées (toutes via `kicad-agentic-mcp` v1.1.3, aucune édition directe de fichier) :
- Bloc droit symétrique du gauche : `J3` (S=`GND`, T=`RCA_R`) → `C201` 4.7 µF → `RV1` gang 2 (broches 4/5/6 = `VOL_R_IN`/`VOL_R`/`GND`) → `U5` unité 1 inverseur −1 (`R201`/`R202`, `C202`, `VMID` sur l'entrée +, sortie `-VSE_R`) → `U5` unité 2 inverseur −1 (`R203`/`R204`, `C203`, sortie `+VSE_R`) → 10 µF → `R207`/`R208` 100 R → 100 pF → `INPUT_C`/`INPUT_D`. Découplages locaux `U5` 0.1 µF + 10 µF sur `+12V-OA`/`GND`, plus `C210` `VMID`/`GND`.
- 20 éléments ajoutés : `J3`, `C201`, `U5` unités 1-3, `R201`-`R204`, `R207`, `R208`, `C202`-`C210`. Total 82 composants, 96 références.
- ERC : **0 erreur / 14 avertissements**. Les 3 `Pin not connected: Symbol RV1 Pin 4/5/6` ont disparu ; aucune erreur nouvelle. Les 14 avertissements sont ceux, préexistants, du bloc alimentation (mismatch librairie `U1`/`U2`/`U7`, off-grid, `Label connected to only one pin: Label 'RESET'`).
- Connectivité recontrôlée en direct sur `J3`, `U5` unité 1 et `RV1` gang 2 : aucune broche flottante, aucune connexion inattendue.
- Persistance vérifiée hors MCP : `HifiAmp_TPA3255.kicad_sch`, 141 685 octets (118 112 avant B1.4), mtime 2026-08-31 11:05:44 +0200 ; `J3`, `C201`, `U5`, `R201`, `R204`, `R208`, `C210`, `INPUT_C`, `INPUT_D`, `VOL_R_IN`, `+VSE_R` présents.
- `C210` reprend le `NEEDS_DATA` de `C110` (découplage `VMID`) ; aucun nouveau `NEEDS_DATA`.
- Aucun gate ERC revendiqué : B1 n'est pas terminé.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3, seule version active, validée sur Eeschema et PCB réels. L'ancien blocage `DocumentType` est résolu et obsolète : ne pas restaurer `v1.1.2` ni l'ancienne build patchée. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

Limitations observées, consignées sans contournement :
- `save_project` échoue hors GUI : `Cannot connect to KiCAD IPC at ipc://C:\Users\FlowUP\AppData\Local\Temp\kicad\api.sock: Connection refused`. Les écritures schématiques sont fichier et persistées ; la persistance se prouve par relecture.
- Symboles multi-unités : `get_pin_connections{reference:"U4", pin_number:"4"}` → `Pin '4' not found on 'U4'`, `get_component_nets` ne renvoie que l'unité 1, adressage par UUID refusé. Unités 2/3 vérifiées indirectement par ERC et coïncidence label/ancre.
- `find_orphan_items` classe en `floating_label` des labels posés sur ancre de broche ; le verdict vient de `run_erc`.
- kicad-cli désigne un item différent parmi des points coïncidents d'un run ERC à l'autre : compte et nature des avertissements constants.

## Décisions actives

- Toute édition schéma/PCB reste réservée à `kicad-control`/MCP ; aucun `.kicad_sch`/`.kicad_pcb` modifié directement.
- Aucun PCB avant gate ERC PASS.
- Huit `NEEDS_DATA` centralisés dans `docs/architecture.md`, dont le découplage `VMID` (`C110`/`C210`, champ Value littéralement `NEEDS_DATA`).
- Une opération de placement MCP est appelée une seule fois puis vérifiée par UUID ; inspecter l'état réel avant toute mutation.
- Connectivité sans fils : labels dont l'ancre coïncide avec l'extrémité de broche. Ne jamais poser de label hors ancre.
- Asymétrie de nommage assumée : canal gauche `-VSE`/`+VSE`, canal droit `-VSE_R`/`+VSE_R`. Renommage du gauche non effectué (risque de régression) ; à trancher avant les livrables H2.
- Fichiers projet KiCad encore non versionnés dans Git (untracked) ; seuls `plan.md`, `progress.md` et `docs/` sont checkpointés.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym`, `sym-lib-table`
- `docs/architecture.md`, `docs/power-block.md`
- `reports/MCP_BUG-documenttype-routing-eeschema.md`

## NEXT ACTION

B1.5 — Ajouter TPA3255, interface différentielle et contrôles puis inspecter les nets : placer le `TPA3255DDV` (HTSSOP-44), relier `INPUT_A`/`INPUT_B`/`INPUT_C`/`INPUT_D` déjà disponibles, câbler `PVDD`, `GND`/`PGND`, `VDD`, `GVDD_AB`/`GVDD_CD` sur 12 V, `M1=0`/`M2=0` pour le BTL stéréo, `FREQ_ADJ = 22.0 kΩ`, `RESET`/`MUTE`/`FAULT`/`OTW` vers la supervision 3.3 V, et sortir `OUT_A`..`OUT_D` sur des nets nommés. Inspecter ensuite les nets et relancer un ERC intermédiaire ; découplages, bootstrap, bulk et filtres LC restent pour B1.6.
