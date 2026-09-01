# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS` inchangée : ERC à 15 violations et 0 erreur, jeu identique à la baseline C2 (10 `endpoint_off_grid`, 5 `lib_symbol_mismatch`, tous instruits et cosmétiques). Aucun fichier KiCad de connectivité n'a été touché depuis.

## Tâche actuelle

E1.1 — créer le PCB 4 couches et documenter stack-up, règles et classes de nets. Non commencée. Le `.kicad_pcb` est **vide, aucune empreinte placée**.

## Dernière tâche validée

**D1 est CLOSE. Toutes les empreintes sont attribuées et revues contre leurs sources fabricant.**

- **D1.13 = PASS**, sur arbitrage utilisateur. La fenêtre 53,5–56,4 V est fermée **par une spécification d'alimentation, pas par un composant**. Exigence `REQ-PSU-1` dans `docs/architecture.md` : sortie de l'alimentation 48 V ≤ 53,5 V en toutes conditions, 3,1 V de marge pour une régulée à ± 5 %. Rien n'est modifié : `V_OVH` reste 56,4 V, `OC_ADJ` reste 22 kΩ, la charge reste 4–8 Ω. Deux rectifications de fond au passage : **53,5 V est une borne de conditions recommandées et non un maximum absolu**, lequel vaut **69 V** et non les 65 V portés depuis B2.4 ; et **abaisser `V_OVH` est arithmétiquement impossible**, la fenêtre à couvrir valant ± 3 % contre ± 6 % de dispersion spécifiée du seuil. Le `LM5069` ne peut être qu'une protection de **défaut** d'alimentation.
- **D1.8 = PASS.** La note « aucun outil MCP n'édite les graphiques après coup » était **fausse** : `set_footprint_graphics` existe. Repère de broche 1 supprimé, sérigraphie ramenée autour du corps. Vérifié au fichier par le principal : dégagement de **0,240 mm** contre 0,20 exigés, rien hors courtyard, cuivre/pâte/masque/`descr` intacts, `F301` résout toujours. `kicad-cli fp export svg` trace les deux empreintes locales sans avertissement.

## Décision prise avant E1.1

**Cuivre standard 35 µm sur les quatre couches.** Les « 7,5 mm en 140 µm » du fusible sont une condition de mesure IEC 60127, pas une exigence : IPC-2221 ne demande que 2,47 mm à 35 µm pour les 4,6 A nominaux, et la coordination du fusible tolère un déclassement jusqu'à 7,4 A, soit 41 %, avant que la crête musicale de 9,3 A ne devienne critique. **Le cuivre épais serait de surcroît nuisible** : 140 µm ne tient pas les intervalles de 0,235 mm du HTSSOP-44 au pas 0,635 mm de `U6`. Rail `PVDD` à tracer à 7,5 mm là où le placement le permet, jamais moins de 2,5 mm. Détail : `docs/architecture.md`, section « Épaisseur de cuivre ».

## Blocage actif

Aucun.

## Contraintes portées en Phase E

- **`U6` se refroidit uniquement par le dessus**, `RθJC(bot)` = `n/a`. Aucun via thermique sous le boîtier. Prévoir le dégagement mécanique du dissipateur et sa mise à la masse, qui porte la liaison `GND` du PowerPAD. **L'import PCB signalera la broche 45 sans pastille : c'est attendu, ce n'est pas un défaut.**
- **Hauteurs** : inductances, quatre tores debout ø28,6 mm sur ≈ 29 mm de haut, **3,1 W de pertes cuivre au total** ; films de sortie 18 × 8 mm sur 15 de haut, courtyard 18,5 × 8,5 ; `C312`–`C315` ø18 sur 35 mm ; `C316`/`C317` ø35 sur **30 mm seulement** ; `C325` à 18 mm.
- **`R306` est un shunt à deux bornes, pas Kelvin.** Liaisons `VIN` et `SENSE` depuis les **bords intérieurs** des pastilles, courant de puissance par les bords extérieurs. Repli : `WSK25122L000FEA`, quatre bornes, même 2512.
- `Q302` sur radiateur : sa SOA suppose le boîtier à 75 °C. `Q301` : 6,9 A continus avec 6 cm² de cuivre en 70 µm **sur son net de drain**, pas une surface libre.
- `C110`/`C210` imposent un établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec le mute en Phase F.
- Connectique déportée JST XH 2,5 mm : deuxième famille à approvisionner, pince à sertir nécessaire. Câble 48 V entre **0,5 et 2,5 mm²**, plage du bornier `J1`.

## NEEDS_DATA ouverts

