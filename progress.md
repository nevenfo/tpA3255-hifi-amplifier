# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS` inchangée, revérifiée à l'ouverture de session : `kicad-cli sch erc` remonte 15 violations, 0 erreur, jeu identique à la baseline C2.

## Tâche actuelle

**E1.2 — placer la puissance Class-D. Débloquée, étape 1 franchie.** Les trois arbitrages qui la conditionnaient sont rendus et committés. La carte est peuplée et connectée ; reste le placement proprement dit.

## Dernière tâche validée

**E1.2 étape 1 = PASS — contour et import.** `Edge.Cuts` porte le rectangle **200 × 150 mm**, coins `(100,100)`–`(300,250)`. Les **122 empreintes** du schéma sont dans le PCB avec leur connectivité.

Validation, tenue au fichier par le principal :

- 122 empreintes, **122 références distinctes, aucun doublon**. Les deux `U6` parasites qui traînaient dans l'arbre de travail ont été supprimées par la synchronisation, une seule `U6` subsiste.
- **74 nets**, 327 pastilles sur 332 portant une équipotentielle.
- `RV1` absente, **attendu** : c'est le seul symbole sans empreinte assignée.
- `kicad-cli pcb drc` : **`schematic_parity` = 0**, la carte est le miroir exact du schéma. **`invalid_outline` a disparu**, le contour est fermé.
- Les 391 violations et 253 non-connectés restants sont l'état normal d'avant placement : empreintes empilées au point d'import, aucune piste.

**Avant elle** : E1.7 = PASS, E1.6 = PASS, E1.1 = PASS, D1 CLOSE.

## Décisions actives

- **Les trois arbitrages de E1.7 sont rendus**, dans `docs/architecture.md` § « Arbitrages rendus, et contour qui en découle » : coffret **Modushop `03/300` 3U** (flancs 300 × 120 × 40, 0,41 °C/W chacun) ; **carte à plat sur l'embase**, `U6` en bord de carte relié au flanc par une **barre aluminium de 30 à 40 mm, section 10 × 60** ; **isolation reportée à la jonction barre/flanc**, la masse signal restant flottante par rapport au châssis. Chaîne totale 1,711 à 1,932 °C/W pour 2,232 de budget, `T_C` entre 63 et 68 °C.
- **Contour arrêté à 200 × 150 mm**, contraint par le placement et non par le coffret ; resserrable après E1.5.
- **`REQ-THERM-3`** : nominal continu 2 × 100 W sur **8 Ω**, le 4 Ω en crête seulement.
- **La marge thermique n'est plus confortable** : c'est la barre qui la consomme. **La barre doit être aussi courte que le placement le permet**, et le pad d'isolation à `k` ≥ 3 W/m·K.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de configuration s'éditent directement**.
- **`set_design_rules` et `set_active_layer` sont PROSCRITS** : jetons invalides dans `(setup ...)`, fichier illisible par KiCad, aucun retour d'erreur. Les règles vivent dans le `.kicad_pro`, sous `board.design_settings.rules`.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.** Appliqué deux fois cette session.
- **`kicad-cli.exe` fournit une preuve indépendante du MCP** — `sch erc`, `pcb drc --format json`, `sch export netlist`, `fp export svg`. Chemin `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **Lire les cotes par extraction du PDF fabricant**, jamais par listing distributeur ; contrôler l'échelle du tracé et **les conditions de mesure**.
- **PDF sans couche texte** : rendre en PNG par `pymupdf` (`page.get_pixmap(dpi=200)`). **Écrire l'extraction dans un fichier UTF-8 avant affichage** : la console Windows casse sur les accents.
- Bulk maintenu à 15 400 µF. Asymétrie de nommage assumée `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite, à trancher avant H2.

## Blocage actif

Aucun.

## Contraintes de placement pour E1.2

