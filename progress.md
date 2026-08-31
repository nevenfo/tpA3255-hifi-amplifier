# PROGRESS

## Phase actuelle

Phase B2. **GATE C2 = PASS** (inchangé : seules des `Value` et propriétés ont été écrites, sans effet ERC).

## Tâche actuelle

B2.9 — Figer `Q301`, le P-MOS d'anti-inversion.

## Dernière tâche validée

**B2.5 et B2.7 — les trois composants critiques du bloc de protection sont figés sur datasheet**, écritures relues au fichier par le principal.

| Repère | Référence | Preuve |
|---|---|---|
| `Q302` | `IXTK200N10L2` (Littelfuse/IXYS *Linear L2*, TO-264) | SOA **garantie** 625 W à `T_C` = 75 °C / `t_p` = 5 s, contre 389 W exigés — marge 1,61 × |
| `F301` | `Schurter UMT-H` 12,5 A, `3403.0285.11` | 125 VDC, coupure 1000 A en continu ; coordination vérifiée sur la table *Pre-Arcing Time* |
| `D301` | `SMDJ58CA`, 3000 W, DO-214AB | `V_RWM` = 58 V > 56 V ; écrête sous le plafond de 88 V du LM5069 ; **bidirectionnelle** |

Analyse complète, méthode et réserves : section finale de `docs/protection-48v.md`.

### Ce qui a réellement fait basculer les choix

1. **Les courbes SOA sont des images.** Deux recherches web ont échoué pour cette seule raison. La lecture a été faite par **extraction vectorielle des tracés du PDF**, recalés sur les étiquettes d'axes. Méthode validée trois fois : `I_DM` retrouvé à 140,2 A contre 140 A au tableau, et sur les quatre datasheets la valeur déduite coïncide à moins de 1 % près avec la ligne **SOA garantie** du tableau *Safe Operating Area Specification*.
2. **Le candidat évident échoue.** L'`IXTH64N10L2` ne tient que 215 W à 75 °C, soit 0,55 × le besoin. Il faut un die environ trois fois plus gros que ce que 4,6 A nominaux laisseraient supposer — conséquence directe et chiffrée du bulk de 15 400 µF conservé.
3. **Le déclassement de l'équation 19 est devenu inutile** : IXYS publie la courbe directement à `T_C` = 75 °C et garantit une valeur testée à cette température.
4. **La TVS doit être bidirectionnelle.** Une `SMDJ58A` unidirectionnelle, anode sur `GND`, entre en conduction directe sur une inversion d'alimentation, court-circuite la source et fait fondre `F301` — ce qui annule la fonction même de `Q301`. La variante `CA` partage exactement les mêmes caractéristiques électriques et rétablit la cohérence avec le symbole `Device:D_TVS`, déjà bidirectionnel.
5. **Note (3) des maxima absolus du LM5069** : `GATE` flotte 12 V au-dessus de `VIN`, donc le plafond de service est **88 V**, pas 100 V. C'est ce qui écarte la `SMCJ58A` (93,6 V dès 16 A) au profit du boîtier 3000 W.
6. **Un calibre de fusible trop bas romprait la coordination.** À 12,5 A, une limitation de courant LM5069 à 15,4 A pendant 318 ms vaut 1,23 × `In`, pour lequel la datasheet impose ≥ 60 min avant amorçage : ouverture impossible. `F301` est un ultime recours contre un `Q302` en court-circuit, jamais une protection de surcharge.

## Contraintes nouvelles créées par ces choix

- `Q302` doit être **monté sur radiateur** : la SOA suppose le boîtier maintenu à 75 °C. À reporter en Phase E.
- `Q301` doit **bloquer l'inversion à 56 V plus marge**. Attention : il n'est *pas* contraint à `V_DS` ≥ 100 V par l'écrêtage — il conduit pendant celui-ci, grille tenue 15 V sous sa source, donc son `V_DS` reste voisin de zéro. Première déduction fausse, corrigée. C'est l'objet de B2.9.
- Rail d'entrée : les calibres UMT-H sont établis sur pistes de 7,5 mm en cuivre 140 µm. Déclassement sinon.
- `F301` n'a **aucune empreinte KiCad compatible** (`Fuse_Schurter_UMT250` vise 3 × 10,1 mm contre 5,3 × 16 mm). Empreinte locale à créer en D1.4.

## Blocage actif

Aucun.

## NEEDS_DATA ouverts

`Q301` (B2.9), `D302`, `C110`/`C210`, potentiomètre `RV1`, confirmation par dessin mécanique TI que l'EP du TPA3255DDV vaut 5,2 × 14 mm.

Nouveaux, assumés : stabilité de la boucle de limitation de puissance du LM5069 face aux 540 nC de grille de `Q302`, TI ne spécifiant aucune capacité de grille maximale ; `V_C` de la `SMDJ58CA` en dessous de `I_PP`, non spécifiée.

Levés cette session : `Q302` et sa courbe SOA, `R_DS(on)` et résistance thermique du MOSFET, `F301`, `D301`.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.** Cette session : les deux recherches web ont rendu un résultat honnêtement vide sur la SOA, et un PDF récupéré chez un distributeur s'est révélé être un composant sans rapport. Le rapport `kicad-control`, lui, s'est vérifié exact.
- **Outillage acquis, à réutiliser** : `pymupdf` est installé, et `pdftotext` est présent dans `/mingw64/bin`. Cela permet de lire réellement les datasheets — texte, rendu de page en image, et extraction vectorielle des courbes. Les serveurs Littelfuse et DigiKey refusent les requêtes automatisées ; les miroirs tiers fonctionnent, mais **l'identité de tout PDF récupéré doit être contrôlée sur son en-tête** avant exploitation.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- TVS cantonnée aux transitoires rapides ; la protection en surtension est **active**, par `LM5069`.
- Bulk maintenu à 15 400 µF sur arbitrage utilisateur ; `Q302` choisi en conséquence.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.
- Instantanés PDF automatiques du MCP (`*_pre_delete_*.pdf`) exclus par `.gitignore`.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Les écritures sont fichier et persistées ; prouver par relecture.
- **Attributs `on_board` / `in_bom` / `dnp` inaccessibles** ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées.
- `add_power_symbol` : `power_net` désigne le **nom du symbole de librairie**, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un **chemin absolu** et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`.
- `search_footprints` n'indexe pas toute la librairie globale : vérifier sur disque.
- Sortie MCP tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Une recherche naïve de `(property "Reference" ...)` suivie d'une fenêtre de caractères déborde sur le symbole voisin et rend des valeurs fausses. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym` (symbole `LM5069` local), `sym-lib-table`, `fp-lib-table`, `HifiAmp_TPA3255_Local.pretty/`
- `docs/architecture.md`, `docs/power-block.md`, **`docs/protection-48v.md`**
- Librairies KiCad : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/share/kicad/footprints`
- `reports/ERC_C2_gate_final-2026-08-31.json`

## NEXT ACTION

B2.9 — Retenir un P-MOS `Q301` réel bloquant l'inversion à 56 V plus marge, `I_D` ≥ 12 A continus, `R_DS(on)` faible sous `V_GS` = −15 V (grille clampée par `D302`), en boîtier dissipatif. Renseigner sa `Value` via `kicad-control`, puis enchaîner sur D1.1 en assignant les empreintes désormais débloquées : `D301` → `Diode_SMD:D_SMC`, `Q302` → `Package_TO_SOT_THT:TO-264-3_*`.
