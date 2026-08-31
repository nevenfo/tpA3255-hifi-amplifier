# PROGRESS

## Phase actuelle

Phase B — Schéma KiCad.

## Tâche actuelle

C1 — Gate schématique : ERC, classement, archivage et verdict PASS/FAIL.

## Dernière tâche validée

B1.7 — Revue des références, alimentations, nets critiques et connexions inattendues. **B1 est terminée.**

Preuves observées (revue d'inspection pure, aucune mutation, 0 appel d'écriture) :
- Références : aucune anomalie. 117 lignes BOM, plages cohérentes (1xx gauche, 2xx droite, 3xx puissance, historique alimentation). Aucun doublon, aucun `R?`/`C?`. Les 5 `PWR_FLAG` portent une référence `?`, ce qui est normal pour un symbole virtuel.
- Rails tracés source → charges : `PVDD_EXT` 3 points (`J1.1`, `F301`, `D301`) → `PVDD_FUSED` 3 points → `PVDD` 17 points avec `PWR_FLAG` ; `BUCK_VIN` 4 points ; `+15V` 7 points depuis `L1` ; `+12V` 10 points depuis `U2` ; `+12V-OA` 9 points depuis `L6` ; `+3V3` 4 points depuis `U3` ; `VMID` 8 points ; `GND` 63 broches. L'absence de `PWR_FLAG` sur `+12V`, `+3V3` et `VMID` est cohérente : ces nets sont pilotés par une sortie active, pas seulement par des broches Power-Input, et l'ERC ne les signale pas.
- `find_single_pin_nets` : **0 net orphelin**.
- Chaîne audio tracée de bout en bout sur les deux canaux, symétrie électrique confirmée jusqu'aux borniers. Aucune asymétrie réelle au-delà de l'asymétrie de nommage déjà assumée.
- Commande confirmée liaison par liaison : `RESET` `U7.3` → `U6.18` ; `FAULT` `U6.19` et `CLIP_OTW` `U6.21` avec `R302`/`R303` vers `+3V3` ; `M1`/`M2` à `GND` ; `FREQ_ADJ` `R301` 22.0 kΩ ; `OC_ADJ` `R304` 22 kΩ.
- ERC : **0 erreur / 14 avertissements**, tous classés dans les deux catégories attendues — 4 mismatch symbole/librairie locale (`U1`, `U2`, `U7`, `U6`) et 10 off-grid concentrés dans le cluster du bloc alimentation. Aucun avertissement hors catégorie.
- `find_shorted_nets` remonte 30 « shorts » : tous opposent un rail à un **nom de symbole**, artefact du même mismatch symbole/librairie ; l'outil se déclare lui-même consultatif et l'ERC donne 0 erreur. Aucun court réel.
- Correction documentaire : le mapping `U3`/`U7` était inversé dans ce fichier. Le schéma et `docs/power-block.md` font foi — `U7` = TPS3802K33 (superviseur RESET), `U3` = TLV1117-33 (3.3 V). Corrigé ici.
- Anomalie signalée, non corrigée : la chaîne EVM `PVDD` → `R6` 100 kΩ → `C83` 1 µF → `RESET-SW` → `U7.MR` couple la broche MR au rail 48 V. Le continu est bloqué par `C83`, mais la contrainte transitoire au démarrage n'est pas bornée sans la tension absolue maximale de MR ni le dV/dt de `PVDD`. Consigné comme dixième `NEEDS_DATA` dans `docs/architecture.md`.

Réserves héritées de B1.6 :
- `F301` (fusible), `D301` (TVS) et `Q301` (MOSFET anti-inversion) gardent `NEEDS_DATA` en Value, couverts par le `NEEDS_DATA` « protection 48 V inversion/surtension et TVS ».
- Le montage grille-drain auto-polarisé de `Q301` est insuffisant pour un Vgs à 48 V. Signalé, non corrigé faute de source : à trancher avant le gel.
- Le réseau d'amortissement `10 nF + 3.3 Ω` et le `1 nF` visibles en Figure 29 ne sont pas capturés ; ils restent couverts par le `NEEDS_DATA` EMI/stabilité.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3, seule version active. L'ancien blocage `DocumentType` est résolu et obsolète : ne pas restaurer `v1.1.2` ni l'ancienne build patchée. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

Limitations observées, consignées sans contournement :
- `save_project` échoue hors GUI : `Cannot connect to KiCAD IPC at ipc://C:\Users\FlowUP\AppData\Local\Temp\kicad\api.sock: Connection refused`. Les écritures schématiques sont fichier et persistées ; la persistance se prouve par relecture.
- Symboles multi-unités : `get_pin_connections{reference:"U4", pin_number:"4"}` → `Pin '4' not found`, `get_component_nets` ne renvoie que l'unité 1, adressage par UUID refusé.
- `get_net_components`/`get_pin_connections` manquent les broches de `U6` ; la connectivité de ce bloc se prouve par `list_schematic_labels` (coïncidence label/ancre) plus `run_erc`.
- `find_orphan_items` classe en `floating_label` des labels posés sur ancre de broche ; le verdict vient de `run_erc`.
- kicad-cli désigne un item différent parmi des points coïncidents d'un run ERC à l'autre : compte et nature des avertissements constants.

## Décisions actives

- Toute édition schéma/PCB/librairie reste réservée à `kicad-control`/MCP ; aucun fichier KiCad modifié directement. Aucune redélégation par le worker.
- Aucun PCB avant gate ERC PASS.
- Dix `NEEDS_DATA` centralisés dans `docs/architecture.md`, dont le découplage `VMID` (`C110`/`C210`, Value littéralement `NEEDS_DATA`).
- Une opération de placement MCP est appelée une seule fois puis vérifiée par UUID ; inspecter l'état réel avant toute mutation.
- Connectivité sans fils : labels dont l'ancre coïncide avec l'extrémité de broche. Ne jamais poser de label hors ancre.
- Asymétrie de nommage assumée : canal gauche `-VSE`/`+VSE`, canal droit `-VSE_R`/`+VSE_R`. À trancher avant les livrables H2.
- Fichiers projet KiCad encore non versionnés dans Git (untracked) ; seuls `plan.md`, `progress.md` et `docs/` sont checkpointés.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym`, `sym-lib-table`
- `docs/architecture.md`, `docs/power-block.md`
- `reports/MCP_BUG-documenttype-routing-eeschema.md`

## NEXT ACTION

C1 — Gate schématique : exécuter `run_erc` une dernière fois, archiver le rapport dans `reports/`, confirmer le classement des 14 avertissements établi en B1.7, puis consigner explicitement PASS ou FAIL du gate. La Phase D reste interdite tant que le gate n'est pas PASS.
