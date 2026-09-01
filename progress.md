# PROGRESS

## Phase actuelle

Phase D. **GATE C2 = PASS**, revérifié après chaque écriture : ERC à 15 warnings et 0 erreur, jeu de messages strictement identique à la baseline de C2.

## Tâche actuelle

D1.2 — Vérifier boîtiers fabricant, orientations, courants et contraintes d'assemblage.

## Dernière tâche validée

**D1.1, D1.4, D1.5 et D1.6 sont terminées. Plus aucun composant du schéma n'est sans empreinte, et le schéma est intégralement annoté.** Les six `PWR_FLAG` ne requièrent pas d'empreinte, et `RV1` est volontairement hors carte (B2.1, B2.8).

Validation :

- Inventaire refait en itérant sur les blocs `(symbol` de premier niveau du fichier, pas via le MCP.
- Toutes les empreintes de librairie contrôlées présentes **sur disque**, `search_footprints` n'indexant pas tout.
- Netlist exporté et relu : `PVDD_PROT` → `R306` → `PVDD_SENSE`, avec `VIN` et `SENSE` de `U8` bien de part et d'autre du shunt.
- ERC relancé par `kicad-cli` après chaque lot d'écritures.
- Toutes les écritures MCP relues au fichier par le principal. Un agent a rapporté un succès sans avoir rien écrit : c'est la relecture qui l'a détecté.

### Trois défauts trouvés dans le travail D1.5 initial, non signalés par son auteur

1. **Une référence fabricant inventée.** `C325` portait `MKS2C044701O00KSSD` : le code de tolérance « O » n'existe pas chez WIMA, la référence n'est chez aucun distributeur. Remplacée par `MKS2B044701K00KSSD`, 4,7 µF / 50 V / ± 10 %. La variante 63 V réellement commercialisée est à ± 20 % et devait être écartée : elle ramène la marge de démarrage à × 1,25, sous le × 1,30 fixé. **C'est la tolérance qui choisit ce composant, pas la tension.**
2. **Un changement électrique silencieux.** `C325` était passé de 3,9 à 4,7 µF sans que la documentation suive, qui annonçait encore 3,9 µF en dix endroits avec tous les chiffres dérivés. Le choix est bon — 3,9 µF est E24, absent des séries film — mais il allonge `t_flt` de 20,5 %. Recalculé : marge au démarrage × 1,41 au lieu de × 1,30, exposition SOA de `Q302` portée de 318 à 422 ms, toujours très loin dans une courbe garantie à 5 s.
3. **`C326` sans tension nominale** alors qu'il découple `VIN` de `U8` sur `PVDD_PROT`, le nœud le plus contraint du schéma, qui atteint 93,6 V en écrêtage. Porté à `100nF/250V`.

### Ce qui dimensionne réellement ces passifs

Ce n'est presque jamais la puissance, c'est la **tension**. `R305` et `R307` voient 78,6 V et 87,1 V à leurs bornes pendant un écrêtage de `D301` : un 0603, tenu à 50 V, serait violé, d'où le 0805. `R308`, `R309` et `R310` ne voient que quelques volts et restent en 0603. Seul `R306` est dimensionné par la puissance, 0,95 W pendant ≤ 422 ms contre 1 W admis à 70 °C.

Contrôle qui recoupe le brochage au netlist : `R307` + `R308` + `R309` = 205,2 kΩ, donc à 56 V la prise `UVLO` est à 3,88 V (seuil 2,5 V, franchi) et `OVLO` à 2,48 V, juste sous son seuil. L'ordre du diviseur est bien celui qui déclenche légèrement au-dessus de 56 V.

### Méthode qui a débloqué le fusible

La recherche web a rendu les chiffres du dessin d'implantation Schurter **sans pouvoir les attribuer** à une cote, les légendes n'étant pas extractibles. L'attribution a été faite en extrayant la géométrie vectorielle du PDF : les deux pastilles, leurs arêtes et les flèches de cote. Le tracé s'est révélé **non à l'échelle** — rapport pastille/écartement mesuré à 0,312 contre 0,375 aux étiquettes —, donc les étiquettes font foi. Trois recoupements indépendants les confirment : l'écart de 10,00 mm encadre les 9,80 mm de céramique nue, chaque pastille couvre 2,70 des 2,80 mm de terminaison, et déborde de 1,05 mm en bout. Un tracé pris à l'échelle aurait amputé le recouvrement de 20 %.

