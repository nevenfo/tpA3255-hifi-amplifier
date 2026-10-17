# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a et F1.2-b1 closes, F1.2-b2 à faire.** 62 segments, **4 vias**. DRC
courant : **111 violations toutes de sérigraphie** (89 `silk_overlap`, 22 `silk_over_copper`),
**226 non-connectés**, `schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`,
aucun `track_dangling`, aucun `courtyards_overlap`. Compteurs de structure : **124 empreintes,
119 blocs `(units`, `pad_prop_heatsink` = 1**. `C310`/`C311` sur `B.Cu`, le reste sur `F.Cu`.

## Tâche actuelle

**F1.2-b2 — faire passer `BST_B` et `BST_C` sous la rangée `OUT` par vias.**

## Dernière tâche validée

**F1.2-b1 = PASS, critère de boucle révisé par décision utilisateur.** `C310`/`C311` au dos,
tournés de 180°, glissés sous `U6` en (280,75 ; 178,3) et (280,75 ; 171,7). 4 vias tentées
ø0,6/0,3 en `x` = 282,6 sous le corps de `U6`, à l'est des broches : `/PVDD` `y` 172,7775 et
177,2225, `/GND` `y` 169,92 et 180,08.

Validation :

- **Vérification du principal** : DRC `kicad-cli --schematic-parity` → 0 `clearance`/
  `shorting_items`/`track_dangling`/`courtyards_overlap`, parité **3**, non-connectés **226**,
  aucun sur `C310`/`C311` ; fins de ligne normalisées, **seules les empreintes `C310`/`C311`
  changent** ; 119 `(units` (celui de `C311`, perdu au déplacement, réinséré au fichier).
- **Boucle 9,63 mm par condensateur** (centre de pastille → centre de broche + 2 × 1,6 mm),
  recalculée par le principal ; ≈ 10,8 au `x` d'origine. ≤ 7,71 était intenable avec des vias ;
  9,63 est le minimum sans via dans une pastille.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`,
section **F1.2-b**.

- **Croisement `BST`/`OUT` dénoué côté `BST`, jamais sur un `OUT` à 5 A.** Carte non simple-face.
- **À vérifier en F1.2-b2 : « une via par `BST` » est probablement physiquement impossible** —
  `C307`/`C308` sont des CMS `F.Cu`, une plongée sur `B.Cu` doit remonter, donc **2 vias par net,
  4 au total**. Si la mesure le confirme, réviser la validation du plan (« 2 vias exactement »).
- **`B.Cu` n'est plus vide** sous `U6` : `C310`/`C311` couvrent `x` 279,4–282,1, et leurs pistes
  vont aux vias en 282,6. Les plongées `BST` doivent les contourner.
- **Colonne `x` ∈ [276,15 ; 278,85] à libérer sur `F.Cu` en F1.2-b2** (critère hérité de b1) :
  4 segments `BST` y passent encore.
- **Règle DRU d'échappement** : sélectivité à prouver en F1.2-b3.
- **Sérigraphie à traiter en bloc** ; **masses et rails auxiliaires longs au plan de F1.4**.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `b260b44c1e9724b576218e71813d589e`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation** (API à activer, named pipe,
  `(units` perdus au déplacement, fermer l'éditeur sans enregistrer après restauration au fichier)
- `docs/architecture.md`, section **F1.2-b** — ordre inversé `BST_B`/`OUT_A` et `BST_C`/`OUT_D`

## NEXT ACTION

**F1.2-b2 — rerouter `BST_B` (broche 43 → `C307` pad 2) et `BST_C` (→ `C308`) par vias sous la
rangée `OUT`, en libérant la colonne `x` ∈ [276,15 ; 278,85] sur `F.Cu`.** Validation : nombre de
vias justifié (2 par net attendu), aucun `track_dangling`, 0,35 mm en sortie de boîtier, 0
`clearance`/`shorting_items`, parité 3, non-connectés **≤ 226**, 124 empreintes, 119 `(units`.
