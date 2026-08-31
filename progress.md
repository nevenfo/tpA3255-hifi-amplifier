# PROGRESS

## Phase actuelle

Phase B2 — Retouches schématiques post-gate.

## Tâche actuelle

B2.6 — Capturer le bloc `LM5069` au schéma. D1.1 reste suspendue en PARTIAL (103/111) jusqu'au re-gate C2 final.

## Dernière tâche validée

**B2.2, B2.3 et B2.4 terminées et vérifiées par le principal hors MCP.**

- `J2`/`J3` convertis en `Connector_Generic:Conn_01x02` (RCA déportés sur châssis). Nets strictement préservés : `RCA_L`+`GND`, `RCA_R`+`GND`.
- `J4` (`Conn_01x06`, VOLUME_REMOTE) créé, reprenant les 6 nets de `RV1` : `VOL_L_IN`/`VOL_L`/`GND`, `VOL_R_IN`/`VOL_R`/`GND`, masse en position externe de chaque triplet.
- `Q301` requalifié de N-MOS en **P-MOS** (`Transistor_FET:Q_PMOS_GSD`), grille séparée du drain sur `Q301_GATE`, avec `R305` (100 kΩ vers `GND`) et `D302` (Zener 15 V).
- ERC après retouches : **0 erreur / 14 avertissements**, répartis en 4 « mismatch symbole/librairie » (`U1`, `U2`, `U6`, `U7`) et 10 off-grid — **répartition identique à la référence du gate C1**, aucun écart. Rapports : `reports/ERC_B2_regate_2026-08-31.json` et `reports/ERC_B2_regate_final-2026-08-31.json`.

**Défaut trouvé et corrigé par le principal, invisible à l'ERC** : le P-MOS avait d'abord été câblé source en amont. Or la diode de structure d'un P-MOS a son anode sur le **drain**, qui doit donc être en amont pour conduire dans le sens du courant normal. Avec l'orientation initiale, une inversion de polarité rendait la diode passante et la protection inopérante. Câblage vérifié après correction par lecture de la géométrie du symbole : `S` (pin 2, à y −5,08 dans le repère symbole) porte `PVDD`, `D` (pin 3) porte `PVDD_FUSED`, `D302` cathode sur `PVDD`, anode sur `Q301_GATE`.

**B2.4 — étage `LM5069` dimensionné** sur la datasheet officielle `SNVS452G` (rév. janvier 2020), téléchargée et extraite localement, les sources secondaires étant insuffisantes. Équations validées contre l'exemple chiffré de TI avant application (équation 9 redonne 14,90 kΩ ; recoupement `PWRLIM-1` à 303 W contre 300 W). Détail complet dans `docs/protection-48v.md`.

Valeurs figées : `U8` = **`LM5069-2`** (redémarrage automatique ; la variante `-1` se verrouille définitivement), `R1`/`R2`/`R3` = 191 k / 5,11 k / 9,09 kΩ, `R_SNS` = 4 mΩ, `R_PWR` = 147 kΩ, `C_TIMER` = 3,9 µF.

Seuils obtenus : UVLO 40,1 / 36,1 V ; **OVLO coupe à 56,4 V**, reprend à 52,3 V. Pire cas cumulé (seuil interne 2,6 V max + résistances 1 %) : 59,9 V, soit **5,1 V sous le maximum absolu de 65 V** du TPA3255. Limitation de courant entre 12,1 et 15,4 A, donc conduction garantie au-delà des ≈ 9,3 A du pire cas 4 Ω. `t_flt,min` = 122 ms > `t_start,max` = 94 ms, marge × 1,30 vérifiée pire cas contre pire cas.

## Blocage actif

Aucun blocage empêchant d'avancer, mais trois réserves ouvertes :