- **Dissipateur, interface et pression thermiques, boîtier, ventilation, ambiante** — point dimensionnant unique du refroidissement de `U6`, et préalable au contour de carte.
- Alimentation 48 V : **volet tension clos par `REQ-PSU-1`** ; restent ripple, courant continu garanti, comportement au démarrage.
- `RV1` (mécanique du potentiomètre), réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33.
- Fabricant de PCB non choisi. Sans effet sur la décision cuivre.
- Références à finir de décoder, **non inscrites au schéma** : `EEU-FC1J152` pour `C312`–`C315`, `SLPX472M080H3P3` pour `C316`/`C317`, `MKP4F036804F00` + 4 caractères pour `C321`–`C324`.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; lecture hors MCP pour vérifier seulement. **Les tables de librairies s'éditent directement**, ce sont des fichiers de configuration.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.**
- **`kicad-cli.exe` est utilisable directement** et fournit une preuve indépendante du MCP, sans GUI : `sch erc`, `sch export netlist`, `fp export svg`. Chemin : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **La nomenclature du kit d'évaluation est la première source à ouvrir** pour tout poste dont la référence manque.
- **Avant de créer une empreinte locale, épuiser la librairie standard.** Deux des trois locales créées étaient nécessaires.
- **`create_footprint` dérive la sérigraphie de l'enveloppe des pastilles, pas du corps.** Donner les cotes justes du corps corrige le `F.Fab` mais **pas** la sérigraphie dès que les pastilles débordent le corps ; reprendre alors par **`set_footprint_graphics`**, qui existe.
- **Lire les cotes par extraction du PDF fabricant**, jamais par recherche web ni listing distributeur. **Toujours contrôler si un tracé est à l'échelle** — ni Schurter ni Coilcraft ne le sont — et **à quelle figure appartient une cote**.
- **Une référence se valide en la décodant champ par champ contre la clé du fabricant**, puis en recoupant la boîte obtenue avec le tableau de la valeur visée.
- **Serveurs qui servent le PDF à `curl`** : ti.com/lit, **schurter.com/datasheet**, vishay.com/docs, content.kemet.com, cde.com, coilcraft.com/pdfs, wima.de, tdk-electronics.tdk.com, Infineon, `wmsc.lcsc.com`. **Refusent** : Littelfuse, DigiKey, `www.lcsc.com`, `datasheet.lcsc.com`, nichicon.co.jp, rubycon.co.jp, industrial.panasonic.com, coilcraft.com hors `/pdfs`.
- **PDF sans couche texte** : `pdftoppm` absent ; rendre en PNG par `pymupdf` (`page.get_pixmap(dpi=200)`) puis lire le PNG.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- Bulk maintenu à 15 400 µF sur arbitrage utilisateur ; `Q302` choisi en conséquence. TVS cantonnée aux transitoires rapides.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.
- Instantanés PDF automatiques du MCP (`*_pre_delete_*.pdf`) exclus par `.gitignore`.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Les écritures sont fichier et persistées ; prouver par relecture.
- Attributs `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées, et **accepte `uuid` en plus de `reference`** — seul moyen d'adresser un symbole quand plusieurs partagent le même repère.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb` (vide), `.kicad_sym`, `sym-lib-table`, `fp-lib-table` (chemins en `${KIPRJMOD}`)
- `HifiAmp_TPA3255_Local.pretty/` : `Fuse_Schurter_UMT-H_5.3x16mm` et `L_Toroid_Vertical_L28.6mm_W12.3mm_P10.00mm_Coilcraft_PA6331`
- `docs/architecture.md` (`REQ-PSU-1`, stack-up, épaisseur de cuivre, NEEDS_DATA), `docs/power-block.md`, `docs/protection-48v.md`
- PDF du scratchpad de session : `tpa3255.pdf` (`SLASEA8A`), `umth.pdf` (Schurter UMT-H).

## NEXT ACTION

E1.1 — créer le PCB 4 couches via `kicad-control`. Poser le stack-up L1 signaux/puissance, L2 plan `GND` continu, L3 distribution `PVDD`/12 V/3,3 V, L4 signaux et plans locaux, en **35 µm sur les quatre couches**. Définir ensuite les classes de nets et leurs largeurs : `PVDD` et retour de puissance à 7,5 mm cible et 2,5 mm plancher, sorties haut-parleur sur le même calibre, 12 V et 3,3 V intermédiaires, analogique et logique au défaut. **Ne pas tracer le contour de carte** : il dépend du dissipateur et du boîtier, qui restent en `NEEDS_DATA`. Valider en relisant le `.kicad_pcb` au fichier, hors MCP, et en confirmant que `kicad-cli` le relit sans erreur.
