# PROGRESS

## Phase actuelle

Phase D. **GATE C2 = PASS**, revérifié après chaque écriture : ERC à 15 violations et 0 erreur, jeu identique à la baseline C2 (10 `endpoint_off_grid`, 5 `lib_symbol_mismatch`).

## Tâche actuelle

D1.13 — Trancher la fenêtre de tension où le TPA3255 travaille hors spécification. **Toutes les tâches de fond de D1 sont closes** ; restent D1.13, D1.14 et le reliquat cosmétique D1.8.

## Dernière tâche validée

**D1.3 = PASS, et la tâche se retourne : il n'y a pas de vias thermiques à concevoir sous le TPA3255.**

- `U6` portait l'empreinte **d'un autre boîtier** : taguée `Texas_DDW0044B`, elle posait une 45ᵉ pastille de 5,2 × 14 mm **sous** le composant. Le TPA3255 est en `DDV0044D`, dont la datasheet dit que *« the PowerPAD is located on the top side of the device for convenient thermal coupling to the heat sink »*, et dont le land pattern TI ne montre que **44 pastilles**. Le plan coté donne un pad exposé de **7,01 × 4,14 mm nominal**, pas 5,2 × 14. Corrigé en `Package_SO:HTSSOP-44_6.1x14mm_P0.635mm_TopEP4.14x7.01mm`, taguée `Texas_DDV0044D`.
- **Preuve chiffrée : `RθJC(bot)` est donné `n/a`** — TI ne caractérise même pas la voie par le dessous — quand `RθJA` tombe de **50,7 à 2,4 °C/W** avec un dissipateur sur le dessus, facteur 21, « only path for dissipation is to the heatsink ».
- **Le `NEEDS_DATA` annonçant un EP de 5,2 × 14 mm est infirmé, pas confirmé.** Le dissipateur devient le point dimensionnant unique du thermique du TPA3255.
- **Écart connu et voulu, à reporter en Phase E** : le symbole a 45 broches, la broche 45 étant `EP_45` câblée à `GND` — correct au sens de TI — alors que l'empreinte n'en a que 44. L'import PCB signalera une broche sans pastille : **c'est attendu**, la mise à la masse du PowerPAD passant par le dissipateur, hors PCB.

**Avant elle, D1.2, D1.10, D1.12, D1.9 et D1.11 = PASS.** Le bornier `J1` a été clos sur le dessin fabricant MaiXu : 300 V / 10 A en UL, 250 V / 15 A en IEC, perçage ø1,30 au pas 5,00 conforme à l'empreinte.

- **Inductances.** La BOM du `TPA3255EVM` donne `MA5172-AE` Coilcraft, et sa datasheet (document Coilcraft 943) catalogue dans la même famille **`PA6331-AE` : 15 µH, DCR 31 mΩ, `I_sat` 20 A, `I_rms` 9,8 A à 20 °C d'échauffement** — exactement le cahier des charges. **Retenue sur arbitrage utilisateur.** La réserve énergétique était fondée : la pièce réelle est un **tore traversant debout de ø28,6 × 12,3 mm**, là où le `HCI-1350` supposé mesurait 12,8 × 12,8 × 4,7 mm.
- **Le tracé de la datasheet n'est pas à l'échelle** : 2,836 pt/mm en vue de face contre 2,463 en profil, 15 % d'écart. Les étiquettes font foi, comme chez Schurter. Entraxe **10,0 ± 0,5 mm**, broches 0,96 à 1,07 mm.
- Aucune empreinte standard ne convenait : le `Bourns_5700` a la bonne longueur mais 11,43 mm d'entraxe, hors tolérance ; le `Pulse_D` a le bon pas mais un courtyard trop court de 1,9 mm. D'où la **deuxième empreinte locale justifiée du projet**, `L_Toroid_Vertical_L28.6mm_W12.3mm_P10.00mm_Coilcraft_PA6331`, créée par le MCP et relue au fichier.
- **Les graphiques imposés par `create_footprint` sont corrects cette fois** : courtyard à 0,25 mm autour de l'élément le plus extérieur — les pastilles, non le corps — et sérigraphie à 0,15 mm hors du corps sans recouvrir les pastilles. Ce sont les conventions KLC, meilleures que les cotes que le principal avait commandées. **La leçon de `CF_Film_Box` se précise : le générateur n'était pas en cause, les cotes qu'on lui donnait l'étaient.**
- **D1.12** : `fp-lib-table` déclarait la librairie locale par un chemin absolu Windows — un clone du dépôt ailleurs aurait cassé la résolution des deux empreintes locales sans avertissement. Remplacé par `${KIPRJMOD}/…`. `sym-lib-table` utilisait déjà cette forme, donc l'incohérence était isolée, et l'ERC prouve que KiCad résout bien la variable ici puisqu'il charge le symbole local `LM5069` déclaré de la même façon.

