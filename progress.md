# PROGRESS

## Phase actuelle

Phase D — Empreintes.

## Tâche actuelle

D1 — Attribuer et valider les empreintes.

## Dernière tâche validée

C1 — Gate schématique. **GATE ERC = PASS.** Phases B et C terminées.

Preuves observées :
- Rapport ERC archivé comme livrable : `reports/ERC_B1_gate_2026-08-31.json`, 5 513 octets, produit par `run_erc` MCP. Contenu vérifié hors MCP : **0 occurrence de `"severity": "error"`, 14 de `"severity": "warning"`**.
- Les 4 avertissements « mismatch symbole/librairie » (`U1`, `U2`, `U7`, `U6`) sont **cosmétiques**, établi par comparaison broche par broche entre la définition de librairie et la copie en cache dans `lib_symbols` du `.kicad_sch` : numéros, noms, types électriques et positions strictement identiques pour les quatre. Un diff tokenisé du S-expression complet ne laisse subsister que des différences d'espacement de sérialisation. Aucune resynchronisation effectuée, aucune mutation.
- Les 10 avertissements off-grid sont géométriques : fils ou broches à 0.0254–0.0270 mm hors grille dans le bloc alimentation. Sans effet sur la connectivité — `find_orphan_items` = 0, `find_single_pin_nets` = 0, ERC = 0 erreur. L'extraction de netlist repose sur la coïncidence électrique des points, pas sur l'alignement de grille.
- ERC relancé après analyse : **0 erreur / 14 avertissements**, identique.
- Verdict du gate prononcé par le principal : **PASS**. Aucun problème ERC réel non traité. La Phase D est ouverte.

Réserve reportée du gate : ne jamais déplacer manuellement les points off-grid en Phase D ou E sans revérifier ensuite la coïncidence label/ancre — la connectivité de ce schéma en dépend entièrement.

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

D1.1 — Associer chaque composant à une empreinte compatible et disponible : inventorier les 118 composants, attribuer une empreinte à chacun depuis les librairies KiCad disponibles ou une empreinte locale à créer, en priorité sur les boîtiers imposés — `U6` TPA3255DDV HTSSOP-44 avec PowerPAD, régulateurs `U1`/`U2`/`U3`/`U7`, `U4`/`U5` OPA1612AIDR SOIC-8, inductances de puissance 15 µH, bulk 1500 µF/4700 µF, borniers et RCA. Rapporter la couverture atteinte et la liste des composants restant sans empreinte disponible.