- **`U6` se refroidit uniquement par le dessus**, `RθJC(bot)` = `n/a`, aucun via thermique sous le boîtier. La broche 45 du symbole n'a **pas** de pastille dans `HTSSOP-44_…_TopEP` : l'erreur du rapport d'import à ce sujet est **attendue et correcte**, ce n'est pas un défaut à corriger.
- **L'emprise de la barre de liaison est une zone d'interdiction de hauteur**, pas un dégagement : sa face inférieure repose sur le dessus de `U6`, à ≈ 1,2 mm du PCB. Seuls des CMS plats passent dessous.
- **Orientation retenue dans le coffret** : `U6` contre un flanc, donc en **bord latéral** ; analogique bas niveau au bord opposé ; RCA, sorties haut-parleur et entrée 48 V sur l'arête arrière ; `J4` vers la façade.
- **Hauteurs** : tores `L301`–`L304` debout ø28,6 × 29 mm, 3,1 W de pertes cuivre au total ; `C321`–`C324` films 18 × 8 × 15 mm ; `C312`–`C315` ø18 × 35 mm ; `C316`/`C317` ø35 × **30 mm** ; `C325` 18 mm.
- **`R306` est un shunt à deux bornes, pas Kelvin** : `VIN`/`SENSE` depuis les bords **intérieurs** des pastilles. Repli `WSK25122L000FEA`.
- `Q302` en TO-264 sur radiateur, SOA supposant le boîtier à 75 °C. `Q301` en TO-220 : 6,9 A avec 6 cm² de cuivre 70 µm **sur son net de drain**.
- `C110`/`C210` : établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec le mute en Phase F.
- Repère d'implantation de l'EVM, 160 × 120 mm : entrées à gauche, puce au centre-gauche, LC et bulk à droite, sorties en bord droit.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyses dans `reports/MCP_BUG-documenttype-routing-eeschema.md` et `reports/MCP_BUG-setup-tokens-kicad-pcb.md`.

- **Aucun des 203 outils du MCP ne sait synchroniser schéma → PCB** — ni import de netlist, ni « update PCB from schematic ». `kicad-cli` ne l'expose pas non plus. **Seule voie : l'action Outils → « Mise à jour du PCB à partir du Schéma » de l'éditeur de PCB.**
- **Les outils MCP dépendants de l'IPC exigent que l'éditeur concerné ait le fichier ouvert**, pas seulement que KiCad tourne. Gestionnaire seul ⇒ `KiCad does not handle kiapi.common.commands.GetOpenDocuments for this document type`. `kicad_common.json` porte déjà `api.enable_server = true`.
- **L'éditeur de PCB doit être ouvert depuis le gestionnaire de projet.** Lancé en autonome sur le `.kicad_pcb`, KiCad refuse la synchronisation au schéma.
- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Écritures fichier persistées ; prouver par relecture.
- `place_component` fonctionne en mode fichier mais **n'a aucun champ net** ; `add_net` ne fait que déclarer un nom.
- `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées, et accepte `uuid` en plus de `reference`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : `lib_symbols` précède les instances ; itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées. En KiCad 10 les pastilles portent `(net "NOM")` **sans identifiant numérique**, et il n'y a pas de table de nets en fin de fichier.

## NEEDS_DATA ouverts

- Alimentation 48 V : volet tension clos par `REQ-PSU-1` ; restent ripple, courant continu garanti, démarrage.
- `RV1` mécanique — seul symbole sans empreinte ; réponse/EMI du filtre LC ; common-mode du TPA3255 ; broche MR du TPS3802K33.
- Plan de perçage de l'embase `01/05` non publié : trous de fixation à ajouter plus tard, sans effet sur le placement.
- Fabricant de PCB non choisi. **Épaisseur de cuivre par couche non exposée par le MCP** et pas de bloc `stackup` explicite dans le `.kicad_pcb` : à rendre explicite dans Board Setup et à la commande.
- `EEU-FC1J152` (`C312`–`C315`) reste non décodée à la source — `industrial.panasonic.com` refuse `curl`. Les deux autres références sont levées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — **122 empreintes connectées, contour 200 × 150, rien de placé ni routé**
- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_sym`, `sym-lib-table`, `fp-lib-table`
- `HifiAmp_TPA3255_Local.pretty/` : `Fuse_Schurter_UMT-H_5.3x16mm`, `L_Toroid_Vertical_L28.6mm_W12.3mm_P10.00mm_Coilcraft_PA6331`
- `docs/architecture.md` — sections « Arbitrages rendus, et contour qui en découle », « E1.7 — la solution n'est pas un dissipateur, c'est le coffret », « Rectification de `REQ-THERM-1` », `REQ-THERM-3`
- **KiCad doit rester ouvert** (gestionnaire + éditeur de PCB) pour toute opération MCP sur la carte.

## NEXT ACTION

**E1.2 étape 2 — placer le bloc de puissance Class-D**, dans cet ordre : `U6` en premier au bord latéral droit de la carte, avec sa zone d'interdiction de hauteur pour la barre de liaison ; puis bootstrap `C310`/`C311`, découplages `C301`–`C309`, et le bulk `C316`/`C317` + `C312`–`C315` au plus près des broches `PVDD`.

Valider en relisant le `.kicad_pcb` au fichier — coordonnées effectives de `U6`, aucun composant plus haut que 1,2 mm sous l'emprise de la barre — puis par `kicad-cli pcb drc --format json`, dont le nombre de violations `clearance` doit décroître à mesure que les empreintes quittent la pile d'import, `schematic_parity` restant à 0.
