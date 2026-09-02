# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`. Baseline ERC : **16 violations, 0 erreur** — 10 `endpoint_off_grid` et 6 `lib_symbol_mismatch`, ces derniers étant le prix des symboles locaux du projet. Baseline DRC : **138 violations, 254 non-connectés, `schematic_parity` = 0**.

## Tâche actuelle

**E1.9 — la règle DRC d'isolation interne aux boîtiers à pas fin.** Doit faire tomber les 42 violations `clearance`, toutes intra-empreinte. Non commencée.

## Dernière tâche validée

**E1.2 = PASS, close par ses deux perçages M3.** `H1` en `(285.000, 163.000)` et `H2` en `(285.000, 187.000)`, entraxe 24 mm, encadrant `U6` — **motif validé par l'utilisateur** contre les variantes 30 mm et trois points. Empreintes `MountingHole:MountingHole_3.2mm_M3`, `F.Cu`, rotation 0, sans net ni symbole au schéma.

**Deux défauts de `add_mounting_hole` corrigés au passage** : l'outil forge un `fpid` `MountingHole:MountingHole_3.2mm` inexistant en bibliothèque, et écrit `(size 3.7 3.7)` pour un `(drill 3.2)` — soit un anneau de cuivre flottant de 0,25 mm sur toutes les couches, sous la vis et la barre d'aluminium nu. Le second défaut avait échappé au rapport d'agent, vu à la relecture du fichier. Corrigé par `place_component` en IPC avec l'empreinte réelle, qui pose `(size 3.2 3.2)`, cuivre nul, courtyard compris.

Validation, tenue au fichier par le principal :

- MD5 du `.kicad_pcb` `1dc8f28c…` → `2eba885c…` : l'enregistrement a bien eu lieu.
- 124 empreintes, exactement deux références `H*` sans doublon, coordonnées, couche et rotation exactes, aucun net.
- Les **21 placements de E1.2 intacts au micron**, les 101 empreintes du bloc d'import inchangées.
- `kicad-cli pcb drc` : `schematic_parity` toujours **0**, 254 non-connectés inchangés, et **aucune des 138 violations n'implique `H1` ni `H2`**.

**Avant elle** : E1.8, E1.7, E1.6, E1.1 = PASS, D1 CLOSE.

## Décisions actives

- **`U6` en rotation 180°, centre `(285, 175)`.** Les deux flancs du `HTSSOP-44` ne sont pas interchangeables : côté `x < 0` du symbole tout le bas niveau, côté `x > 0` toute la puissance — six `PVDD`, `OUT_A`–`D`, quatre `BST`, six `GND`. La rotation 180° tourne la puissance vers l'intérieur de la carte et laisse le bas niveau échapper vers la lisière droite.
- **La barre de liaison est dressée** : 10 mm d'épaisseur dans le plan de la carte, 60 mm de hauteur. Section de conduction inchangée à 600 mm², donc **0,250 à 0,333 °C/W inchangés**, mais l'ombre portée tombe à une bande de 10 mm. **Exigence non négociable qui en découle : la barre doit s'élargir en pied côté flanc pour y présenter au moins 1200 mm²**, sans quoi le terme d'isolation double et la marge tombe à 0,09 °C/W.
- **Fixation de la barre arrêtée** : deux M3 de passage en `(285, 163)` et `(285, 187)`, **entraxe 24 mm**, encadrant `U6`. Deux vis et non une, pour empêcher la barre de pivoter sur le PowerPAD. **La barre se termine donc en pied élargi** côté `U6` : la lame de 10 mm s'ouvre à ≈ 30 mm de large sur `x` de 280 à 291, pour y porter deux taraudages au même entraxe. La zone d'interdiction s'élargit d'autant : `x` ∈ [280, 291], `y` ∈ [160, 190] côté `U6`. Aucun site de composant n'y tombe — voisin le plus proche `C303` à 8 mm, `U6` à 12 mm de chaque trou.
- **Les règles DRC personnalisées vivent dans `<projet>.kicad_dru`**, jamais dans `board.design_settings.rules` du `.kicad_pro`, qui ne porte que des minima numériques. Le plan affirmait le contraire ; corrigé.
- **Zone d'interdiction de composants** : `x` ∈ [280, 300], `y` ∈ [169, 181]. Pas seulement une limite de hauteur — une pièce d'aluminium nu à 1,2 mm du cuivre n'est pas acceptable au-dessus de pastilles. Les découplages bas niveau sont donc rangés au-dessus de `y` = 166,5 et au-dessous de `y` = 183,5.
- **Contour arrêté à 200 × 150 mm**, `(100,100)`–`(300,250)`, contraint par le placement et non par le coffret ; resserrable après E1.5.
- **Arbitrages de E1.7 rendus** : coffret **Modushop `03/300` 3U**, carte à plat, isolation reportée à la jonction barre/flanc. Chaîne 1,711 à 1,932 °C/W pour 2,232 de budget — à recorriger selon la réserve du pied de barre ci-dessus.
- **`REQ-THERM-3`** : nominal continu 2 × 100 W sur **8 Ω**, le 4 Ω en crête seulement.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de configuration s'éditent directement**.
- **`set_design_rules` et `set_active_layer` sont PROSCRITS** : jetons invalides dans `(setup ...)`, fichier illisible par KiCad, aucun retour d'erreur. Les règles vivent dans le `.kicad_pro`, sous `board.design_settings.rules`.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.** Appliqué trois fois cette session ; a notamment permis de prouver l'enregistrement par le MD5 plutôt que de croire le retour d'outil.
- **`kicad-cli.exe` fournit une preuve indépendante du MCP** — `sch erc`, `pcb drc --format json`. Chemin `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **Lire les cotes par extraction du PDF fabricant**, jamais par listing distributeur ; contrôler l'échelle et **les conditions de mesure**. **PDF sans couche texte** : rendre en PNG par `pymupdf`. **Écrire l'extraction dans un fichier UTF-8 avant affichage**, la console Windows casse sur les accents.
- Bulk maintenu à 15 400 µF. Asymétrie de nommage assumée `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite, à trancher avant H2.

