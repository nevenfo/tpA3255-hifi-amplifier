# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a, F1.2-b1 et F1.2-b2 closes, F1.2-b3 routée mais non validée.**
122 segments, **8 vias**. DRC courant : **111 violations toutes de sérigraphie** (89
`silk_overlap`, 22 `silk_over_copper`), **216 non-connectés**, aucun sur `/OUT_A`–`/OUT_D`,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`,
aucun `courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.2-c2 — appliquer le placement O1 et rerouter l'étage de sortie.** État courant : routage
F1.2-b3 (`ce19b50`) propre au DRC, sélectivité DRU prouvée, mais cols de 0,96–1,07 mm et
`OUT_D` de 101 mm ; il sera repris.

## Dernière tâche validée

**F1.2-c1 = PASS** — étude en lecture seule, option **O1 « brique stricte »** retenue ; positions,
bootstraps, vias et chiffres dans `docs/architecture.md`, section **F1.2-c**.

Validation :

- Courtyards et contour vérifiés par géométrie, aucun chevauchement ; longueurs estimées A 27,9 /
  B 27,9 / C 67,2 / D 70,0 mm ; écarts A/B 0,0, C/D 2,8 ; quatre `OUT` et quatre `OUT_x_F`
  planaires ; MD5 de la carte inchangé.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`,
sections **F1.2-b** et **F1.2-c**.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face.
- **Règle DRU d'échappement `U6` : sélectivité prouvée** en F1.2-b3, à rejouer après c2.
- **Sérigraphie à traiter en bloc** ; **masses et rails auxiliaires longs au plan de F1.4**.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `199511c0266fd228e2d84d4a10a298d9` ; `.kicad_dru` porte les
  deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md`, section **F1.2-b** — mesure d'échappement et courant (IPC-2221)

## NEXT ACTION

**F1.2-c2 — appliquer O1** : déplacer `L301`–`L304` et `C306`–`C309` aux positions de
`docs/architecture.md` § F1.2-c, remplacer les vias `BST` en `x` = 274,6, rerouter `OUT_x`,
`OUT_x_F` et `BST_B`/`BST_C`. Validation : DRC sans `clearance`/`shorting_items`/`track_dangling`/
`courtyards_overlap`, parité 3, non-connectés 216, 3,00 mm hors pas fin, écarts A/B et C/D
chiffrés, 124 empreintes, 119 `(units`.
