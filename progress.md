# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **104 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage et nettoyage. Plus aucun `lib_footprint_mismatch`.

## Tâche actuelle

**E1.5 — revue du placement : symétrie, retour des courants, masses, clearances,
manufacturabilité.** Rien n'est encore posé ; à découper en tranches comme E1.3 et E1.4.

## Dernière tâche validée

**E1.4 = PASS, et le placement de la carte est complet : les 124 empreintes sont posées.**
La tranche finale a posé les 21 dernières en **miroir du canal gauche, décalées de +40 mm
en `y`** — `U5` en (130, 185) contre `U4` en (130, 145), chaque satellite de la série 200 à
l'homologue de son jumeau de la série 100 — plus `C81`/`C82` sur `+12V-OA` et la connectique
`J3`/`J4` en façade. Le bloc d'import est vide.

Validation :

- DRC **104 violations, toutes de sérigraphie** — 85 `silk_overlap`, 19 `silk_over_copper` ;
  aucune `clearance`, aucun `courtyards_overlap`, `lib_footprint_mismatch` toujours **0**.
- `pad_prop_heatsink` toujours présent une fois : la régression de E1.10 ne s'est pas refaite.
- `schematic_parity` = 0 ; 254 non-connectés ; 124 empreintes ; **plus aucune empreinte
  au-delà de `y` = 260**.
- Le diff ne touche **que 21 lignes `(at ...)`** et aucune autre : les 103 placements
  antérieurs sont intacts au micron, prouvé par le diff lui-même.

**Avant elle** : E1.10 (`73abdab`) ; E1.3 close en quatre tranches ; E1.9, E1.2, E1.8, E1.7,
E1.6, E1.1 = PASS.

## Décisions actives

Les décisions de placement figées — rotation de `U6`, canaux en rangées, arête arrière à
`y` = 250, rotation de `U1`, colonne `x` = 297 pour `R301`–`R304`, bande `x` 100..155 réservée
à l'analogique — sont dans `docs/architecture.md`, section « Décisions de placement figées en
E1 ». Le pilotage KiCad est dans `docs/kicad-operations.md`. Restent ici celles qui gouvernent
encore la prochaine action :

- **Les deux voies analogiques sont des translations exactes l'une de l'autre**, +40 mm en `y`,
  même `x`. C'est délibéré : E1.5 doit pouvoir juger la symétrie sur la géométrie mesurée, pas
  sur l'intention déclarée.
- **La barre de liaison est dressée** : 10 mm dans le plan de la carte, 60 mm de hauteur,
  600 mm² de section. **Exigence non négociable : pied élargi d'au moins 1200 mm² côté flanc**,
  sinon la marge thermique tombe à 0,09 °C/W. Deux M3 en `(285, 163)` et `(285, 187)`, entraxe
  24 mm. **Zones interdites aux composants** : `x` ∈ [280, 291], `y` ∈ [160, 190], et
  `x` ∈ [280, 300], `y` ∈ [169, 181].
- **Contour arrêté à 200 × 150 mm**, `(100,100)`–`(300,250)` ; resserrable après E1.5.
- **Un déplacement IPC peut perdre une propriété de pastille, et le DRC le dit.** `U1` a perdu
  `(property pad_prop_heatsink)` en étant déplacé — seule occurrence de la carte. La réparation
  est « Mise à Jour des Empreintes à partir des Librairies », options texte décochées.
  **À vérifier après tout déplacement de circuit intégré porteur d'un pad exposé.**
- **Un DRC de comparaison se lance dans le répertoire du projet**, sinon il compte de faux
  `lib_footprint_issues` de librairie introuvable et la baseline est fausse. Le binaire est
  `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
- **`REQ-THERM-3`** : nominal continu 2 × 100 W sur **8 Ω**, le 4 Ω en crête seulement. Coffret
  Modushop `03/300` 3U, carte à plat.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de
  configuration s'éditent directement**.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes, contour 200 × 150, **toutes placées**
- `HifiAmp_TPA3255.kicad_dru` — règle d'isolation intra-empreinte de E1.9
- `HifiAmp_TPA3255.kicad_sym` — symboles locaux `LM2940IMP_12_FIXED`, `TLV1117_33_FIXED`
- **`docs/kicad-operations.md` — comment piloter KiCad ici** : séquence IPC, pièges MCP,
  proscriptions, relecture hors MCP. À lire avant toute manipulation de la carte.
- `docs/architecture.md` — contraintes de placement actives, `NEEDS_DATA`, budgets thermiques
- **État KiCad** : fermé, aucun verrou `~*.lck`.

## NEXT ACTION

**E1.5, tranche « symétrie mesurée des deux voies » — audit en lecture seule, aucune écriture
de carte.** Extraire du `.kicad_pcb` la position et la rotation de chaque empreinte de la série
100 et de son homologue de la série 200, puis vérifier que l'écart est **exactement (0, +40)**
et la rotation identique pour chaque paire. Reporter les paires qui dévient, et pour chacune
dire si l'écart est **délibéré** — un composant sans jumeau, ou une contrainte locale — ou une
**erreur de placement à corriger**. Étendre la même mesure aux étages de sortie des deux voies
s'ils portent des références appariées.

Livrable : la liste des paires conformes, la liste des écarts avec leur verdict, et une
conclusion tranchée sur le fait que la symétrie inter-voies est acquise ou non. Aucune
modification du `.kicad_pcb` dans cette tranche : c'est une mesure, et elle doit rester
comparable à celle qu'on refera après correction.

Validation : le rapport couvre **toutes** les paires 1xx/2xx existantes sans en omettre, chaque
écart porte un verdict motivé, et le `.kicad_pcb` est **identique au bit près** avant et après.
