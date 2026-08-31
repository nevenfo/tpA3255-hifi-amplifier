# PROGRESS

## Phase actuelle

Phase B — Schéma KiCad.

## Tâche actuelle

B1.6 — Ajouter découplages, puissance, filtres LC, sorties et protections.

## Dernière tâche validée

B1.5 — TPA3255, interface différentielle et contrôles ajoutés et nets inspectés.

Preuves observées (toutes via `kicad-agentic-mcp` v1.1.3, aucune édition directe de fichier) :
- Symbole `TPA3255DDV` créé dans `HifiAmp_TPA3255_Local`, mono-unité 44 broches + PowerPAD (45). Brochage extrait de la datasheet TI `SLASEA8A` rév. A, section 6 « Pin Configuration and Functions », boîtier `DDV` : le diagramme top-view p. 3 fait foi pour numéros et noms, la table p. 4 (colonnes désynchronisées à l'extraction) a été croisée avec Table 1 Mode Selection, Recommended Operating Conditions et Absolute Maximum Ratings pour les types électriques.
- `U6` placé (UUID `1a25a659`) plus `R301` 22.0 kΩ (`FREQ_ADJ`), `R302`/`R303` 10.0 kΩ (pull-ups `FAULT`/`CLIP_OTW` vers `+3V3`).
- Câblage vérifié : `INPUT_A`/`INPUT_B`/`INPUT_C`/`INPUT_D` ; `M1`/`M2` à `GND` (BTL stéréo, connexion directe conforme Table 1) ; `RESET` relié à `U3` TPS3802K33 ; `OUT_A`/`OUT_B`/`OUT_C`/`OUT_D` en labels nommés en attente des filtres LC.
- Alimentations confirmées par inspection : net `PVDD` = 48 V externe, 15 labels dont 6 sur les broches `PVDD_AB` (36/37/38) et `PVDD_CD` (29/30/31) de `U6`, les 9 autres sur le bloc alimentation (`J1` PVDD 48 V IN, `D3`, `R6`, `C39`, `C3`, `C11`, `C4`, `C2`, `PWR_FLAG`). `VDD` (2), `GVDD_AB` (1), `GVDD_CD` (22) sur `+12V`. Broches de masse (12/13/25/26/33/34/41/42) et PowerPAD (45) sur `GND`.
- ERC : **0 erreur / 25 avertissements** (14 avant B1.5). Delta expliqué : −1 (`Label connected to only one pin: Label 'RESET'` résolu), +11 labels à une seule broche (un par net reporté à B1.6), +1 mismatch symbole/librairie de la même catégorie que `U1`/`U2`/`U7`. Aucune erreur nouvelle.
- Persistance : `HifiAmp_TPA3255.kicad_sch` 192 536 octets, `HifiAmp_TPA3255.kicad_sym` 15 380 octets, mtime 2026-08-31 11:22:13 +0200, révisions MCP `83a63fbeb7a17d30-192536` et `6da009be53da495c-15380`.
- Aucun gate ERC revendiqué : B1 n'est pas terminé.

Écarts consignés en B1.5 :
- Réutiliser un nom de symbole supprimé laissait un cache de broches périmé dans `lib_symbols` du `.kicad_sch`, produisant 2 fausses erreurs ERC (broches PVDD de sortie mal typées). Contourné en renommant le symbole `TPA3255B` (champ Value affiché `TPA3255DDV`).
- Les 4 broches de sortie physiquement dupliquées (39/40 et 27/28) sont typées `passive` au lieu de `power_out`, pour éviter un conflit de pilotes ERC entre broches d'un même demi-pont.

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
- Huit `NEEDS_DATA` centralisés dans `docs/architecture.md`, dont le découplage `VMID` (`C110`/`C210`, Value littéralement `NEEDS_DATA`).
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

B1.6 — Ajouter découplages, puissance, filtres LC, sorties et protections : peupler les 11 nets reportés par B1.5 (`OC_ADJ`, `DVDD`, `AVDD`, `VBG`, `C_START`, `BST_A/B/C/D`) et ajouter, selon `docs/architecture.md` § « Découplage et réservoir » et § « Filtre de sortie », `VDD` 10 µF + 0.1 µF, `VBG` 1 µF, `GVDD_AB`/`GVDD_CD` 0.1 µF chacun, `BST_x`–`OUT_x` 0.033 µF chacun, PVDD 1 µF/100 V par groupe de broches plus bulk, les quatre inductances 15 µH et quatre condensateurs film 680 nF du filtre de sortie, et les deux borniers haut-parleur 2 pôles sans référence commune. Inspecter ensuite les nets et relancer un ERC ; les 11 avertissements « label connecté à une seule broche » doivent disparaître.
