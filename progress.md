# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **102 violations, 254 non-connectés, `schematic_parity` = 0** — 99 de sérigraphie,
normales avant routage, plus 3 `lib_footprint_mismatch` (cf. décisions et E1.10).

## Tâche actuelle

**E1.3 est close.** **83 des 124 empreintes sont placées**, 41 restent dans le bloc d'import.
La suite est **E1.4 — analogique faible bruit, volume et contrôles**, dans la bande
`x` 100..155 réservée à cet effet, plus **E1.10**, ouverte en cours de phase : rétablir
`(property pad_prop_heatsink)` sur le pad exposé de `U1`.

## Dernière tâche validée

**E1.3 = PASS, tranche « supervision et configuration » comprise.** `U7` et son réseau —
`R32`/`R26`/`C67` sur `VSENSE`, `R6`/`C83`/`C84` sur `RESET_RC` — occupent la bande sous `U3`,
`x` 156..184 / `y` 122..134. Les quatre résistances de configuration de `U6` sont en colonne
`x` = 297 : leurs broches sortant toutes du flanc `x` = 288,71 sur `y` 168,97..177,86, donc en
pleine zone d'interdiction de la barre, aucune ne pouvait se poser en regard de la sienne.

Validation :

- DRC **102 violations, dont 99 de sérigraphie** ; **aucune `clearance`, aucun
  `courtyards_overlap`, aucun `copper_edge_clearance`** ; `lib_footprint_mismatch` toujours 3,
  `U7` n'en ajoute aucun ; `schematic_parity` = 0 ; 254 non-connectés inchangés.
- 124 empreintes, les 11 positions et les 72 placements antérieurs relus du fichier
  enregistré, intacts au micron.

**Avant elle** : les tranches « alimentation auxiliaire » (`0c4e3c7`), « entrée 48 V »
(`1ae53af`) et « filtres de sortie » (`ec83d2f`) ; E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

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
- **Les trois `lib_footprint_mismatch` ont deux causes distinctes, et ce n'est pas la perte de
  `(units)`.** Tout déplacement IPC fait réécrire l'empreinte et lui retire ce bloc, mais `U7`
  le perd **sans** produire de mismatch : l'hypothèse initiale est réfutée. Pour `U2`/`U3`,
  c'est la fermeture implicite d'un polygone de sérigraphie de SOT-223 — cosmétique. Pour
  `U1`, c'est **la perte de `(property pad_prop_heatsink)` sur son pad exposé**, seule
  occurrence de la carte, disparue exactement au commit qui l'a déplacé : réelle, et traitée
  par **E1.10**. Aucun UUID perdu, 4543 avant comme après ; aucun net changé.
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

**E1.10 — rétablir `(property pad_prop_heatsink)` sur le pad exposé de `U1`**, avant d'ouvrir
E1.4. C'est une régression de l'outil, pas une décision : la carte n'en portait qu'une
occurrence, sur le pad 11 du `LM5010`, et elle disparaît exactement au commit qui déplace `U1`.
Sans effet sur le placement, mais bloquante pour le remplissage de zones en F1.4 et pour le
pochoir en H2 — donc à réparer tant que le contexte est frais plutôt qu'à découvrir au routage.

Voie à privilégier : **« Mettre à jour les empreintes depuis la bibliothèque »** dans l'éditeur
de PCB, qui rétablirait du même coup les deux polygones normalisés de `U2`/`U3`, donc les trois
`lib_footprint_mismatch`. **Action isolée, et à surveiller : elle touche toutes les empreintes
de la carte.** Relire les 83 positions avant et après.

Valider : `pad_prop_heatsink` présent une fois dans le fichier, `lib_footprint_mismatch` à 0,
`schematic_parity` toujours 0, 254 non-connectés, 124 empreintes, **83 placements intacts au
micron**, et aucune `clearance` ni `courtyards_overlap`. Si l'action déplace ou altère quoi que
ce soit d'autre, l'abandonner et revenir au fichier commité plutôt que de rattraper à la main.
