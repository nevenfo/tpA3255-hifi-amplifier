# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS` inchangée, revérifiée cette session : `kicad-cli sch erc` remonte 15 violations, 0 erreur, jeu identique à la baseline C2.

## Tâche actuelle

**E1.2 — placer la puissance Class-D.** Deux étapes sur trois franchies : contour + import, puis placement du bloc de puissance. Reste les deux perçages M3 de la barre, volontairement différés après E1.8.

## Dernière tâche validée

**E1.2 étape 2 = PASS — bloc de puissance placé.** 21 empreintes posées : `U6` en `(285, 175)` **rotation 180°**, bootstrap `C306`–`C309`, découplages `PVDD` `C310`/`C311`, découplages bas niveau `C301`–`C305`/`C318`–`C320`, bulk `C312`–`C317`.

Validation, tenue au fichier par le principal indépendamment du rapport d'agent :

- L'empreinte **MD5 du `.kicad_pcb` a changé** — l'enregistrement a bien eu lieu, ce que le seul retour d'outil n'aurait pas prouvé.
- Les **21 positions et angles relus sont exacts**, sans écart à la table.
- Les **101 autres empreintes n'ont pas bougé** : toutes encore dans le bloc d'import.
- 122 empreintes, 122 références distinctes, contour `Edge.Cuts` intact.
- `kicad-cli pcb drc` : violations **391 → 137**, `schematic_parity` reste **0**, 253 non-connectés inchangés puisque rien n'est routé.

**Avant elle** : E1.2 étape 1 = PASS (import des 122 empreintes connectées, 74 nets), E1.7 = PASS, E1.6 = PASS, E1.1 = PASS, D1 CLOSE.

## Décisions actives

- **`U6` en rotation 180°, centre `(285, 175)`.** Les deux flancs du `HTSSOP-44` ne sont pas interchangeables : côté `x < 0` du symbole tout le bas niveau, côté `x > 0` toute la puissance — six `PVDD`, `OUT_A`–`D`, quatre `BST`, six `GND`. La rotation 180° tourne la puissance vers l'intérieur de la carte et laisse le bas niveau échapper vers la lisière droite.
- **La barre de liaison est dressée** : 10 mm d'épaisseur dans le plan de la carte, 60 mm de hauteur. Section de conduction inchangée à 600 mm², donc **0,250 à 0,333 °C/W inchangés**, mais l'ombre portée tombe à une bande de 10 mm. **Exigence non négociable qui en découle : la barre doit s'élargir en pied côté flanc pour y présenter au moins 1200 mm²**, sans quoi le terme d'isolation double et la marge tombe à 0,09 °C/W.
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

- `HifiAmp_TPA3255.kicad_pcb` — 122 empreintes connectées, contour 200 × 150, **21 placées**, 101 encore dans le bloc d'import `x` 221,7..321,0 / `y` 200,1..298,8
- `HifiAmp_TPA3255.kicad_sym` — contient `LM2940IMP_12_FIXED`, le **précédent à reproduire pour E1.8**
- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `sym-lib-table`, `fp-lib-table`
- `docs/architecture.md` — « E1.2 — le placement retourne la section de la barre », « Arbitrages rendus, et contour qui en découle », « E1.7 — la solution n'est pas un dissipateur, c'est le coffret »
- **État KiCad actuel** : processus `kicad.exe` PID 25092, éditeur de PCB ouvert depuis le gestionnaire, schéma fermé.

## NEXT ACTION

**E1.8 — raccorder le tab de `U3` à `/+3V3`.** Reproduire le précédent `LM2940IMP_12_FIXED` : créer un symbole local `TLV1117_33_FIXED` dans `HifiAmp_TPA3255.kicad_sym` portant la quatrième broche du `DCY`, y repointer `U3`, puis resynchroniser le PCB.

Séquence KiCad imposée : fermer l'éditeur de PCB, ouvrir l'éditeur de schéma **depuis le gestionnaire** pour n'avoir qu'un seul handler, faire l'édition, refermer, rouvrir l'éditeur de PCB depuis le gestionnaire, relancer « Mise à jour du PCB à partir du Schéma » — cette fois **en décochant « supprimer les empreintes sans symbole »** n'est pas nécessaire, aucune empreinte hors schéma n'existe encore.

Valider par : `pad 4` de `U3` portant `/+3V3` à la relecture du fichier, `schematic_parity` toujours à 0, ERC toujours à 15 violations / 0 erreur, et les 21 placements de E1.2 intacts. Enchaîner ensuite sur les deux perçages M3 pour clore E1.2, puis E1.9.
