# PROGRESS

## Phase actuelle

Phase B — Schéma KiCad.

## Tâche actuelle

B1.7 — Revoir références, alimentations, nets critiques et connexions inattendues.

## Dernière tâche validée

B1.6 — Découplages, puissance, filtres LC, sorties et protections ajoutés et corrigés.

Preuves observées (toutes via `kicad-agentic-mcp` v1.1.3, aucune édition directe de fichier) :
- 32 composants ajoutés puis corrigés. Découplages TPA3255 conformes à `SLASEA8A` : `C301` 10 µF + `C302` 0.1 µF (`VDD`), `C303` 1 µF (`VBG`), `C304`/`C305` 0.1 µF (`GVDD_AB`/`GVDD_CD`), `C306`-`C309` 0.033 µF (bootstrap `BST_x`→`OUT_x`), `C310`/`C311` 1 µF/100 V (PVDD local), `C312`-`C315` 1500 µF/63 V et `C316`/`C317` 4700 µF/80 V (bulk, ancre EVM ajustable).
- Valeurs sourcées le 2026-08-31 par lecture directe du PDF `SLASEA8A` rév. A : `C318` 1 µF (`AVDD`), `C319` 1 µF (`DVDD`), `C320` 47 nF (`C_START`) — Figure 29 p. 22 ; `R304` 22 kΩ (`OC_ADJ`, seuil 17.0 A mode CB3C) — Table 4 p. 18. À ne pas confondre avec `R301`, le 22.0 kΩ distinct de `FREQ_ADJ`.
- `OSC_IOM` (9) et `OSC_IOP` (10) laissées non connectées avec drapeau no-connect (`c0b9bae1`, `d6ecc480`), conformément à la table Pin Functions : « Oscillator synchronization interface. Do not connect if not used. »
- Filtre de sortie : `L301`-`L304` 15 µH puis `C321`-`C324` 680 nF film (`d06dedf5`, `ecb4ba24`, `dec1e2de`, `ced78652`) de `OUT_A_F`/`OUT_B_F`/`OUT_C_F`/`OUT_D_F` vers `GND`. Borniers `J301`/`J302` entre `OUT_A_F`/`OUT_B_F` et `OUT_C_F`/`OUT_D_F` ; aucune borne haut-parleur reliée à `GND`.
- Correction d'un écart : le filtre avait d'abord été capturé avec 2 condensateurs différentiels, par interprétation erronée de la règle « sorties BTL jamais reliées à la masse », qui vise les bornes haut-parleur et non le condensateur de filtre. Topologie remise à 4 × 680 nF vers `GND`, seule cohérente avec la coupure calculée 49.8 kHz.
- Correction d'un second écart : la chaîne de protection était un stub isolé. Le label de `J1` broche 1 portait `PVDD` au lieu de `PVDD_EXT`, court-circuitant la protection. Renommé. Chemin réel désormais : `J1` → `PVDD_EXT` (4 points : `J1.1`, `F301.1`, `D301.A1`, jonction) → `F301` → `PVDD_FUSED` (3 points) → `Q301` → `PVDD` (24 points : bulk, `U6` `PVDD_AB`/`PVDD_CD`, `PWR_FLAG`, `Q301.S`). `PWR_FLAG` reste du côté alimenté.
- ERC : 25 → **14 avertissements, 0 erreur**. Les 11 « label connecté à une seule broche » de la baseline ont tous disparu. Les 14 restants sont préexistants (mismatch symbole/librairie locale, off-grid sur le bloc alimentation).
- Persistance vérifiée hors MCP : `HifiAmp_TPA3255.kicad_sch` 241 387 octets, mtime 2026-08-31 11:53:56 +0200, révision MCP `499caba8320b6ab6-241387` ; `C321`-`C324`, `PVDD_EXT`, `PVDD_FUSED` et 4 drapeaux no-connect présents.
- Aucun gate ERC revendiqué : B1.7 reste à faire avant la Phase C.

Réserves consignées en B1.6 :
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
- Neuf `NEEDS_DATA` centralisés dans `docs/architecture.md`, dont le découplage `VMID` (`C110`/`C210`, Value littéralement `NEEDS_DATA`).
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

B1.7 — Revoir références, alimentations, nets critiques et connexions inattendues : passer en revue l'unicité et la cohérence des références (plages 1xx gauche, 2xx droite, 3xx puissance), la continuité de chaque rail (`PVDD`, `+15V`, `+12V`, `+12V-OA`, `+3V3`, `VMID`, `GND`) depuis sa source jusqu'à ses charges, la présence d'un `PWR_FLAG` par rail alimenté, et l'absence de net à un seul point ou de connexion inattendue. Produire la liste des nets critiques avec leur nombre de points connectés, puis conclure explicitement si B1 est prêt pour le gate ERC de la Phase C.
