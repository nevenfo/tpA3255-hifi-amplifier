# PROGRESS

## Phase actuelle

Phase D. **GATE C2 = PASS**, revérifié : ERC à 15 warnings et 0 erreur, jeu de messages identique à la baseline de C2.

## Tâche actuelle

D1.4 — Empreintes locales. Quatre composants restent sans empreinte : `F301`, `J2`, `J3`, `J4`.

## Dernière tâche validée

**D1.5 est terminée.** Les neuf passifs du bloc de protection sont assignés, et `C110`/`C210` ont été traités dans la foulée. Il ne reste que quatre composants sans empreinte, tous dans D1.4.

Validation :

- Les six empreintes de librairie ont été vérifiées présentes **sur disque**, pas via `search_footprints`.
- ERC relancé par `kicad-cli` avant et après écriture : 15 warnings, 0 erreur, même jeu de messages.
- Netlist exporté et relu : la topologie confirme le rôle de chaque passif, notamment `PVDD_PROT` → `R306` → `PVDD_SENSE` avec `VIN` et `SENSE` de `U8` de part et d'autre du shunt.
- Les quatre écritures MCP ont été relues au fichier par le principal.

### Trois défauts trouvés dans le travail initial, non signalés par son auteur

1. **Une référence fabricant inventée.** `C325` portait `MKS2C044701O00KSSD` : le code de tolérance « O » n'existe pas chez WIMA et la référence n'est présente chez aucun distributeur. Remplacée par `MKS2B044701K00KSSD`, 4,7 µF / 50 V / ± 10 %, attestée. La variante 63 V réellement commercialisée est à ± 20 % et devait être écartée : elle ramène la marge de démarrage à × 1,25, sous le × 1,30 que la conception s'est fixé. **C'est la tolérance qui choisit ce composant, pas la tension.**
2. **Un changement électrique silencieux.** `C325` était passé de 3,9 à 4,7 µF sans que la documentation suive, qui annonçait encore 3,9 µF en dix endroits et tous les chiffres dérivés. Le choix est bon — 3,9 µF est E24 et absent des séries film — mais il allonge `t_flt` de 20,5 %. Recalculé et documenté : marge au démarrage × 1,41 au lieu de × 1,30, exposition SOA de `Q302` portée de 318 à 422 ms.
3. **`C326` sans tension nominale** alors qu'il est le condensateur le plus contraint du schéma : il découple `VIN` de `U8` sur `PVDD_PROT`, qui atteint 93,6 V en écrêtage. Porté à `100nF/250V`.

### Ce qui dimensionne réellement ces passifs

Ce n'est presque jamais la puissance, c'est la **tension**. `R305` et `R307` voient 78,6 V et 87,1 V à leurs bornes pendant un écrêtage de `D301` : un 0603, tenu à 50 V, serait violé, d'où le 0805. `R308`, `R309` et `R310` ne voient que quelques volts et restent en 0603. Seul `R306` est dimensionné par la puissance, 0,95 W pendant ≤ 422 ms contre 1 W admis à 70 °C.

Contrôle qui recoupe le brochage au netlist : `R307` + `R308` + `R309` = 205,2 kΩ, donc à 56 V la prise `UVLO` est à 3,88 V (seuil 2,5 V, franchi) et `OVLO` à 2,48 V, juste sous son seuil. L'ordre du diviseur est donc bien celui qui déclenche légèrement au-dessus de 56 V.

## Contraintes nouvelles créées par ces choix

- **`R306` est un shunt à deux bornes, pas Kelvin.** À 4 mΩ, le cuivre des pastilles s'ajoute à la valeur mesurée. Les liaisons vers `VIN` et `SENSE` doivent partir des **bords intérieurs** des pastilles, le courant de puissance entrant par les bords extérieurs. Repli si le routage l'interdit : `WSK25122L000FEA`, quatre bornes, même boîtier 2512.
- `C110`/`C210` imposent un établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec la temporisation de mute en Phase F.
- `Q302` doit être monté sur radiateur : sa SOA suppose le boîtier à 75 °C. À reporter en Phase E.
- `Q301` : courant continu plafonné à 6,9 A avec la seule surface de cuivre de référence de sa datasheet, 6 cm² en 70 µm.
- Rail d'entrée : les calibres UMT-H supposent des pistes de 7,5 mm en cuivre 140 µm. Déclassement sinon.
- `F301` n'a aucune empreinte KiCad compatible (`Fuse_Schurter_UMT250` vise 3 × 10,1 mm contre 5,3 × 16 mm). Empreinte locale à créer en D1.4.

