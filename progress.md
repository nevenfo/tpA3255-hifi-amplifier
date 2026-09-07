# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`. Baseline ERC : 16 violations, 0 erreur
(10 `endpoint_off_grid`, 6 `lib_symbol_mismatch` — prix des symboles locaux). DRC courant :
**90 violations, 254 non-connectés, `schematic_parity` = 0**. Les 90 sont **toutes de la
sérigraphie** — normales avant routage et nettoyage de sérigraphie.

## Tâche actuelle

**E1.3 — placer filtres LC, sorties, alimentation et boucles de retour.** En cours, deux
tranches sur trois faites. **48 des 124 empreintes sont placées**, 76 restent dans le bloc
d'import. Reste à placer : **l'alimentation auxiliaire** (buck `U1` LM5010 avec sa boucle de
retour `R39`/`R40`, LDO `U2`/`U3`, supervision `U7`) et les **quatre résistances de
configuration de `U6`** — `R301` `FREQ_ADJ`, `R302` `FAULT`, `R303` `CLIP_OTW`, `R304`
`OC_ADJ`.

## Dernière tâche validée

**E1.3, tranche « entrée 48 V » = PASS** (`1ae53af`). La chaîne `J1` → `F301` → `D301` →
`Q301` → `R306` → `Q302` est déroulée du bas-gauche vers le haut-droite, dans le sens où le
courant circule ; `PVDD` quitte le MOSFET de hot-swap à 30 mm du bulk au lieu de 47. La TVS
est **après** le fusible, de sorte qu'un clamp en court-circuit fasse encore sauter `F301`.
Le réseau haute impédance du `LM5069` est groupé contre `U8` plutôt qu'étiré le long du
chemin de puissance.

Validation :

- DRC inchangé à **90 violations**, toutes de sérigraphie, **aucune `clearance`**, aucun type
  nouveau ; `schematic_parity` = 0 ; 254 non-connectés inchangés.
- Les 48 positions relues du fichier enregistré concordent **au micron** ; 124 empreintes.

**Avant elle** : E1.3 tranche « filtres de sortie » (`ec83d2f`, DRC 96 → 90) ; E1.9 = PASS
(42 violations `clearance` éteintes par `HifiAmp_TPA3255.kicad_dru`) ; E1.2, E1.8, E1.7,
E1.6, E1.1 = PASS ; D1 CLOSE.

## Décisions actives

- **`U6` en rotation 180°, centre `(285, 175)`.** Les deux flancs du `HTSSOP-44` ne sont pas
  interchangeables : le bas niveau d'un côté, toute la puissance de l'autre. La rotation
  tourne la puissance vers l'intérieur de la carte et laisse le bas niveau échapper vers la
  lisière droite.
- **Les canaux sont en rangées, pas en colonnes.** Les quatre broches `OUT` quittent `U6` sur
  un seul flanc en huit millimètres ; les rangées gardent chaque paire BTL adjacente et
  ramènent l'écart de longueur de nœud commuté entre canaux de 44 mm à 18 — ce qui prime sur
  la symétrie gauche-droite à 48 V et ~450 kHz.
- **L'arête arrière est à `y` = 250, par conséquence et non par préférence** : le bulk est
  figé sur `y` 119..182, donc une entrée 48 V plus haut aurait tiré les sorties haut-parleur
  sur la même arête, là où `C312`–`C315` barrent la route aux tores.
- **La barre de liaison est dressée** : 10 mm dans le plan de la carte, 60 mm de hauteur,
  600 mm² de section conservés, ombre portée réduite à 10 mm. **Exigence non négociable : elle
  s'élargit en pied côté flanc pour y présenter au moins 1200 mm²**, sinon la marge thermique
  tombe à 0,09 °C/W. Fixation arrêtée : deux M3 en `(285, 163)` et `(285, 187)`, entraxe 24 mm,
  encadrant `U6` — deux vis, pour l'empêcher de pivoter sur le PowerPAD. Détail et budgets
  dans `docs/architecture.md`.
- **Contour arrêté à 200 × 150 mm**, `(100,100)`–`(300,250)`, contraint par le placement et non
  par le coffret ; resserrable après E1.5.
- **Coffret Modushop `03/300` 3U, carte à plat**, isolation reportée à la jonction
  barre/flanc. Chaîne 1,711 à 1,932 °C/W pour 2,232 de budget.
- **`REQ-THERM-3`** : nominal continu 2 × 100 W sur **8 Ω**, le 4 Ω en crête seulement.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de
  configuration s'éditent directement**.
## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, contour 200 × 150, **48 placées**, 76 encore
  dans le bloc d'import autour de `x` 221..321 / `y` 272..299
- `HifiAmp_TPA3255.kicad_dru` — règle d'isolation intra-empreinte de E1.9
- `HifiAmp_TPA3255.kicad_sym` — symboles locaux `LM2940IMP_12_FIXED`, `TLV1117_33_FIXED`
- **`docs/kicad-operations.md` — comment piloter KiCad ici** : séquence IPC, pièges MCP,
  proscriptions, relecture hors MCP. À lire avant toute manipulation de la carte.
- `docs/architecture.md` — contraintes de placement actives, `NEEDS_DATA`, budgets thermiques
- **État KiCad** : fermé, aucun verrou `~*.lck`.

## NEXT ACTION

**E1.3, tranche « alimentation auxiliaire » — placer le buck `LM5010` et sa descendance.**
Sortir du bloc d'import les 26 empreintes du bloc — `U1`, `L1`, `D1`, `D3`, `C1`–`C4`, `C11`,
`C39`, `C12`, `C13`, `R2`, `R39`, `R40`, `C6`, `C8`, `U2`, `C5`, `C9`, `C38`, `U3`, `C10`,
`L6`, `C81`, `C82` — vers la bande **`x` 160..190, `y` 120..195**, libre entre la chaîne
d'entrée 48 V (qui s'arrête à `y` ≈ 199) et le haut de carte. Cette bande touche `U8`/`Q302`,
d'où sort `PVDD`, et laisse **`x` 100..155 pour l'analogique bas niveau de E1.4**, au bord
opposé de `U6` comme l'exige l'orientation dans le coffret. Serrer la boucle de commutation
`D3`/`C2`/`C3`/`C4`/`U1`/`D1`/`L1` avant tout confort de routage ; garder `R39`/`R40` — le
diviseur de retour — courts et à l'écart du nœud `SW`.

Lancer KiCad par `explorer.exe`, ouvrir l'éditeur de PCB depuis le gestionnaire, déplacer en
IPC via `kicad-control` (cf. `docs/kicad-operations.md`). Valider par relecture du fichier —
positions exactes, 48 placements antérieurs intacts au micron, 124 empreintes — puis par
`kicad-cli pcb drc --format json` : `schematic_parity` reste 0, 254 non-connectés inchangés,
aucune violation `clearance`, aucun type nouveau.
