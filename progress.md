# PROGRESS

## Phase actuelle

Phase D — Empreintes.

## Tâche actuelle

D1.1 — Attribution des empreintes. **PARTIAL, non cochée.**

## Dernière tâche validée

C1 — Gate schématique. **GATE ERC = PASS** (0 erreur / 14 avertissements cosmétiques ou off-grid bénins). Phases A, B, C terminées.

## État de D1.1

**103 empreintes assignées / 111 composants en nécessitant une** (120 instances de symboles = 116 désignateurs distincts après regroupement des unités de `U4`/`U5`, moins 5 `PWR_FLAG` sans empreinte par conception).

Vérifié par le principal hors MCP, par relecture directe du `.kicad_sch` et du disque :
- 21 empreintes distinctes utilisées, **toutes résolvables** : fichier `.kicad_mod` présent pour chacune.
- Les composants sans empreinte sont exactement les 8 listés ci-dessous ; les 5 refs `?` restantes sont bien les `PWR_FLAG`.
- Librairies : table globale KiCad 10.0 imbriquée (`type "Table"` → template, 155 libs) + `HifiAmp_TPA3255_Local.pretty` (une seule entrée préexistante, `CF_Film_Box_P5.00mm_7.2x3.5mm`, utilisée par `C321`–`C324`).

**Correction critique acquise sur `U6` (TPA3255DDV)** : l'empreinte précédemment assignée `Package_SO:HTSSOP-44_6.1x14mm_P0.635mm_TopEP4.14x7.01mm` était défectueuse — 44 pads, **aucun pad thermique**, alors que le symbole `HifiAmp_TPA3255_Local:TPA3255B` déclare un pin 45 `EP` de type `power_in`. Remplacée par `Package_SO:HTSSOP-44-1EP_6.1x14mm_P0.635mm_EP5.2x14mm_Mask4.31x8.26mm`, dont le pad `"45"` porte `size 5.2 14` et `property pad_prop_heatsink`. Persistance et immobilité géométrique prouvées : `x=340.36, y=180.34` et UUID inchangés, ERC post-mutation identique (0 erreur / 14 avertissements).

Variante `_ThermalVias` disponible dans la même librairie mais **délibérément non retenue** : le motif de vias thermiques doit être dimensionné par le calcul thermique réel en Phase E, pas hérité d'un défaut générique.

## Blocage actif

8 composants sans empreinte, tous par indétermination du composant physique, aucun par échec d'outil :

| Réf | Boîtier requis | Cause |
|---|---|---|
| `J2`, `J3` | Connecteur RCA | **Aucune empreinte RCA/cinch/phono dans les 155 librairies KiCad 10.0** (vérifié par le principal). Création locale impossible sans cotes mécaniques d'un P/N choisi. |
| `RV1` | Potentiomètre double 10 k LOG | `NEEDS_DATA mecanique` — axe, pas de pattes, montage PCB ou déporté non tranchés |
| `C110`, `C210` | Céramique SMD | `NEEDS_DATA` découplage `VMID` — capacité/tension non tranchées, donc boîtier indéterminé |
| `F301` | Fusible | `NEEDS_DATA` — SMD ou porte-fusible traversant non tranché |
| `D301` | TVS | `NEEDS_DATA` — boîtier dépend du calibre |
| `Q301` | MOSFET anti-inversion | `NEEDS_DATA` — SOT-23 / DPAK / TO-220 selon calibre |

Prochaine tentative : trancher les choix mécaniques avec l'utilisateur, puis sourcer les P/N fabricant.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3, seule version active. Ancien blocage `DocumentType` résolu et obsolète : ne pas restaurer `v1.1.2`. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

Limitations observées, consignées sans contournement :
- `save_project` échoue hors GUI : `Cannot connect to KiCAD IPC at ipc://C:\Users\FlowUP\AppData\Local\Temp\kicad\api.sock: Connection refused`. Les écritures sont fichier et persistées ; la persistance se prouve par relecture.
- `list_library_footprints` refuse le nickname `fp-lib-table` : `{"error":{"kind":"file_not_found","path":"HifiAmp_TPA3255_Local"},"message":"Not a directory: HifiAmp_TPA3255_Local"}` — exige le chemin `.pretty` complet.
- `search_footprints("RCA")` → `{"count":0,"results":[]}` : l'index MCP ne couvre pas la librairie globale sur disque. Ne pas conclure d'une absence d'index à une absence réelle sans vérification disque.
- Symboles multi-unités : `get_pin_connections{reference:"U4", pin_number:"4"}` → `Pin '4' not found` ; `get_component_nets` ne renvoie que l'unité 1. N'a pas gêné l'assignation d'empreinte, qui est restée cohérente sur les 3 instances d'unité de `U4` et `U5`.
- Inspection large en un appel : sortie de 71 989 caractères dépassant la limite d'affichage MCP ; découper.

## Décisions actives

- Toute édition schéma/PCB/librairie reste réservée à `kicad-control`/MCP. Lecture de fichier hors MCP autorisée pour vérifier, jamais pour muter.
- Aucune mutation géométrique du schéma : la connectivité repose entièrement sur la coïncidence label/ancre de broche. Ne jamais déplacer les points off-grid sans revérifier ensuite cette coïncidence.
- Une opération de mutation MCP est appelée une seule fois puis vérifiée par relecture.
- Vias thermiques du PowerPAD `U6` : traités en Phase E par calcul, pas par empreinte générique.
- Asymétrie de nommage assumée : canal gauche `-VSE`/`+VSE`, canal droit `-VSE_R`/`+VSE_R`. À trancher avant les livrables H2.
- Le montage grille-drain auto-polarisé de `Q301` est insuffisant pour un Vgs à 48 V. Signalé, non corrigé faute de source : à trancher avant le gel.
- Réseau d'amortissement `10 nF + 3.3 Ω` et `1 nF` de la Figure 29 non capturés ; couverts par le `NEEDS_DATA` EMI/stabilité.
- `NEEDS_DATA` centralisés dans `docs/architecture.md`. Trois nouveaux issus de D1.1 : P/N du RCA `J2`/`J3`, P/N du potentiomètre `RV1`, confirmation par dessin mécanique TI que l'EP du DDV vaut bien 5,2 × 14 mm.
- Fichiers projet KiCad désormais versionnés dans Git ; artefacts volatils exclus par `.gitignore`.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym`, `sym-lib-table`, `fp-lib-table`, `HifiAmp_TPA3255_Local.pretty/`
- `docs/architecture.md`, `docs/power-block.md`
- `reports/ERC_B1_gate_2026-08-31.json`, `reports/MCP_BUG-documenttype-routing-eeschema.md`

## NEXT ACTION

Trancher avec l'utilisateur les 4 choix mécaniques bloquants — montage des RCA `J2`/`J3`, montage du potentiomètre `RV1`, format du fusible `F301`, calibre et boîtier de la protection `D301`/`Q301` — puis sourcer les P/N fabricant et clore D1.1 avant d'ouvrir D1.2.
