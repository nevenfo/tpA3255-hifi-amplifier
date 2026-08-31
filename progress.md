# PROGRESS

## Phase actuelle

Phase B2 terminée pour sa partie topologique. **GATE C2 = PASS.**

## Tâche actuelle

Décision utilisateur en attente sur le dimensionnement du bulk, qui conditionne B2.5 (vérification SOA) puis la reprise de D1.1.

## Dernière tâche validée

**C2 — Re-gate ERC. GATE C2 = PASS**, prononcé par le principal après vérification indépendante du rapport archivé.

- ERC final : **0 erreur / 15 avertissements** — 5 « mismatch symbole/librairie » (`U1`, `U2`, `U6`, `U7`, `U8`) et 10 off-grid, aucune autre catégorie. Rapport : `reports/ERC_C2_gate_final-2026-08-31.json`.
- Le 5ᵉ mismatch est `U8`, écart attendu pour un symbole créé localement, de même nature bénigne que les quatre autres.
- Réserve du gate C1 maintenue : ne jamais déplacer les points off-grid sans revérifier ensuite la coïncidence label/ancre.

### Chaîne de protection capturée et vérifiée

`J1` → `PVDD_EXT` → `F301` → `PVDD_FUSED` → `D301` (TVS) et `Q301` (anti-inversion P-MOS) → `PVDD_PROT` → `R306` 4 mΩ → `PVDD_SENSE` → `Q302` → `PVDD` → bulk et TPA3255.

- `U8` (`LM5069-2`) : `VIN`=`PVDD_PROT`, `SENSE`=`PVDD_SENSE`, `GATE`=`HS_GATE` vers la grille de `Q302`, `OUT`=`PVDD`, `PGD` en no-connect explicite.
- Diviseur de seuils en configuration *Option A* : `R307` 191 k entre `PVDD_PROT` et `UVLO`, `R308` 5,11 k entre `UVLO` et `OVLO`, `R309` 9,09 k entre `OVLO` et `GND`. `R310` 147 k de `PWR` à `GND`, `C325` 3,9 µF de `TIMER` à `GND`, `C326` 100 nF de découplage sur `VIN`.
- `PWR_FLAG` ajouté sur `PVDD_PROT` (6 au total) : résout l'erreur `Input Power pin not driven` sur `U8` pin 2.
- Les 22 consommateurs préexistants de `PVDD` sont inchangés.

### Défauts trouvés et corrigés par le principal, tous invisibles à l'ERC

1. **Orientation du P-MOS `Q301`** : câblé source en amont, donc protection anti-inversion inopérante — la diode de structure d'un P-MOS a son anode sur le drain, qui doit être en amont. Corrigé et vérifié par lecture de la géométrie du symbole.
2. **Coordination fusible/TVS** : `D301` était en amont de `F301`. Une TVS dont le mode de défaillance est le court-circuit doit être en aval du fusible, sinon elle met la source en court sans qu'aucun fusible ne coupe. Rattachée à `PVDD_FUSED`.
3. **Empreinte de `U6`** (phase D1.1) : HTSSOP-44 sans pad thermique alors que le symbole déclare un pin 45 `EP`. Remplacée par la variante `-1EP`.

## Blocage actif

**Décision utilisateur requise : dimensionnement du bulk.**

La datasheet `SNVS452G` §9.2.1.1 l'énonce directement — *« the FET's total energy dissipation equals the total energy stored in the output capacitor (½CV²) »*. L'exigence SOA est donc structurelle et ne dépend pas du réglage.

| Bulk | Exposition SOA pire cas |
|---|---|
| 15 400 µF (actuel) | 389 W pendant **318 ms** |
| 8 200 µF | 377 W pendant **179 ms** |
| 4 700 µF | 390 W pendant **98 ms** |

Un MOSFET de commutation ordinaire ne convient pas : il faut une SOA garantie en mode linéaire. Réduire le bulk raccourcit fortement la durée d'exposition, ce qui est le levier décisif — à arbitrer contre le ripple. **Non tranché.**

Tant que ce point n'est pas réglé, B2.5 reste bloquée et `Q302` ne peut pas être choisi.

Réduire le bulk modifierait la netlist et **imposerait un nouveau gate ERC**. Figer des références sans changer la topologie (B2.5, B2.7) ne l'impose pas.

## NEEDS_DATA ouverts

Maintenus sur consigne explicite de l'utilisateur plutôt qu'inventés : `Q302` et sa courbe SOA, `R_DS(on)` et résistance thermique du MOSFET retenu, P-MOS `Q301`, Zener `D302`, TVS `D301`, fusible `F301`, `C110`/`C210`, potentiomètre `RV1`, et la confirmation par dessin mécanique TI que l'EP du TPA3255DDV vaut 5,2 × 14 mm.

Levés cette session : brochage VSSOP-10 du `LM5069` (section 6 de la datasheet), et le choix de la variante `-2`.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2` : blocage `DocumentType` résolu, analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Les écritures sont fichier et persistées ; prouver par relecture.
- **Attributs `on_board` / `in_bom` / `dnp` inaccessibles**, et aucune suppression de propriété isolée : `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées.
- `add_power_symbol` : le paramètre `power_net` désigne le **nom du symbole de librairie**, pas le net cible. Passer `"PWR_FLAG"` ; le rattachement au net se fait uniquement par coïncidence de position.
- `get_schematic_component` / `get_component_nets` exigent un **chemin absolu**.
- Ces deux outils mésattribuent les broches `power_in` : elles remontent `"net":"LM5069-2"` de type `PowerSymbol` alors que `list_schematic_labels` et `get_net_connections` confirment les bons `NetLabel`. Le fichier est correct ; c'est la résolution de net de l'outil qui est fautive.
- Outils chargés par `load_toolset` accessibles seulement via `kicad_invoke` ; appel direct → `Error: No such tool available`.
- `search_footprints` n'indexe pas toute la librairie globale : vérifier sur disque avant de conclure à une absence.
- Sortie MCP tronquée au-delà d'environ 72 000 caractères.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.** Trois redressements à ce jour : un comptage ERC faux, une affirmation erronée sur le sens de la diode de structure, et un rapport prétendant à tort n'avoir rien fait.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- TVS cantonnée aux transitoires rapides ; la protection en surtension est **active**, par `LM5069`. Aucune TVS passive ne peut borner ce rail sous 65 V (facteur de clamp requis 1,354 contre 1,3 à 1,6 pour la technologie avalanche).
- `LM5066` écarté : télémétrie PMBus inutile ici.
- Vias thermiques du PowerPAD `U6` traités en Phase E par calcul.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.
- Fichiers projet KiCad versionnés ; artefacts volatils exclus par `.gitignore`.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym` (contient le symbole `LM5069` créé localement), `sym-lib-table`, `fp-lib-table`, `HifiAmp_TPA3255_Local.pretty/`
- `docs/architecture.md`, `docs/power-block.md`, **`docs/protection-48v.md`**
- `reports/ERC_C2_gate_final-2026-08-31.json`

## NEXT ACTION

Obtenir de l'utilisateur l'arbitrage sur le bulk (15 400 / 8 200 / 4 700 µF), puis sourcer un MOSFET `Q302` à SOA garantie en mode linéaire satisfaisant l'exposition retenue, clore B2.5, et reprendre D1.1 sur les 8 empreintes manquantes.
