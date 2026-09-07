# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **101 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage et nettoyage. Plus aucun `lib_footprint_mismatch`.

## Tâche actuelle

**E1.4 — analogique faible bruit, volume et contrôles.** Canal gauche posé, **103 des 124
empreintes placées**. Reste le **canal droit et la connectique** : `U5` et la série 200,
`J3`, `J4`, plus `C81`/`C82` sur le rail `+12V-OA`.

## Dernière tâche validée

**E1.4, tranche « canal gauche » = PASS.** `U4` en (130, 145) et ses 19 satellites, dans la
bande `x` 100..155 réservée au bord opposé de `U6`. Le brochage relevé commande la disposition :
l'ampli A occupe le flanc gauche du SOIC-8, l'ampli B le flanc droit, et B ré-inverse la sortie
de A pour produire la phase opposée — chaque réseau de contre-réaction reste donc du côté de
son ampli, seul `R103` traversant puisqu'il relie la sortie de A à l'entrée de B.

Validation :

- DRC **101 violations, toutes de sérigraphie** ; aucune `clearance`, aucun
  `courtyards_overlap`, `lib_footprint_mismatch` toujours **0**.
- `pad_prop_heatsink` toujours présent une fois : la régression de E1.10 ne s'est pas refaite.
- `schematic_parity` = 0 ; 254 non-connectés ; 124 empreintes ; les 83 placements antérieurs
  intacts au micron.

**Avant elle** : E1.10 (`73abdab`) ; E1.3 close en quatre tranches ; E1.9, E1.2, E1.8, E1.7,
E1.6, E1.1 = PASS.

## Décisions actives

Les décisions de placement figées — rotation de `U6`, canaux en rangées, arête arrière à
`y` = 250, rotation de `U1`, colonne `x` = 297 pour `R301`–`R304`, bande `x` 100..155 réservée
à l'analogique — sont dans `docs/architecture.md`, section « Décisions de placement figées en
E1 ». Le pilotage KiCad est dans `docs/kicad-operations.md`. Restent ici celles qui gouvernent
encore la prochaine action :

- **La barre de liaison est dressée** : 10 mm dans le plan de la carte, 60 mm de hauteur,
  600 mm² de section. **Exigence non négociable : pied élargi d'au moins 1200 mm² côté flanc**,
  sinon la marge thermique tombe à 0,09 °C/W. Deux M3 en `(285, 163)` et `(285, 187)`, entraxe
  24 mm. **Zones interdites aux composants** : `x` ∈ [280, 291], `y` ∈ [160, 190], et
  `x` ∈ [280, 300], `y` ∈ [169, 181].
- **Contour arrêté à 200 × 150 mm**, `(100,100)`–`(300,250)` ; resserrable après E1.5.
- **Un déplacement IPC peut perdre une propriété de pastille, et le DRC le dit.** `U1` a perdu
  `(property pad_prop_heatsink)` en étant déplacé — seule occurrence de la carte — ce qu'a
  révélé le `lib_footprint_mismatch` correspondant. La réparation est « Mise à Jour des
  Empreintes à partir des Librairies », options texte décochées. **À vérifier après tout
  déplacement de circuit intégré porteur d'un pad exposé.** À l'inverse, la perte du bloc
  `(units)` est sans effet : `U7` le perd sans produire de mismatch.
- **Un DRC de comparaison se lance dans le répertoire du projet**, sinon il compte de faux
  `lib_footprint_issues` de librairie introuvable et la baseline est fausse.
- **`REQ-THERM-3`** : nominal continu 2 × 100 W sur **8 Ω**, le 4 Ω en crête seulement. Coffret
  Modushop `03/300` 3U, carte à plat.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de
  configuration s'éditent directement**.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, contour 200 × 150, **83 placées**, 41 encore
  dans le bloc d'import autour de `x` 221..321 / `y` 272..299
- `HifiAmp_TPA3255.kicad_dru` — règle d'isolation intra-empreinte de E1.9
- `HifiAmp_TPA3255.kicad_sym` — symboles locaux `LM2940IMP_12_FIXED`, `TLV1117_33_FIXED`
- **`docs/kicad-operations.md` — comment piloter KiCad ici** : séquence IPC, pièges MCP,
  proscriptions, relecture hors MCP. À lire avant toute manipulation de la carte.
- `docs/architecture.md` — contraintes de placement actives, `NEEDS_DATA`, budgets thermiques
- **État KiCad** : fermé, aucun verrou `~*.lck`.

## NEXT ACTION

**E1.4, tranche « canal droit et connectique » — clore E1.4.** Poser les 21 dernières
empreintes en **miroir du canal gauche, décalées de +40 mm en `y`**, pour que les deux voies
soient géométriquement comparables : `U5` (130, 185), `R201` (122, 184.5), `R202` (122, 181.5),
`C202` (122, 179), `C210` (122, 188), `R203` (130, 178.5), `C208` (136, 182), `C209` (140, 182),
`R204` (136, 185), `C203` (140, 185), `C204` (126, 192), `C205` (132, 192), `R207` (126, 196),
`R208` (132, 196), `C206` (126, 199.5), `C207` (132, 199.5), `C201` (112, 185), `J3` (104, 185).
Puis la connectique et le rail : `J4` (108, 112) vers la façade — six pads au pas de 2,50 mm à
partir de son pad 1, donc il s'étend jusqu'à `x` ≈ 120,5 — et `C81` (150, 139), `C82` (154, 139)
en aval de `L6`, déjà posé en (162, 139).

`C210` reste collé à `U5` pour la même raison que `C110` à `U4`. Ne pas déborder au-delà de
`x` = 156 : le bloc d'alimentation auxiliaire occupe `x` 156..186 sur `y` 122..194.

Valider par relecture du fichier — positions exactes, 103 placements antérieurs intacts au
micron, 124 empreintes, `pad_prop_heatsink` toujours à 1 — puis par `kicad-cli pcb drc
--format json` **lancé depuis le répertoire du projet** : `schematic_parity` reste 0, 254
non-connectés, aucune `clearance`, aucun `courtyards_overlap`, `lib_footprint_mismatch`
toujours 0.
