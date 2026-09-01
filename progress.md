# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS` inchangée : ERC à 15 violations, 0 erreur, jeu identique à la baseline C2. Aucun fichier KiCad de connectivité touché depuis E1.1.

## Tâche actuelle

E1.2 — placer la puissance Class-D. **Bloquée sur trois arbitrages utilisateur.** E1.7 a levé le `NEEDS_DATA` au sens du critère de sélection : la famille de solution est identifiée, sourcée et chiffrée avec marge. Ce qui manque n'est plus une donnée introuvable mais un choix.

## Dernière tâche validée

**E1.7 = PASS.** Trois découvertes, dont deux rectifient E1.6.

1. **Le dissipateur de l'EVM ne tient pas l'exigence.** `ATS-TI1OP-519-C1-R3` : sa fiche `qats.com` ne publie **aucune valeur à vitesse d'air nulle** et plafonne à 2,2 °C/W sous 1 m/s, soit `T_C` = 102 °C contre 75 visés. En reste la mécanique : deux taraudages M3, entraxe 36,8 mm, boulonnage par le dessous à travers le PCB.
2. **La résistance d'étalement manquait au budget.** Les 22,4 W entrent par 29 mm² : au mieux 0,411 °C/W en base aluminium, 0,211 en cuivre. La cible d'un `RθSA` de catalogue passe de 1,0 à **0,58 °C/W**. `REQ-THERM-1` est reformulé sous forme invariante : **interface + étalement + dissipateur ≤ 1,5625 °C/W**.
3. **La solution est le coffret, pas un dissipateur rapporté.** Un flanc de coffret évacue vers les 25 °C de la pièce, pas vers les 40 °C internes : 0,67 °C/W de budget regagné. Modushop/HiFi 2000 *Pesante Dissipante*, document fabricant : **0,45 °C/W par flanc en 2U/300**, 0,41 en 3U/300, 0,31 en 4U/300, 0,23 en 4U/400. Largeur intérieure 360 mm sur tous les modèles, hauteurs 80/120/165/210, profondeurs 300/400. **Même le plus petit modèle donne `T_C` = 57 °C**, et laisse 0,80 °C/W pour la barre de liaison.

**Avant elle : E1.6 = PASS** (22,4 W à 2 × 100 W sur 8 Ω, `REQ-THERM-1`/`-2`), **E1.1 = PASS** (4 couches, 8 classes, 71 affectations, contour délibérément non tracé), **D1 CLOSE**.

## Décisions actives nouvelles

- **`REQ-THERM-3`** : le régime nominal continu est 2 × 100 W sur **8 Ω** ; le 4 Ω est admis **en crête seulement**. Arbitrage utilisateur. Le dimensionnement électrique reste établi sur 4 Ω — c'est un plafond thermique, pas électrique. Garde-fou matériel : `OTW` puis coupure du TPA3255.
- **Courte et épaisse, ou rien.** Une barre de liaison de 50 mm en section 10 × 60 coûte 0,417 °C/W ; une équerre de 60 mm en 6 × 40 coûte 1,250 et dépasse à elle seule le budget.
- **Le PowerPAD est `GND`** : le presser sur un flanc relie la masse signal au châssis. Trois issues chiffrées dans `docs/architecture.md`, dont un intercalaire **AlN 0,5 mm à 0,101 °C/W**.
- **`qats.com/DataSheet/<référence>` sert le PDF à `curl`.** **`fischerelektronik.de` refuse tout**, 403 même avec en-tête navigateur ; `tme.eu` 403. Fischer est non vérifiable à la source primaire sur ce poste.
- L'agent `web-research` n'a ouvert aucun PDF fabricant sur cette tâche ; les candidats retenus l'ont été par vérification directe du principal.

## Blocage actif

Aucun blocage technique. **Trois arbitrages utilisateur en attente**, portés dans la NEXT ACTION.

## Contraintes portées en Phase E

- **`U6` se refroidit uniquement par le dessus**, `RθJC(bot)` = `n/a`, aucun via thermique sous le boîtier. **L'import PCB signalera la broche 45 sans pastille : c'est attendu.**
- **L'emprise au sol d'un dissipateur posé sur la puce est une zone d'interdiction de hauteur**, pas un simple dégagement : sa base est à ≈ 1 mm du PCB, seuls des 0603 passent dessous. C'est ce qui disqualifie le dissipateur rapporté et impose le couplage au châssis.
- **Hauteurs** : tores debout ø28,6 × 29 mm, **3,1 W de pertes cuivre** ; films de sortie 18 × 8 × 15 mm ; `C312`–`C315` ø18 × 35 mm ; `C316`/`C317` ø35 × **30 mm** ; `C325` 18 mm.
- **`R306` est un shunt à deux bornes, pas Kelvin.** `VIN`/`SENSE` depuis les bords **intérieurs** des pastilles. Repli `WSK25122L000FEA`.
- `Q302` sur radiateur, SOA supposant le boîtier à 75 °C. `Q301` : 6,9 A avec 6 cm² de cuivre 70 µm **sur son net de drain**.
- `C110`/`C210` : établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec le mute en Phase F.
- Connectique déportée JST XH 2,5 mm, pince à sertir nécessaire. Câble 48 V de 0,5 à 2,5 mm².
- Repère de taille : l'EVM fait **160 × 120 mm**, entrées à gauche, puce au centre-gauche, LC et bulk à droite, sorties en bord droit.

