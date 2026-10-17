# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.2-a, F1.2-b1 et F1.2-b2 closes, F1.2-b3 routée mais non validée.**
122 segments, **8 vias**. DRC courant : **111 violations toutes de sérigraphie** (89
`silk_overlap`, 22 `silk_over_copper`), **216 non-connectés**, aucun sur `/OUT_A`–`/OUT_D`,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`,
aucun `courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.2-b3 — `/OUT_A`–`/OUT_D` routés sur `F.Cu`, sans via, DRC propre, 216 non-connectés.**
Sélectivité DRU **prouvée** (copie avec `U6` retiré de la condition : 14 `clearance`, toutes
piste `OUT` ↔ pastille `U6` ; 0 avec la règle). **Critère « 3,00 mm hors du pas fin » non tenu** :
~12 mm sous 3,00 mm par net, cols de 0,96–1,07 mm (`OUT_B`/`OUT_C`) entre les pastilles de
`C306`–`C309`, 0,90 mm (`OUT_A`) près de la via `BST_B`. Longueurs A 31,7 / B 58,3 / C 63,7 /
**D 101,2 mm** : `L303`/`L304` sont en `x` = 240, à 45 mm de `U6`. Vias `BST` déplacées en
(280 ; 169,1) et (280 ; 180,9), `BST_A`/`BST_D` à `y` 182,5 / 167,5.

## Dernière tâche validée

**F1.2-b2 = PASS, nombre de vias révisé par la mesure.** `BST_B`/`BST_C` plongent sur `B.Cu`
avec **2 vias par net**, remontées en (274,6 ; 177,275) et (274,6 ; 172,725) vers `C307`/`C308`
pad 1 ; `BST_A`/`BST_D` contournent sur `F.Cu`, sans via. Tout en 0,35 mm. (Sorties et
contournements déplacés depuis par F1.2-b3, voir Tâche actuelle.)

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
  `L301` (`OUT_ion.
- **Sérigraphie à traiter en bloc** ; **masses et rails auxiliaires longs au plan de F1.4**.

## Blocage actif

Aucun. **Arbitrage rendu : reprise du placement de l'étage de sortie (F1.2-c).** Cadre E1.5
conservé — grille 2 × 2 des tores imposée par le contour (courtyard 29,10 × 13,10 mm, bande `y`
183–225) ; ouverts : pas, position et affectation des tores, position de `C306`–`C309`.
Repères : `L301` (276 ; 195), `L302` (276 ; 212), `L303` (240 ; 195), `L304` (240 ; 212) ;
`C321`–`C324` en `y` 229, `x` 268/246/224/202 ; borniers `J301` (246 ; 241), `J302` (224 ; 241).

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `199511c0266fd228e2d84d4a10a298d9` ; `.kicad_dru` porte les
  deux règles internes aux boîtiers à pas fin
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md`, section **F1.2-b** — mesure d'échappement et courant (IPC-2221)

## NEXT ACTION

**F1.2-c1 — étude au fichier, sans écriture, du replacement des tores et des bootstraps** :
grille resserrée, affectations planaires tore ↔ `OUT`, bootstraps en miroir hors du chemin
`OUT`. Validation : longueurs `OUT` → self par option, planarité avec `OUT_x_F` et borniers,
écart A/B et C/D chiffré, option retenue écrite dans `docs/architecture.md`.