1. **Exigence SOA structurelle (B2.5)** : `P_LIM × t_flt` suit l'énergie de charge du bulk quel que soit le réglage. Les 15 400 µF actuels imposent au MOSFET de tenir **6,95 A sous 56 V pendant 318 ms, soit 389 W en mode linéaire**. Un MOSFET de commutation ordinaire ne convient pas : il faut une SOA garantie en mode linéaire. Réduire le bulk raccourcit fortement l'exposition (179 ms à 8 200 µF, 98 ms à 4 700 µF) — arbitrage contre le ripple, **non tranché avec l'utilisateur**.
2. **`RV1` non exclu du PCB (B2.8)** : aucun des 202 outils MCP n'expose l'attribut `on_board` ni la suppression d'une propriété isolée. `RV1` reste `(on_board yes)` et porte une propriété parasite vide `exclude_from_board`. Action manuelle KiCad requise.
3. `NEEDS_DATA` maintenus sur demande explicite de l'utilisateur plutôt qu'inventés : MOSFET `Q302` et sa SOA, `R_DS(on)` et résistance thermique, brochage VSSOP-10 du `LM5069`, P-MOS `Q301`, Zener `D302`, `F301`, `D301`, `C110`/`C210`.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2` : blocage `DocumentType` résolu, analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

Limitations observées, consignées sans contournement :
- `save_project` / `open_project` échouent hors GUI : `Cannot connect to KiCAD IPC at ipc://...api.sock: Connection refused`. Les écritures schématiques sont fichier et persistées ; la persistance se prouve par relecture MCP.
- **Attributs de symbole `on_board` / `in_bom` / `dnp` inaccessibles**, et aucune suppression de propriété isolée. `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées ; passer `exclude_from_board` en champ crée une propriété texte inerte.
- `get_schematic_component` / `get_component_nets` exigent un **chemin absolu** : un chemin relatif donne `IO error: canonicalizing : Le chemin d'accès spécifié est introuvable. (os error 3)`.
- Outils chargés par `load_toolset` accessibles uniquement via la passerelle `kicad_invoke` ; appel direct → `Error: No such tool available`.
- `list_library_footprints` refuse le nickname : exige le chemin `.pretty` complet.
- `search_footprints` n'indexe pas toute la librairie globale : ne jamais conclure d'une absence d'index à une absence réelle sans vérification disque.
- Symboles multi-unités : `get_pin_connections` sur `U4` pin 4 → `Pin '4' not found`.
- Sortie MCP tronquée au-delà d'environ 72 000 caractères : découper les inspections larges.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture de fichier hors MCP autorisée pour vérifier, jamais pour muter.
- **Les rapports d'agents sont vérifiés indépendamment par le principal avant tout verdict.** Deux erreurs de rapport ont déjà été redressées ainsi : un comptage ERC faux (« 5 + 9 » au lieu de 4 + 10) et une affirmation erronée sur le sens de la diode de structure.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre de broche.
- Architecture de protection retenue : `J1` → `F301` → `Q301` (anti-inversion P-MOS) → `D301` (TVS, transitoires rapides seulement) → `LM5069` + MOSFET série (surtension active et limitation d'appel de courant) → bulk → TPA3255.
- **Aucune TVS passive ne peut borner ce rail sous 65 V** : facteur de clamp requis 1,354 contre 1,3 à 1,6 pour la technologie avalanche. D'où l'OVP actif. Démonstration et relevés dans `docs/protection-48v.md`.
- `LM5066` écarté : sa télémétrie PMBus n'apporte rien ici, et la simplicité prime.
- Vias thermiques du PowerPAD `U6` traités en Phase E par calcul, pas par empreinte générique.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant les livrables H2.
- Fichiers projet KiCad versionnés dans Git ; artefacts volatils exclus par `.gitignore`.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym`, `sym-lib-table`, `fp-lib-table`, `HifiAmp_TPA3255_Local.pretty/`
- `docs/architecture.md`, `docs/power-block.md`, **`docs/protection-48v.md`**
- `reports/ERC_B1_gate_2026-08-31.json`, `reports/ERC_B2_regate_final-2026-08-31.json`

## NEXT ACTION

B2.6 — Capturer le bloc `LM5069` au schéma via `kicad-control` : créer le symbole `LM5069-2` (VSSOP-10) s'il n'existe pas en librairie, placer `U8`, le MOSFET série `Q302` (valeur `NEEDS_DATA`), le shunt `R_SNS` 4 mΩ et le réseau de programmation `R1`/`R2`/`R3` = 191 k / 5,11 k / 9,09 kΩ, `R_PWR` = 147 kΩ, `C_TIMER` = 3,9 µF. Insérer le bloc **entre `Q301` et le bulk** : la source de `Q301` doit alimenter un nouveau net intermédiaire, et `PVDD` devient la sortie du hot-swap. Relancer ensuite l'ERC pour le gate C2 final.
