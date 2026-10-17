# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a, F1.2-b1 et F1.2-b2 closes, F1.2-b3 à faire.** 66 segments,
**8 vias**. DRC courant : **111 violations toutes de sérigraphie** (89 `silk_overlap`, 22
`silk_over_copper`), **226 non-connectés** dont **10 sur `/OUT_A`–`/OUT_D`**,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`,
aucun `courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.2-b3 — router `/OUT_A` à `/OUT_D` de `U6` aux bootstraps puis aux selfs.**

## Dernière tâche validée

**F1.2-b2 = PASS, nombre de vias révisé par la mesure.** `BST_B`/`BST_C` plongent sur `B.Cu`
avec **2 vias par net** : sorties en (279 ; 181,0325) et (279 ; 168,9675), remontées en
(274,6 ; 177,275) et (274,6 ; 172,725), vers `C307`/`C308` pad 1. `BST_A`/`BST_D` contournent
sur `F.Cu` par `y` 182,4 et 167,6, sans via. Tout en 0,35 mm.

Validation :

- **Vérification du principal** : DRC `kicad-cli --schematic-parity` identique à F1.2-b1
  (0 `clearance`/`shorting_items`/`track_dangling`/`courtyards_overlap`, parité 3, 226) ;
  **seuls les nets `/BST_A`–`/BST_D` changent**, aucune empreinte ; **0 segment et 0 via `F.Cu`
  dans `x` ∈ [276,15 ; 278,85], `y` ∈ [168 ; 182]** ; 119 `(units`.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`,
section **F1.2-b**.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Chevelus `OUT` à résoudre** : `U6` 27/28 → `C309` pad 2 → `L304` (`OUT_D`) ; 32 → `C308`
  pad 2 → `L303` (`OUT_C`) ; 35 → `C307` pad 2 → `L302` (`OUT_B`) ; 39/40 → `C306` pad 2 →
  `L301` (`OUT_A`).
- **Couloir mesuré** : pour `x` ∈ [276 ; 278,7], bande `y` 168,275–181,725 (13,45 mm) ; resserrée
  à **10,46 mm** (`y` 169,77–180,23) près des vias `BST` en `x` ≈ 278,7–279,3. Quatre voies à
  3,00 mm + 3 isolations de 0,5 mm demandent 13,5 mm : **largeur d'échappement réduite à
  justifier dans le resserrement**, puis reprise à 3,00 mm ; sinon décaler les vias `BST` en
  (279 ; …) vers l'ouest. À gauche de `x` = 276 : vias (274,6 ; …) et pastilles `C306`–`C309`
  en `x` 273,03–273,98.
- **Règle DRU d'échappement** : sélectivité à prouver ici, en retirant `U6` de sa condition.
- **Sérigraphie à traiter en bloc** ; **masses et rails auxiliaires longs au plan de F1.4**.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `db7db5ceba5dd712652c83541963aad2` ; `.kicad_dru` porte les
  deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md`, section **F1.2-b** — mesure d'échappement et courant (IPC-2221)

## NEXT ACTION

**F1.2-b3 — router les quatre `OUT` de `U6` à leurs bootstraps puis à leurs selfs, sur `F.Cu`.**
Validation : non-connectés **226 → 216**, largeur d'échappement justifiée puis 3,00 mm hors du pas
fin, 0 `clearance`/`shorting_items`/`track_dangling`, parité 3, **sélectivité de la règle DRU
prouvée**, 124 empreintes, 119 `(units`.
