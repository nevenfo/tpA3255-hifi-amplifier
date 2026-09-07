# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **99 violations, 254 non-connectés, `schematic_parity` = 0** — 96 de sérigraphie,
normales avant routage, plus 3 `lib_footprint_mismatch` admis (cf. décisions).

## Tâche actuelle

**E1.3 — placer filtres LC, sorties, alimentation et boucles de retour.** En cours, trois
tranches sur quatre faites. **72 des 124 empreintes sont placées**, 52 restent dans le bloc
d'import. Reste la tranche **« supervision et configuration »** : `U7` TPS3802 et son réseau
— `R32`/`R26`/`C67` sur `VSENSE`, `R6`/`C83`/`C84` sur `RESET_RC` — plus les quatre
résistances de configuration de `U6`.

## Dernière tâche validée

**E1.3, tranche « alimentation auxiliaire » = PASS.** Les 24 empreintes du buck `LM5010` et de
sa descendance sont posées en colonne dans `x` 158..186, `y` 134..194 — bande libre entre la
chaîne d'entrée 48 V et le haut de carte, adjacente à `U8`/`Q302` d'où sort `PVDD`, et qui
laisse `x` 100..155 à l'analogique de E1.4. Un défaut corrigé avant clôture : `C11` laissait
0,300 mm à la pastille `/PVDD` de `D3` pour 0,500 exigés par `PWR_48V` — demi-largeur de
pastille du `D_SMA` négligée ; `C11` repoussée de `x` 183 à 185.

Validation :

- DRC **99 violations, dont 96 de sérigraphie** ; **aucune `clearance`, aucun
  `courtyards_overlap`** ; `schematic_parity` = 0 ; 254 non-connectés inchangés.
- 124 empreintes, les 24 positions et les 48 placements antérieurs relus du fichier
  enregistré, intacts au micron.
- Baseline recalculée sur le commit précédent **dans le contexte de librairies du projet** —
  90 violations, 0 `lib_footprint_mismatch` — sans quoi la mesure aurait faussement compté
  5 `lib_footprint_issues` de librairie introuvable.

**Avant elle** : E1.3 tranches « entrée 48 V » (`1ae53af`) et « filtres de sortie »
(`ec83d2f`) ; E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS ; D1 CLOSE.

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
- **Contour arrêté à 200 × 150 mm**, `(100,100)`–`(300,250)`, contraint par le placement et non
  par le coffret ; resserrable après E1.5.
- **Trois `lib_footprint_mismatch` à la baseline, et il en viendra d'autres.** Tout déplacement
  IPC fait réécrire l'empreinte par KiCad, qui en retire le bloc `(units)` — **aucun UUID perdu,
  4543 avant comme après**, aucune géométrie ni aucun net changé. Pendant exact des
  `lib_symbol_mismatch` admis à l'ERC, à attendre **à chaque circuit intégré déplacé**.
  Corollaire : **un DRC de comparaison se lance dans le répertoire du projet**, sinon il compte
  de faux `lib_footprint_issues` et la baseline est fausse.
- **`REQ-THERM-3`** : nominal continu 2 × 100 W sur **8 Ω**, le 4 Ω en crête seulement. Coffret
  Modushop `03/300` 3U, carte à plat.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de
  configuration s'éditent directement**.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, contour 200 × 150, **72 placées**, 52 encore
  dans le bloc d'import autour de `x` 221..321 / `y` 272..299
- `HifiAmp_TPA3255.kicad_dru` — règle d'isolation intra-empreinte de E1.9
- `HifiAmp_TPA3255.kicad_sym` — symboles locaux `LM2940IMP_12_FIXED`, `TLV1117_33_FIXED`
- **`docs/kicad-operations.md` — comment piloter KiCad ici** : séquence IPC, pièges MCP,
  proscriptions, relecture hors MCP. À lire avant toute manipulation de la carte.
- `docs/architecture.md` — contraintes de placement actives, `NEEDS_DATA`, budgets thermiques
- **État KiCad** : fermé, aucun verrou `~*.lck`.

## NEXT ACTION

**E1.3, tranche « supervision et configuration » — clore E1.3.** Sortir du bloc d'import
11 empreintes. `U7` et son réseau vont dans la bande réservée sous `U3`, `x` 156..184 /
`y` 122..134 : `R32` (162, 132), `R26` (162, 129), `C67` (166, 130), `U7` (172, 130),
`R6` (178, 132), `C83` (178, 129), `C84` (178, 126). Les quatre résistances de configuration
de `U6` vont en colonne `x` = 297, hors des bandes interdites par la barre : `R303` (297, 162),
`R302` (297, 165), `R301` (297, 184), `R304` (297, 187) — pull-ups vers le haut, résistances de
programmation vers le bas, chacune du côté d'où sort sa broche. **Vérifier au DRC que `x` = 297
tient le dégagement au bord droit du contour à `x` = 300** ; replier sur `x` = 296 sinon.

Lancer KiCad par `explorer.exe`, ouvrir l'éditeur de PCB depuis le gestionnaire, déplacer en
IPC via `kicad-control` (cf. `docs/kicad-operations.md`). Valider par relecture du fichier —
positions exactes, 72 placements antérieurs intacts au micron, 124 empreintes — puis par
`kicad-cli pcb drc --format json` **lancé depuis le répertoire du projet** : `schematic_parity`
reste 0, 254 non-connectés, aucune `clearance`, aucun `courtyards_overlap`, et seulement le
`lib_footprint_mismatch` attendu pour `U7`.
