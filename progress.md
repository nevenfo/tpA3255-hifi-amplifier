# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a close, F1.2-b débloquée et en cours.** **58 segments, zéro via** : les
quatre boucles de bootstrap, `PVDD`, les découplages auxiliaires (F1.1), et les quatre sorties
filtrées `self → condensateur → bornier` à 3,00 mm (F1.2-a). DRC courant : **113 violations toutes
de sérigraphie** (91 `silk_overlap`, 22 `silk_over_copper`), **230 non-connectés**,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`.
Compteurs de structure : **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1**.

## Tâche actuelle

**F1.2-b1 — basculer `C310`/`C311` sur `B.Cu` et rerouter `/PVDD` par vias.** Première des trois
unités ouvertes par l'arbitrage ; F1.2-b2 puis F1.2-b3 suivent. Arrêtée sur un geste GUI, voir
« Blocage actif ».

## Dernière tâche validée

**F1.2-a = PASS.** Les quatre nets `/OUT_x_F` sont routés, **23 segments à 3,00 mm, zéro via**,
après deux échanges d'emplacement entre composants identiques — `L302` ↔ `L303` et `C322` ↔ `C323`
— qui alignent l'ordre des condensateurs sur celui des borniers et rendent la topologie planaire.

Validation :

- **Non-connectés 238 → 230**, les huit chevelus prédits ; **0 `clearance`, 0 `shorting_items`,
  0 `track_dangling`, 0 `solder_mask_bridge`, 0 `copper_edge_clearance`** ; `schematic_parity`
  toujours **3** ; sérigraphie **113 inchangée**.
- **Vérification indépendante du principal** : DRC relancé en `kicad-cli --schematic-parity` ;
  **58 segments tous sur `F.Cu`**, **0 via** ; **zéro intersection** entre les 23 segments, testée
  segment à segment ; diff Git ne montrant **que les quatre `(at)` échangés et les 23 segments**.

## Décisions actives

Les règles durables sont dans `docs/kicad-operations.md`, le placement et les mesures dans
`docs/architecture.md`, section **F1.2-b**. Restent ici :

- **Arbitrage F1.2-b rendu, en deux gestes indépendants.** `C310`/`C311` **passent au dos sur
  `B.Cu`**, vias sous les broches `PVDD`/`GND` — la colonne `x` ∈ [276 ; 279] se libère et la
  boucle de découplage **raccourcit** au lieu de s'allonger. Le croisement structurel se dénoue
  par **une via sur chaque `BST`**, jamais sur un `OUT` à 5 A. **Coût assumé : la carte cesse
  d'être simple-face**, second passage d'assemblage pour deux composants.
- **Règle DRU d'échappement posée et prouvée inerte** ; **sa sélectivité reste à prouver** en
  retirant `U6` de sa condition, quand une piste l'exercera — c'est une validation de F1.2-b3.
- **113 violations de sérigraphie restent à traiter en bloc**, aucune n'ayant d'effet cuivre.
- **Masses locales et rails auxiliaires trop longs partent au plan de F1.4**, où `AVDD` et `DVDD`
  — 28,95 et 34,82 mm — seront réexaminés. Un seul net `/GND`, aucune zone dessinée.

## Blocage actif

**F1.2-b1 attend un geste GUI que l'automatisation ne peut pas garantir : le bureau est occupé.**
Aucun outil MCP ne retourne une empreinte placée — piège consigné dans `docs/kicad-operations.md`
—, donc `C310`/`C311` ne passent sur `B.Cu` que par l'interface. Le geste a été tenté puis
abandonné **sans aucun clic** : la fenêtre au premier plan a basculé seule vers une fenêtre Excel,
et la position du curseur a changé entre deux lectures **sans mouvement émis**. Le canevas de
l'éditeur n'expose aucun élément UIA pour les empreintes — sélection par coordonnées pixel
seulement —, donc un focus disputé retournerait la mauvaise empreinte. **Ce n'est pas le blocage
`SendInput` de session RDP déjà connu** : l'activation de fenêtre réussissait.

**Prochaine tentative : rejouer le geste quand le poste est libre.** L'IPC n'est pas affecté, la
suite de F1.2-b1 reprend par MCP dès le flip constaté.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `2cc389a517fef7deb02fb5a290e66bef`, inchangé ; `.kicad_pro`
  porte les classes de net, `.kicad_dru` les deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation**
- `docs/architecture.md`, section **F1.2-b** — champ d'échappement, mesure et arbitrage rendu

## NEXT ACTION

**F1.2-b1 — retourner `C310` et `C311` sur `B.Cu` par l'interface (sélection, `F`, `Ctrl+S`), puis
rerouter `/PVDD` par vias sous les broches `PVDD` de `U6`.** Seul le retournement demande le
geste ; le reste passe par IPC. Ancres : `C310` en (277,5 ; 178,3) rotation −90°, `C311` en
(277,5 ; 171,7) rotation +90°, deux `Capacitor_SMD:C_1210_3225Metric` alignés juste à gauche de
`U6` ; pastilles `/PVDD` de `U6` en `x` = 281,2875, pads 29/30/31 en `y` = 172,1425 / 172,7775 /
173,4125 et pads 36/37/38 en `y` = 176,5875 / 177,2225 / 177,8575, les pistes F1.1 touchant le pad
médian de chaque groupe. Valider par : boucle de découplage **≤ 7,71 mm** de trajet équivalent ;
colonne `x` ∈ [276,15 ; 278,85] libre de cuivre `F.Cu` ; DRC sans `clearance`, `shorting_items` ni
`track_dangling` ; parité **3** ; non-connectés **inchangés à 230** ; **124 empreintes et 119
blocs `(units`**.
