# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **99 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage et nettoyage. Plus aucun `lib_footprint_mismatch`.

## Tâche actuelle

**E1.3 et E1.10 sont closes.** **83 des 124 empreintes sont placées**, 41 restent dans le bloc
d'import. La suite est **E1.4 — analogique faible bruit, volume et contrôles** : les deux
canaux symétriques (séries 100 et 200), les OPA `U4`/`U5`, les entrées RCA `J2`/`J3`, le
volume `J4`, et `C81`/`C82` sur le rail `+12V-OA`.

## Dernière tâche validée

**E1.10 = PASS — la régression du pad thermique de `U1` est réparée.** « Mise à Jour des
Empreintes à partir des Librairies » sur toute la carte, options texte décochées pour ne
réinitialiser ni position de référence ni contenu de champ.

Validation :

- `pad_prop_heatsink` revient à **exactement une occurrence**, sur le pad 11 de `U1`.
- Les trois `lib_footprint_mismatch` tombent à **0**. DRC **99 violations, toutes de
  sérigraphie** ; `schematic_parity` = 0 ; 254 non-connectés inchangés ; 124 empreintes.
- **Aucune des 124 positions ni rotations ne bouge**, comparées une à une au commit précédent.
- Le diff ne touche **aucun net** : il rétablit la propriété, referme deux polygones de
  sérigraphie de SOT-223, aligne quatre styles de trait, et retire les 62 blocs `(units)`
  restants — le fichier n'en porte plus aucun.

**Avant elle** : E1.3 close en quatre tranches (`f9a326c`, `0c4e3c7`, `1ae53af`, `ec83d2f`) ;
E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

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

**E1.4 — placer l'analogique faible bruit, le volume et les contrôles**, les 41 empreintes
restantes, dans la bande **`x` 100..155** réservée depuis E1.3, au bord opposé de `U6`. Attention :
la chaîne d'entrée 48 V y descend déjà — `J1` est en (144, 241) — donc la bande utile s'arrête
vers `y` 230.

Deux canaux à traiter **symétriquement** : série 100 et série 200, chacun avec son OPA (`U4`,
`U5`), ses résistances 0,1 % de gain (`R101`–`R104`, `R201`–`R204`) et ses `100R` de série vers
le TPA (`R107`/`R108`, `R207`/`R208`). Poser d'abord les OPA et les résistances de précision,
qui fixent tout le reste. `C110`/`C210` restent **chacun au plus près de son OPA** — c'est la
raison d'être de leur dédoublement sur `VMID`. `J2`/`J3`/`J4` sont en JST XH déporté : leur position est un choix de câblage, à
poser en lisière. `C81`/`C82` suivent `+12V-OA` en aval de `L6`, déjà posé en (162, 139).

Lancer KiCad par `explorer.exe`, ouvrir l'éditeur de PCB depuis le gestionnaire, déplacer en
IPC via `kicad-control` (cf. `docs/kicad-operations.md`), par tranches d'une vingtaine
d'empreintes. Valider par relecture du fichier — positions exactes, 83 placements antérieurs
intacts au micron, 124 empreintes — puis par `kicad-cli pcb drc --format json` **lancé depuis
le répertoire du projet** : `schematic_parity` reste 0, 254 non-connectés, aucune `clearance`,
aucun `courtyards_overlap`, et **`lib_footprint_mismatch` toujours 0** — s'il en réapparaît un
sur `U4` ou `U5`, vérifier la propriété de pastille avant de l'admettre.