## Blocage actif

Aucun.

## NEEDS_DATA ouverts

`RV1` (mécanique du potentiomètre), confirmation par dessin mécanique TI que l'EP du TPA3255DDV vaut 5,2 × 14 mm, alimentation externe 48 V, inductances 15 µH et films 680 nF, dissipateur et thermique, réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33.

Assumés : stabilité de la boucle de limitation de puissance du LM5069 face aux 540 nC de grille de `Q302`, TI ne spécifiant aucune capacité de grille maximale ; `V_C` de la `SMDJ58CA` sous `I_PP`, non spécifiée.

Levés en D1.5 : `D302` (`BZT52C15`), `C110` et `C210` (10 µF/25 V, levés **par le calcul** — bruit thermique de la source de Thévenin de 5 kΩ et réjection de rail, aucune source extérieure n'était nécessaire).

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.** Confirmé une fois de plus en D1.5 : une référence fabricant inventée est passée dans le fichier faute de ce contrôle.
- **`kicad-cli.exe` est utilisable directement** (`sch erc`, `sch export netlist`) et fournit une preuve indépendante du MCP, sans GUI. Chemin : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **Outillage** : `pymupdf` installé, `pdftotext` dans `/mingw64/bin`. Les serveurs Littelfuse et DigiKey refusent les requêtes automatisées ; les miroirs tiers fonctionnent, mais l'identité de tout PDF récupéré doit être contrôlée sur son en-tête.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- TVS cantonnée aux transitoires rapides ; la protection en surtension est active, par `LM5069`.
- Bulk maintenu à 15 400 µF sur arbitrage utilisateur ; `Q302` choisi en conséquence.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.
- Instantanés PDF automatiques du MCP (`*_pre_delete_*.pdf`) exclus par `.gitignore`.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Les écritures sont fichier et persistées ; prouver par relecture.
- Attributs `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`.
- `search_footprints` n'indexe pas toute la librairie globale : vérifier sur disque.
- Sortie MCP tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Une recherche naïve de `(property "Reference" ...)` suivie d'une fenêtre de caractères déborde sur le symbole voisin et rend des valeurs fausses. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym` (symbole `LM5069` local), `sym-lib-table`, `fp-lib-table`, `HifiAmp_TPA3255_Local.pretty/`
- `docs/architecture.md`, `docs/power-block.md`, **`docs/protection-48v.md`**
- Librairies KiCad : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/share/kicad/footprints`

## Défaut ouvert, bloquant pour la Phase E

Cinq `PWR_FLAG` portent la référence `?` au lieu de `#FLG0x`. Sans effet ERC, mais `kicad-cli` signale déjà « erreurs de numérotation » et KiCad refuse « Update PCB from schematic » sur un schéma non annoté. Ouvert en D1.6. La correction par MCP est incertaine : les cinq symboles partagent la même référence `?`, donc l'adressage par repère est ambigu.

## Réserve de conception consignée, non tranchée

À 100 V, un P-canal reste environ trois fois moins bon qu'un N-canal. L'alternative serait un contrôleur de diode idéale (`LM74700`, `LM5050`) pilotant un N-canal, dont la pompe de charge fournit exactement la commande côté haut dont l'absence avait fait rejeter le N-canal en B2.3. Non retenue : la topologie P-MOS est tranchée et 0,70 W est acceptable. Consignée pour rester révisable.

## NEXT ACTION

D1.4 — Créer les empreintes locales manquantes et clore les quatre derniers composants. Deux natures différentes : `F301` exige une **empreinte locale** dans `HifiAmp_TPA3255_Local.pretty/`, aucune empreinte KiCad ne couvrant le corps 5,3 × 16 mm du Schurter UMT-H ; `J2`, `J3` et `J4` n'attendent qu'un **choix de famille de connecteur**, deux points pour `J2`/`J3` vers les RCA de châssis et six points pour `J4` vers le potentiomètre déporté. Ce choix relève de la préférence d'assemblage de l'utilisateur (barrette à vis, JST XH, Molex KK) et doit lui être posé avant d'assigner quoi que ce soit.
