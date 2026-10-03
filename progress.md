# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a close, F1.2-b débloquée et en cours.** **58 segments, zéro via** : les
quatre boucles de bootstrap, `PVDD`, les découplages auxiliaires (F1.1), et les quatre sorties
filtrées `self → condensateur → bornier` à 3,00 mm (F1.2-a). DRC courant : **113 violations toutes
de sérigraphie** (91 `silk_overlap`, 22 `silk_over_copper`), **230 non-connectés**,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`.
Compteurs de structure : **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1**.

## Tâche actuelle

**F1.2-b1 — basculer `C310`/`C311` sur `B.Cu` et rerouter `/PVDD` par vias.** Moitié 1 faite :
le flip est commité (`7114e41`, Konnect v1.2.0 `flip_component`, `kicad-cli pcb export pos` les
donne `bottom`). Reste le reroutage `/PVDD`/`GND` par vias sous `U6`.

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

**Le reroutage exige l'IPC, donc l'éditeur de PCB ouvert ; la session Windows est sur le bureau
sécurisé** (verrouillage ou invite UAC : `SecureDesktopActive` sur KiCad comme sur Firefox).
KiCad tourne, lancé par `explorer.exe`, gestionnaire de projet seul ; le double-clic sur
`HifiAmp_TPA3255.kicad_pcb` n'a pu être émis. Carte inchangée. **Reprise : session déverrouillée
→ ouvrir l'éditeur de PCB depuis le gestionnaire (sans schéma) → reroutage par IPC.**

Relevés pour le reroutage, faits sur le fichier courant :

- Le flip a **inversé les pastilles** : `/PVDD` à l'extérieur, `/GND` à l'intérieur — `C310`
  pad 1 PVDD (277,5 ; 179,775), pad 2 GND (277,5 ; 176,825) ; `C311` pad 1 PVDD (277,5 ;
  170,225), pad 2 GND (277,5 ; 173,175), `B.Cu`, 1,15 × 2,7 mm.
- Segments F1.1 `/PVDD` 0,8 mm dangling, à supprimer : `e2dc4dc0-5a66-404f-a7c1-66630b4ea0c0`,
  `22785dab-2adc-4eec-84f0-d3940d274222` ; les 1,1 mm (279,9 → 281,2875) `137c5f6d…`,
  `09acd9a7…` restent et servent de sortie aux vias.
- **Aucune piste `GND` n'existait** ici : pistes neuves vers les broches `GND` de `U6` 25/26
  (`y` 169,60/170,24), 33/34 (174,68/175,32), 41/42 (179,76/180,40).
- Vias prévues vers `x` ≈ 279,5 : l'isolation 0,5 mm `PVDD`/`PWR_OUT` vaut via ↔ pastille, la
  règle `.kicad_dru` ne l'assouplit pas.
- **La colonne ne sera libre qu'après F1.2-b2** : `BST_C` (`y` 171,7) et `BST_B` (`y` 178,3)
  la traversent encore de `x` 275,5 à 279,25 ; ce critère de F1.2-b1 se constate donc en
  F1.2-b2.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `619eedd795f6f35dfc6af4dec222a1c1` après flip ; `.kicad_pro`
  porte les classes de net, `.kicad_dru` les deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — pilotage KiCad et pièges d'outillage, **à lire avant toute
  manipulation**
- `docs/architecture.md`, section **F1.2-b** — champ d'échappement, mesure et arbitrage rendu

## NEXT ACTION

**F1.2-b1 — rerouter `/PVDD` et `GND` de `C310`/`C311` (désormais sur `B.Cu`) par vias vers les
broches `PVDD` de `U6`, et supprimer les segments F.Cu F1.1 devenus orphelins.** Ancres : `C310`
(277,5 ; 178,3) −90°, `C311` (277,5 ; 171,7) +90°, `C_1210_3225Metric` ; pads `/PVDD` de `U6` en
`x` = 281,2875, 29/30/31 en `y` = 172,1425 / 172,7775 / 173,4125, 36/37/38 en `y` = 176,5875 /
177,2225 / 177,8575. Valider par : boucle **≤ 7,71 mm** ; colonne `x` ∈ [276,15 ; 278,85] libre
sur `F.Cu` ; DRC sans `clearance`, `shorting_items` ni `track_dangling` ; parité **3** ;
non-connectés **230** ; **124 empreintes, 119 blocs `(units`**.