### D1.6 — le défaut d'annotation était plus grave que consigné

Les cinq `PWR_FLAG` à référence `?` n'étaient pas une gêne cosmétique : ils étaient **exportés au netlist comme six composants réels nommés `?`**, que « Update PCB from schematic » aurait tenté de placer sur la carte, sans empreinte. Annotés `#FLG01` à `#FLG05`, ils en sont désormais exclus comme tout symbole à préfixe `#`.

Ce qui a débloqué l'écriture : **`edit_schematic_component` accepte un paramètre `uuid`**, alternatif à `reference`. C'est ce qui lève l'ambiguïté quand plusieurs symboles partagent un repère, et c'est réutilisable pour tout adressage ambigu.

Preuve retenue : netlist exporté avant et après, 74 nets de part et d'autre, aucun net créé ni supprimé, aucun nœud de composant réel déplacé, et pour seule différence les cinq `?.1` retirés de `/+12V-OA`, `/PVDD`, `/GND`, `/+15V` et `/BUCK_VIN`. `kicad-cli` ne signale plus « erreurs de numérotation ».

Les références `U4` et `U5` apparaissent trois fois chacune et **ne sont pas un défaut** : ce sont les unités des AOP doubles.

## Contraintes nouvelles créées par ces choix

- **`R306` est un shunt à deux bornes, pas Kelvin.** À 4 mΩ, le cuivre des pastilles s'ajoute à la valeur mesurée. Les liaisons vers `VIN` et `SENSE` doivent partir des **bords intérieurs** des pastilles, le courant de puissance entrant par les bords extérieurs. Repli si le routage l'interdit : `WSK25122L000FEA`, quatre bornes, même boîtier 2512.
- `C110`/`C210` imposent un établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec la temporisation de mute en Phase F.
- `Q302` doit être monté sur radiateur : sa SOA suppose le boîtier à 75 °C. À reporter en Phase E.
- `Q301` : courant continu plafonné à 6,9 A avec la seule surface de cuivre de référence de sa datasheet, 6 cm² en 70 µm.
- Rail d'entrée : les calibres UMT-H supposent des pistes de 7,5 mm en cuivre 140 µm. Déclassement sinon.
- Connectique déportée en JST XH : une deuxième famille à approvisionner à côté des MaiXu MX126-5.0, et une pince à sertir nécessaire.

## Blocage actif

Aucun.

## Défaut ouvert

**D1.8, en attente d'un arbitrage utilisateur.** Les deux empreintes locales ont des graphiques imposés par le générateur `create_footprint` du MCP. Sur `CF_Film_Box_P5.00mm_7.2x3.5mm`, utilisée par `C321` à `C324`, le **courtyard fait 2,6 mm pour un corps de 3,5 mm** : le DRC ne signalera pas un composant placé trop près en Phase E. Sur `Fuse_Schurter_UMT-H_5.3x16mm`, cuivre, pâte, masque et courtyard sont exacts, mais le générateur ajoute un repère de broche 1 sur un composant non polarisé, dont le cercle tombe hors du courtyard. **Aucun outil MCP n'édite les lignes, rectangles, textes ou tags d'une empreinte de bibliothèque** — `edit_footprint_pad` ne touche que les pastilles, les toolsets `pcb_*` n'opèrent que sur un `.kicad_pcb`. Corriger exige soit une dérogation ponctuelle à la règle « toute édition de librairie passe par le MCP », soit une version du MCP exposant l'édition des graphiques.

## NEEDS_DATA ouverts

`RV1` (mécanique du potentiomètre), confirmation par dessin mécanique TI que l'EP du TPA3255DDV vaut 5,2 × 14 mm, alimentation externe 48 V, inductances 15 µH et films 680 nF, dissipateur et thermique, réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33.

Assumés : stabilité de la boucle de limitation de puissance du LM5069 face aux 540 nC de grille de `Q302`, TI ne spécifiant aucune capacité de grille maximale ; `V_C` de la `SMDJ58CA` sous `I_PP`, non spécifiée.