## Défauts ouverts

- **D1.13 — fenêtre de 53,5 à 56,4 V où une charge de 4 Ω est hors spécification sans que rien ne coupe.** TI plafonne `PVDD` à 53,5 V absolus sous 4 Ω, contre 56,5 V sous ≥ 6 Ω et seulement avec un seuil de surintensité réduit ; le projet vise 4 à 8 Ω et le `LM5069` coupe à 56,4 V. Le raisonnement en place — `Q302` isole le TPA3255 en surtension — ne couvre pas cette bande. Trois issues : abaisser `V_OVH` vers 52 V, restreindre la charge à ≥ 6 Ω, ou démontrer que l'alimentation retenue ne peut pas atteindre 53,5 V. **Dépend du `NEEDS_DATA` alimentation externe ; arbitrage utilisateur probable.**
- **D1.14 — les cinq `lib_symbol_mismatch` sont un tiers de la baseline et ne sont pas diagnostiqués.** Ils portent sur `LM5010ASD`, `LM2940IMP_12_FIXED`, `TPS3802K33`, `LM5069` et `TPA3255B` : la copie embarquée dans le schéma diverge de la librairie, et l'écart peut être anodin comme porter sur un brochage. Reliquats à traiter dans le même mouvement : le cache du schéma contient encore `TPA3255DDV`, `TPA3255DDV_TEST` et `TPA3255`, absents du `.kicad_sym` et inutilisés, et la librairie garde `LM2940IMP-12` alors que `U2` utilise `LM2940IMP_12_FIXED`.
- **D1.8, reliquat cosmétique.** Le volet film est clos ; ne restent que les graphiques de `Fuse_Schurter_UMT-H_5.3x16mm`, sans effet DRC ni fabrication. L'empreinte locale reste justifiée : le standard `Fuse_Schurter_UMT250` vise un corps 3 × 10,1 mm, pastilles à ± 4,25 contre ± 6,875 mm. **À revoir à la lumière de D1.10** : le générateur produit des graphiques corrects quand les cotes le sont, donc une recréation propre est peut-être plus simple qu'une correction.

## Blocage actif

Aucun.

## Contraintes portées en Phase E

- **Le TPA3255 se refroidit uniquement par le dessus.** Aucun via thermique sous le boîtier n'a d'objet, `RθJC(bot)` étant `n/a`. Prévoir le dégagement mécanique du dissipateur au-dessus de `U6`, et sa mise à la masse, qui porte la liaison `GND` du PowerPAD. **L'import PCB signalera la broche 45 sans pastille : c'est attendu, ce n'est pas un défaut à corriger.**
- **Inductances : quatre tores debout de ø28,6 mm, épaisseur 12,3 mm, soit environ 29 mm de hauteur au-dessus du PCB.** Pertes cuivre ≈ 0,8 W chacune à 5 A RMS, **3,1 W au total**, à ajouter au bilan thermique.
- **Films de sortie : la boîte passe de 7,2 × 3,5 à 18 × 8 mm sur 15 de haut**, courtyard de 18,5 × 8,5 mm, quatre fois.
- **Bulk : `C312` à `C315` gagnent 2 mm de courtyard chacun** (16,16 → 18,16 mm), hauteur nominale 35 mm. `C316`/`C317` : ø35 mais **30 mm de haut seulement**, et non les 50 mm que suggère la description de l'empreinte KiCad.
- `C325` culmine à **18 mm**.
- **`R306` est un shunt à deux bornes, pas Kelvin.** Les liaisons vers `VIN` et `SENSE` doivent partir des **bords intérieurs** des pastilles, le courant de puissance entrant par les bords extérieurs. Repli : `WSK25122L000FEA`, quatre bornes, même boîtier 2512.
- `Q302` doit être monté sur radiateur : sa SOA suppose le boîtier à 75 °C.
- `Q301` : courant continu plafonné à 6,9 A avec la surface de cuivre de référence de sa datasheet, 6 cm² en 70 µm.
- Rail d'entrée : les calibres UMT-H supposent des pistes de 7,5 mm en cuivre 140 µm. Déclassement sinon.
- `C110`/`C210` imposent un établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec la temporisation de mute en Phase F.
- Connectique déportée en JST XH : deuxième famille à approvisionner à côté des MaiXu MX126-5.0, et pince à sertir nécessaire.
- **Le câble d'alimentation 48 V doit rester entre 0,5 et 2,5 mm²** : c'est la plage que le bornier `J1` accepte.