## NEEDS_DATA ouverts

- ~~Dissipateur et boîtier~~ — **famille tranchée en E1.7** ; restent les trois arbitrages.
- Alimentation 48 V : volet tension clos par `REQ-PSU-1` ; restent ripple, courant continu garanti, démarrage.
- `RV1` mécanique, réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33.
- Fabricant de PCB non choisi. **Épaisseur de cuivre par couche non exposée par le MCP** : à rendre explicite dans Board Setup et à la commande.
- Références à finir de décoder, non inscrites au schéma : `EEU-FC1J152`, `SLPX472M080H3P3`, `MKP4F036804F00` + 4 caractères.

## Décisions actives portées

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; lecture hors MCP pour vérifier. **Les fichiers de configuration s'éditent directement.**
- **`set_design_rules` et `set_active_layer` sont PROSCRITS** : jetons invalides dans `(setup ...)`, fichier illisible par KiCad, aucun retour d'erreur. Les règles se posent dans le `.kicad_pro`, sous `board.design_settings.rules`.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.**
- **`kicad-cli.exe` fournit une preuve indépendante du MCP** : `sch erc`, `sch export netlist`, `fp export svg`, `pcb drc`. Chemin `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **La nomenclature du kit d'évaluation est la première source à ouvrir.** C'est elle qui a livré le dissipateur et sa fixation en E1.7.
- **Avant de créer une empreinte locale, épuiser la librairie standard.**
- **`create_footprint` dérive la sérigraphie de l'enveloppe des pastilles** ; corriger par **`set_footprint_graphics`**, qui existe.
- **Lire les cotes par extraction du PDF fabricant**, jamais par recherche web ni listing distributeur. **Toujours contrôler si un tracé est à l'échelle**, et à quelle figure appartient une cote. **Lire aussi les conditions de mesure** : c'est l'absence de ligne à vitesse d'air nulle qui a disqualifié le dissipateur de l'EVM.
- **Serveurs qui servent le PDF à `curl`** : ti.com/lit, **qats.com/DataSheet**, **info.boydcorp.com/hubfs**, **hifi2000.shop**, schurter.com/datasheet, vishay.com/docs, content.kemet.com, cde.com, coilcraft.com/pdfs, wima.de, tdk-electronics.tdk.com, Infineon, `wmsc.lcsc.com`. **Refusent** : **fischerelektronik.de**, **tme.eu**, Littelfuse, DigiKey, `www.lcsc.com`, `datasheet.lcsc.com`, nichicon.co.jp, rubycon.co.jp, industrial.panasonic.com, coilcraft.com hors `/pdfs`.
- **PDF sans couche texte** : rendre en PNG par `pymupdf` (`page.get_pixmap(dpi=200)`) puis lire le PNG. **Écrire l'extraction dans un fichier UTF-8 avant de l'afficher** : la console Windows casse sur les accents.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- Bulk maintenu à 15 400 µF ; `Q302` choisi en conséquence. TVS cantonnée aux transitoires rapides.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyses dans `reports/MCP_BUG-documenttype-routing-eeschema.md` et `reports/MCP_BUG-setup-tokens-kicad-pcb.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Écritures fichier persistées ; prouver par relecture.
- `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées, et **accepte `uuid` en plus de `reference`**.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : `lib_symbols` précède les instances ; itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb` (vide de connectivité), `.kicad_sym`, `sym-lib-table`, `fp-lib-table`
- `HifiAmp_TPA3255_Local.pretty/` : `Fuse_Schurter_UMT-H_5.3x16mm`, `L_Toroid_Vertical_L28.6mm_W12.3mm_P10.00mm_Coilcraft_PA6331`
- `docs/architecture.md` — sections « Rectification de `REQ-THERM-1` », « Ce que fait l'EVM », « E1.7 — la solution n'est pas un dissipateur, c'est le coffret », `REQ-THERM-3`
- PDF du scratchpad de session : `slou441.pdf` (guide EVM), `ats_ti1op.pdf`, `modushop_thermal.pdf`, `boyd_blh.pdf`

## NEXT ACTION

**Obtenir les trois arbitrages qui conditionnent le contour de carte**, puis enchaîner E1.2 :

1. **Modèle de coffret** — `02/300` (2U, 0,45 °C/W, `T_C` 57 °C) suffit thermiquement ; les modèles supérieurs achètent de la marge et de la hauteur, pas de la nécessité.
2. **Architecture mécanique** — carte à plat sur l'embase avec `U6` en bord de carte et barre de liaison courte et massive, ou carte montée verticalement contre le flanc. Ce choix fixe la forme du contour, pas seulement sa taille.
3. **Traitement de la masse** — châssis = `GND` assumé, intercalaire AlN 0,5 mm à 0,101 °C/W, ou isolation reportée sur la grande surface barre/flanc.

Dès ces trois points tranchés : importer les empreintes via `kicad-control`, tracer `Edge.Cuts`, **placer `U6` en premier** avec son chemin thermique, puis valider en relisant le `.kicad_pcb` au fichier — `kicad-cli pcb drc` ne devant plus remonter `invalid_outline`.