Levés en D1.5 : `D302` (`BZT52C15`), `C110` et `C210` (10 µF/25 V, levés **par le calcul** — bruit thermique de la source de Thévenin de 5 kΩ et réjection de rail, aucune source extérieure n'était nécessaire).

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.** Confirmé deux fois en D1.5/D1.4 : une référence fabricant inventée est passée dans le fichier faute de ce contrôle, et un agent a rapporté une assignation qu'il n'avait pas faite.
- **`kicad-cli.exe` est utilisable directement** (`sch erc`, `sch export netlist`) et fournit une preuve indépendante du MCP, sans GUI. Chemin : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **Lire les dessins cotés par extraction vectorielle du PDF, pas par recherche web.** `pymupdf` est installé, `pdftotext` est dans `/mingw64/bin`. Méthode validée sur les courbes SOA puis sur le dessin d'implantation Schurter. **Toujours contrôler si le tracé est à l'échelle** avant d'en déduire une cote : celui de Schurter ne l'est pas.
- Les serveurs Littelfuse et DigiKey refusent les requêtes automatisées ; les miroirs tiers fonctionnent, mais l'identité de tout PDF récupéré doit être contrôlée sur son en-tête.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- TVS cantonnée aux transitoires rapides ; la protection en surtension est active, par `LM5069`.
- Bulk maintenu à 15 400 µF sur arbitrage utilisateur ; `Q302` choisi en conséquence.
- Connectique déportée : JST XH 2,5 mm, vertical par défaut, sur arbitrage utilisateur.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.
- Instantanés PDF automatiques du MCP (`*_pre_delete_*.pdf`) exclus par `.gitignore`.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Les écritures sont fichier et persistées ; prouver par relecture.
- Attributs `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées.
- **`create_footprint` impose ses propres graphiques** : `F.Fab` chanfreiné, `F.SilkS` plein avec cercle de broche 1, textes à ± 4,05, pas de `tags`. Non paramétrable, et aucun outil ne les édite après coup.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`.
- **`edit_schematic_component` accepte `uuid` en plus de `reference`** : seul moyen d'adresser un symbole quand plusieurs partagent le même repère.
- `search_footprints` n'indexe pas toute la librairie globale : vérifier sur disque.
- Sortie MCP tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Une recherche naïve de `(property "Reference" ...)` suivie d'une fenêtre de caractères déborde sur le symbole voisin et rend des valeurs fausses. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb`
- `HifiAmp_TPA3255.kicad_sym` (symbole `LM5069` local), `sym-lib-table`, `fp-lib-table`
- `HifiAmp_TPA3255_Local.pretty/` : `CF_Film_Box_P5.00mm_7.2x3.5mm`, `Fuse_Schurter_UMT-H_5.3x16mm`
- `docs/architecture.md`, `docs/power-block.md`, **`docs/protection-48v.md`**
- Librairies KiCad : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/share/kicad/footprints`

## Réserve de conception consignée, non tranchée

À 100 V, un P-canal reste environ trois fois moins bon qu'un N-canal. L'alternative serait un contrôleur de diode idéale (`LM74700`, `LM5050`) pilotant un N-canal, dont la pompe de charge fournit exactement la commande côté haut dont l'absence avait fait rejeter le N-canal en B2.3. Non retenue : la topologie P-MOS est tranchée et 0,70 W est acceptable. Consignée pour rester révisable.

## NEXT ACTION

D1.2 — Vérifier boîtiers fabricant, orientations, courants et contraintes d'assemblage sur les empreintes désormais toutes assignées. Trois points sont déjà identifiés comme non triviaux et doivent être traités en priorité : **D1.7**, confirmer sur le dessin coté WIMA que l'épaisseur de corps du `MKS2B044701K00KSSD` vaut bien 7,2 mm, cote prise comme maximum de la série MKS2 au pas de 5 mm mais non lue à la source, une erreur donnant un composant qui n'entre pas dans son empreinte ; l'orientation verticale de `Q301` et `Q302`, retenue par défaut, la variante à semelle plaquée dépendant du radiateur ; et les boîtiers traversants des bulks `C312` à `C317`, dont l'encombrement conditionne le placement en Phase E.