## Blocage actif

Aucun.

## Pilotage de KiCad — appris à la dure cette session

**Le MCP ne sait pas tout faire, et l'IPC est capricieux. Cette séquence est la seule qui fonctionne :**

1. **Aucun des 203 outils du MCP ne sait synchroniser schéma → PCB.** `kicad-cli` non plus. Seule voie : **Outils → « Mise à jour du PCB à partir du Schéma »** dans l'éditeur de PCB, en action GUI.
2. **`move_component` / `rotate_component` ne fonctionnent qu'en IPC**, pas en mode fichier. `place_component` est en mode fichier mais **n'a aucun champ net**.
3. **L'éditeur de PCB doit être ouvert depuis le gestionnaire de projet.** Un `pcbnew.exe` lancé isolément crée un processus séparé **qui ne partage pas le canal IPC** — l'IPC ne le voit pas. Lancé en autonome, il refuse en plus la synchronisation au schéma.
4. **Un seul document ouvert à la fois.** Schéma et PCB ouverts ensemble donnent `KiCad document context is ambiguous: expected exactly one PCB or schematic handler, found 2`. Fermer l'autre fenêtre par son handle Win32 suffit — la fenêtre du gestionnaire, elle, ne compte pas comme handler.
5. `kicad_common.json` porte déjà `api.enable_server = true` ; il manquait seulement le processus.
6. **Toujours prouver un enregistrement par le MD5 du fichier**, jamais par le retour de `save_project`.
7. **Ne jamais lancer KiCad depuis le shell de l'agent : il est élevé, KiCad hérite du jeton administrateur, et UIPI bloque alors silencieusement tout clic et toute frappe** venant d'une session d'automatisation non élevée — les appels rapportent un faux succès. **Lancer par `explorer.exe <chemin du projet>`**, qui s'exécute avec le jeton utilisateur normal. Contrôler le résultat en lisant le `TokenElevation` du processus : il doit valoir `normal`.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyses dans `reports/MCP_BUG-documenttype-routing-eeschema.md` et `reports/MCP_BUG-setup-tokens-kicad-pcb.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`.
- `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées, et accepte `uuid` en plus de `reference`.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : dans le format KiCad 10, un bloc d'empreinte porte `(layer ...)` et `(uuid ...)` **avant** `(at x y rot)` — une regex qui attend `(at` juste après `(footprint` échoue. Les pastilles portent `(net "NOM")` **sans identifiant numérique**, et il n'y a pas de table de nets en fin de fichier. Dans le `.kicad_sch`, `lib_symbols` précède les instances : itérer sur les blocs de premier niveau à parenthèses équilibrées.

## Contraintes de placement restantes

- **`U6` se refroidit uniquement par le dessus**, aucun via thermique sous le boîtier. La broche 45 du symbole n'a pas de pastille dans `HTSSOP-44_…_TopEP` : l'erreur d'import à ce sujet est **attendue et correcte**.
- **Orientation dans le coffret** : `U6` contre le flanc droit ; analogique bas niveau au bord opposé ; sorties haut-parleur et entrée 48 V sur l'arête arrière ; `J4` vers la façade. `J2`/`J3`/`J4` sont en **JST XH déporté**, leur panneau est donc un choix de câblage, pas une contrainte de carte.
- **Hauteurs** : tores `L301`–`L304` debout ø28,6 × 29 mm, 3,1 W de pertes cuivre ; `C321`–`C324` films 18 × 8 × 15 mm ; `C312`–`C315` ø18 × 35 mm ; `C316`/`C317` ø35 × **30 mm** ; `C325` 18 mm.
- **`R306` est un shunt à deux bornes, pas Kelvin** : `VIN`/`SENSE` depuis les bords **intérieurs** des pastilles.
- `Q302` en TO-264 sur radiateur, SOA supposant le boîtier à 75 °C. `Q301` en TO-220 : 6,9 A avec 6 cm² de cuivre 70 µm **sur son net de drain**.
- `C110`/`C210` : établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec le mute en Phase F.

## NEEDS_DATA ouverts

- Alimentation 48 V : volet tension clos par `REQ-PSU-1` ; restent ripple, courant continu garanti, démarrage.
- `RV1` mécanique — seul symbole sans empreinte ; réponse/EMI du filtre LC ; common-mode du TPA3255 ; broche MR du TPS3802K33.
- Plan de perçage de l'embase `01/05` non publié.
- Fabricant de PCB non choisi. **Épaisseur de cuivre par couche non exposée par le MCP**, pas de bloc `stackup` explicite : à rendre explicite dans Board Setup et à la commande.
- `EEU-FC1J152` (`C312`–`C315`) non décodée à la source — `industrial.panasonic.com` refuse `curl`.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — **124 empreintes**, contour 200 × 150, **21 placées + H1/H2**, 101 encore dans le bloc d'import `x` 221,7..321,0 / `y` 200,1..298,8
- `HifiAmp_TPA3255.kicad_dru` — **à créer en E1.9**, n'existe pas encore
- `HifiAmp_TPA3255.kicad_sym` — contient `LM2940IMP_12_FIXED`, le **précédent à reproduire pour E1.8**
- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `sym-lib-table`, `fp-lib-table`
- `docs/architecture.md` — « E1.2 — le placement retourne la section de la barre », « Arbitrages rendus, et contour qui en découle », « E1.7 — la solution n'est pas un dissipateur, c'est le coffret »
- **État KiCad actuel** : processus `kicad.exe` **PID 14180**, éditeur de PCB ouvert depuis le gestionnaire, **schéma fermé** — fermé par `WM_CLOSE` sur son handle Win32 pour lever le « document context is ambiguous » qui bloquait `save_project`. Le titre de fenêtre porte un `*` tant que des modifications live ne sont pas enregistrées : c'est un indicateur fiable.

## NEXT ACTION

**E1.9 — écrire la règle DRC d'isolation interne aux boîtiers à pas fin.** Créer `HifiAmp_TPA3255.kicad_dru` avec une règle `clearance` de **0,15 mm** conditionnée aux paires de pastilles d'une **même empreinte**, via `A.memberOfFootprint('U6') && B.memberOfFootprint('U6')` et de même pour `U1` et `U8`. Abaisser conjointement `min_clearance` de `0.2` à `0.15` dans `board.design_settings.rules` du `.kicad_pro`, sans quoi le plancher absolu annule la règle sur les 8 cas à 0,150 mm ; les classes de nets restent inchangées à 0,25 et 0,50 mm.

Écarts réels à couvrir : `0,235 mm` × 26, `0,200 mm` × 8, `0,150 mm` × 8, sur `U6` (26), `U1` (8), `U8` (8).

Valider par `kicad-cli pcb drc --format json` : **les 42 violations `clearance` tombent à 0**, le total passe de 138 à 96, `schematic_parity` reste à 0, les 254 non-connectés sont inchangés, et aucune violation nouvelle n'apparaît — en particulier aucune entre empreintes distinctes.

Le `.kicad_pro` étant un fichier de configuration, il s'édite directement ; mais **KiCad l'a ouvert**, donc le fermer avant l'édition pour qu'il ne l'écrase pas.
