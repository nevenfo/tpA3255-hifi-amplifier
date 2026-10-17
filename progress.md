# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a close, F1.2-b en cours.** 68 segments, **4 vias** (les premières de la
carte). DRC courant : **111 violations toutes de sérigraphie** (89 `silk_overlap`, 22
`silk_over_copper`), **226 non-connectés**, `schematic_parity` = **3**, aucune `clearance`,
aucun `shorting_items`, aucun `track_dangling`. Compteurs de structure : **124 empreintes, 119
blocs `(units`, `pad_prop_heatsink` = 1**. `C310`/`C311` sur `B.Cu`, toutes les autres sur `F.Cu`.

## Tâche actuelle

**F1.2-b1 — `C310`/`C311` au dos, `/PVDD` et `/GND` reroutés par vias.** Câblage fait et DRC propre,
**mais le critère de boucle ≤ 7,71 mm échoue** : la tâche reste ouverte, en attente d'arbitrage.

## Dernière tâche validée

**F1.2-a = PASS** (quatre sorties filtrées, 23 segments à 3,00 mm, zéro via). Depuis, F1.2-b1
a produit un état **sain mais non validé** :

Validation :

- `C310`/`C311` tournés de 180° sur `B.Cu` (−90 et 90) : chaque pastille fait face à sa broche
  — `C311` GND 170,225 ↔ broches 25/26, PVDD 173,175 ↔ 29–31 ; `C310` PVDD 176,825 ↔ 36–38,
  GND 179,775 ↔ 41/42. Sans cette rotation, le flip imposait un croisement.
- 4 vias ø0,6/0,3 : `/PVDD` (279,9 ; 172,7775) et (279,9 ; 177,2225) au bout des segments 1,1 mm
  F1.1 ; `/GND` (280,55 ; 169,85) et (280,55 ; 180,2), décalées à droite des diagonales
  `BST_C`/`BST_B`. Segments 0,8 mm F1.1 orphelins supprimés.
- **Vérification du principal** : DRC `kicad-cli --schematic-parity` → 0 `clearance`/
  `shorting_items`/`track_dangling` (HEAD en avait 2), non-connectés **232 → 226**, les six
  résolus étant ceux de `C310`/`C311` ; diff, fins de ligne normalisées : **seules les empreintes
  `C310`/`C311` changent**.
- **Boucle, même métrique que F1.1 (centre de pastille → centre de broche le long des pistes)** :
  PVDD 3,79 + GND 3,83 + 2 × 1,6 de traversée ≈ **10,8 mm par condensateur, contre 7,71**.
  Le condensateur n'ayant pas bougé en `x`, les traversées s'**ajoutent** au trajet plan :
  l'affirmation « la boucle raccourcit » de l'arbitrage F1.2-b était fausse.

## Décisions actives

Les règles durables sont dans `docs/kicad-operations.md`, le placement et les mesures dans
`docs/architecture.md`, section **F1.2-b**. Restent ici :

- **Arbitrage F1.2-b** : `C310`/`C311` au dos ; le croisement `BST`/`OUT` se dénoue par **une via
  sur chaque `BST`** (F1.2-b2), jamais sur un `OUT`. Carte non simple-face, coût assumé.
- **La colonne `x` ∈ [276,15 ; 278,85] n'est libre qu'après F1.2-b2** : `BST_B`/`BST_C` (et les
  amorces `BST_A`/`BST_D`) la traversent encore ; ce critère de F1.2-b1 se constate en F1.2-b2.
- **Règle DRU d'échappement posée et prouvée inerte**, sélectivité à prouver en F1.2-b3.
- **Sérigraphie à traiter en bloc** ; **masses et rails auxiliaires longs au plan de F1.4**.

## Blocage actif

**Décision utilisateur : le critère de boucle ≤ 7,71 mm n'est pas tenable avec les condensateurs
à leur `x` actuel.** Options mesurées :

1. **Accepter ≈ 10,8 mm** et réviser le critère (aucune reprise).
2. **Glisser `C310`/`C311` vers la droite sous `U6`** sur `B.Cu`, extrémité de pastille sur les
   vias (`x` ≈ 279,9) : ≈ **8,7 mm**, coût nul en fabrication, la face dessous sous `U6` étant
   libre (PowerPAD sur le dessus).
3. **Vias dans les pastilles** des condensateurs : ≈ **5,4 mm**, mais vias bouchées et capées
   (option de fabrication payante).

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `7257cdc74fc2190b1a102b232268074e` ; `.kicad_pro` porte les
  classes de net, `.kicad_dru` les deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — pilotage KiCad, **à lire avant toute manipulation** (API à
  activer, named pipe, `(units` perdus au déplacement)
- `docs/architecture.md`, section **F1.2-b**

## NEXT ACTION

**F1.2-b1 — obtenir l'arbitrage sur la boucle de découplage (options 1, 2 ou 3 ci-dessus), puis
l'appliquer** ; validation : boucle au nouveau critère retenu, DRC sans `clearance`,
`shorting_items` ni `track_dangling`, parité **3**, non-connectés **≤ 226**, **124 empreintes,
119 `(units`**.