## NEEDS_DATA ouverts

`RV1` (mécanique du potentiomètre), alimentation externe 48 V, **dissipateur et thermique — désormais le point dimensionnant unique du refroidissement du TPA3255**, réponse/EMI du filtre LC, common-mode du TPA3255, broche MR du TPS3802K33.

**Levé : l'EP du TPA3255.** Il ne fait pas 5,2 × 14 mm et n'est pas sous le boîtier : 7,01 × 4,14 mm nominal, sur la face supérieure.

**Levé : l'inductance de sortie.** `PA6331-AE` est figée sur arbitrage utilisateur ; reste à confirmer en H2 qu'elle est toujours approvisionnable.

Candidats d'ancrage **non inscrits au schéma** tant que la clé fabricant n'est pas décodée jusqu'au bout : `EEU-FC1J152` (Panasonic, ø18) pour `C312` à `C315` ; `SLPX472M080H3P3` (CDE) pour `C316`/`C317` ; `MKP4F036804F00` plus quatre caractères de tolérance et conditionnement pour `C321` à `C324`.

Assumés : stabilité de la boucle de limitation de puissance du LM5069 face aux 540 nC de grille de `Q302` ; `V_C` de la `SMDJ58CA` sous `I_PP` ; pas de 7,5 mm du boîtier ø18, inchangé par la correction mais non relu chez Panasonic.

## Décisions actives

- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP. Lecture hors MCP pour vérifier seulement. **Les tables de librairies, elles, s'éditent directement** : ce sont des fichiers de configuration, sans connectivité à corrompre.
- **Les rapports d'agents sont systématiquement vérifiés par le principal avant tout verdict.**
- **`kicad-cli.exe` est utilisable directement** (`sch erc`, `sch export netlist`) et fournit une preuve indépendante du MCP, sans GUI. Chemin : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`.
- **La nomenclature du kit d'évaluation qui sert d'ancre au design est la première source à ouvrir** pour tout poste dont la référence manque : elle donne le boîtier réel et le diélectrique, et sert de point d'entrée pour décoder la clé du fabricant. Elle a résolu D1.9, D1.10 et D1.11 en une seule lecture.
- **Avant de créer une empreinte locale, épuiser la librairie standard.** Sur les trois locales créées, deux étaient nécessaires.
- **`create_footprint` produit des graphiques conformes aux conventions KLC quand les cotes fournies sont justes.** L'échec de `CF_Film_Box` venait des cotes, pas du générateur. Lui donner les cotes du corps et le laisser calculer les marges.
- **Lire les cotes par extraction du PDF fabricant, pas par recherche web ni par listing distributeur.** `pymupdf` installé, `pdftotext` dans `/mingw64/bin`. **Toujours contrôler si un tracé est à l'échelle** : ni celui de Schurter ni celui de Coilcraft ne le sont ; se mesure en comparant, sur les vecteurs du PDF, l'échelle déduite de deux cotes différentes. **Et toujours vérifier à quelle figure appartient une cote** : le « ø2 ± 0,1 » des snap-in est une cote de perçage, pas de broche.
- **Une référence ne se valide pas sur son aspect, mais en la décodant champ par champ contre la clé du fabricant, puis en recoupant la boîte obtenue avec le tableau de la valeur visée.**
- **Serveurs qui servent le PDF à `curl`** : ti.com/lit, vishay.com/docs, content.kemet.com, cde.com, coilcraft.com/pdfs, wima.de, tdk-electronics.tdk.com, Infineon, **`wmsc.lcsc.com`**. **Serveurs qui refusent ou ne répondent pas** : Littelfuse, DigiKey, `www.lcsc.com` et `datasheet.lcsc.com`, nichicon.co.jp, rubycon.co.jp, industrial.panasonic.com (timeout complet), coilcraft.com hors `/pdfs` (403).
- **Un PDF sans couche texte se lit quand même** : `pdftoppm` n'est pas installé, donc l'outil de lecture d'image du harness ne prend pas le PDF directement ; le rendre en PNG par `pymupdf` (`page.get_pixmap(dpi=200)`) puis lire le PNG. C'est ainsi qu'a été lu le plan MaiXu.
- Aucune mutation géométrique : la connectivité repose sur la coïncidence label/ancre.
- TVS cantonnée aux transitoires rapides ; la protection en surtension est active, par `LM5069`.
- Bulk maintenu à 15 400 µF sur arbitrage utilisateur ; `Q302` choisi en conséquence.
- Connectique déportée : JST XH 2,5 mm, vertical par défaut, sur arbitrage utilisateur.
- Asymétrie de nommage assumée : `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite. À trancher avant H2.
- Instantanés PDF automatiques du MCP (`*_pre_delete_*.pdf`) exclus par `.gitignore`.
- Réserve consignée non tranchée : à 100 V un P-canal reste environ trois fois moins bon qu'un N-canal ; l'alternative serait un contrôleur de diode idéale (`LM74700`, `LM5050`) pilotant un N-canal. Non retenue, la topologie P-MOS est tranchée et 0,70 W est acceptable.

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. Ne pas restaurer `v1.1.2`. Analyse dans `reports/MCP_BUG-documenttype-routing-eeschema.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`. Les écritures sont fichier et persistées ; prouver par relecture.
- Attributs `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que Reference/Value/Footprint/Datasheet plus des propriétés personnalisées.
- **`edit_schematic_component` accepte `uuid` en plus de `reference`** : seul moyen d'adresser un symbole quand plusieurs partagent le même repère.
- **`create_footprint` impose ses propres graphiques** et aucun outil ne les édite après coup — mais ces graphiques sont corrects, voir les décisions actives.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- `get_schematic_component` / `get_component_nets` exigent un chemin absolu et mésattribuent les broches `power_in`.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà d'environ 72 000 caractères.
- **Piège de relecture hors MCP** : le `.kicad_sch` contient une section `lib_symbols` avant les instances. Itérer sur les blocs `(symbol` de premier niveau à parenthèses équilibrées.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pro`, `.kicad_sch`, `.kicad_pcb` (**vide, aucune empreinte placée** : les changements d'empreinte n'imposent aucune resynchronisation), `.kicad_sym`, `sym-lib-table`, `fp-lib-table`
- `HifiAmp_TPA3255_Local.pretty/` : `Fuse_Schurter_UMT-H_5.3x16mm` et `L_Toroid_Vertical_L28.6mm_W12.3mm_P10.00mm_Coilcraft_PA6331`
- `docs/architecture.md`, `docs/power-block.md`, `docs/protection-48v.md`
- Librairies KiCad : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/share/kicad/footprints`
- PDF déjà téléchargés, dans le scratchpad de session : `slou441.pdf` (BOM EVM TPA3255, p. 14-15), `ma5172.pdf` (Coilcraft doc 943), `e_WIMA_MKP_4.pdf`, `SLP.pdf` (CDE), `058059pll-si.pdf` (Vishay), `KEM_A4082_ALC80.pdf`.

## NEXT ACTION

D1.14 — diagnostiquer les cinq `lib_symbol_mismatch`, qui sont un tiers de la baseline C2 et n'ont jamais été instruits. Pour chacun des cinq symboles, comparer la copie embarquée dans la section `lib_symbols` du `.kicad_sch` à celle du `.kicad_sym`, **broche à broche d'abord** — numéro, nom, type électrique — puis champs de propriétés, et classer l'écart : anodin ou portant sur le brochage. Un écart de brochage remettrait en cause des vérifications déjà faites et doit être signalé immédiatement. Ne rien réécrire avant d'avoir le diagnostic des cinq. Traiter ensuite les reliquats inutilisés, en montrant d'abord qu'aucun composant n'y renvoie. **D1.13 est laissée en attente : elle dépend du `NEEDS_DATA` alimentation externe et appelle un arbitrage utilisateur.**
