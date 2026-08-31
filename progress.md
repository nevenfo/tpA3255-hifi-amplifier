# PROGRESS

## Phase actuelle

Phase D — Empreintes.

## Tâche actuelle

B2 — Retouches schématiques issues des décisions mécaniques et de protection. D1.1 est suspendue en PARTIAL et reprendra après le re-gate C2.

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

Aucun. Les 8 empreintes manquantes sont débloquées par les décisions utilisateur du 2026-08-31, qui rouvrent du travail schématique (B2) et imposent un re-gate ERC (C2) avant de clore D1.1.

Décisions utilisateur prises :
- `J2`/`J3` : RCA **déportés sur châssis**, reliés par connecteur de câblage. Signal analogique le plus sensible sur fils : blindage à soigner.
- `RV1` : potentiomètre **déporté par nappe blindée**, hors carte. Point haute impédance.
- Protection 48 V : niveau **renforcé**, avec limitation de l'appel de courant.
- Surtension : **`LM5069` + MOSFET N externe**, coupant réellement l'alimentation. `LM5066` écarté, sa télémétrie PMBus n'apportant rien ici. TVS cantonnée aux transitoires rapides. Seuils à marge explicite sous les limites absolues du TPA3255. Toute grandeur dépendant du MOSFET exact, de la TVS, du shunt ou des seuils précis reste `NEEDS_DATA` plutôt qu'inventée.

**Résultat de conception majeur, consigné dans `docs/protection-48v.md`** : aucune TVS passive au silicium ne satisfait à la fois `V_RWM ≥ 48 V` et `V_C < 65 V`. Le facteur de clamp exigé serait 65/48 = 1,354, contre 1,3 à 1,6 pour la technologie avalanche, familles automobiles load dump comprises. `SMCJ48A` et `SLD8S48A` clampent toutes deux à 77,4 V. Une TVS seule sur ce rail est une protection illusoire contre une surtension soutenue. D'où l'OVP actif.

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
- `docs/architecture.md`, `docs/power-block.md`, `docs/protection-48v.md`
- `reports/ERC_B1_gate_2026-08-31.json`, `reports/MCP_BUG-documenttype-routing-eeschema.md`

## NEXT ACTION

B2.4 — Dimensionner l'étage `LM5069` à partir des équations de sa datasheet (`SNVS452`) : seuils de sous-tension et de surtension avec marge sous les 65 V absolus, résistance de shunt pour ≈ 4,6 A continus, limitation de puissance et temporisateur, puis vérification SOA (B2.5) pour la charge des 15 400 µF. Ensuite seulement, capture schématique du bloc via `kicad-control` (B2.6).
