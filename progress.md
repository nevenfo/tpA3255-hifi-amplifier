# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **104 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage et nettoyage. Plus aucun `lib_footprint_mismatch`.

## Tâche actuelle

**E1.5 — revue du placement.** La tranche « symétrie mesurée » est close. Restent les
corrections qu'elle a démontrées, puis le retour des courants et les masses.

## Dernière tâche validée

**E1.5, tranche « symétrie mesurée » = PASS**, en lecture seule. Les 18 paires analogiques sont
à **(0 ; +40 ; 0°) exact** ; les selfs à (0 ; +17) exact ; condensateurs de sortie et borniers à
(−22 ; 0) exact. **Trois écarts établis** — `VMID` décentré, bootstraps hors miroir, pont BTL
déséquilibré en intra-voie. Détail complet : `docs/architecture.md`, section « E1.5 — symétrie
mesurée du placement ».

Validation :

- Couverture complète : 18 paires analogiques, 4 blocs de sortie, aucun apparié omis.
- Chaque écart porte un verdict qui distingue le **délibéré** de l'**erreur**.
- `.kicad_pcb` **identique au bit près** avant et après — `cmp` positif, `git diff` vide.

**Avant elle** : E1.4 (`698e925`), placement des 124 empreintes clos ; E1.10 ; E1.3 en quatre
tranches ; E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

## Décisions actives

Placement figé, budgets thermiques et pilotage KiCad sont dans `docs/architecture.md` et
`docs/kicad-operations.md`. Restent ici celles qui gouvernent la prochaine action :

- **Le brochage de `U6` est un miroir autour de `y` = 175, pas un escalier.** Les paires réelles
  de l'étage de sortie sont **A↔D et B↔C**, jamais A↔C/B↔D comme la numérotation le suggère.
  Toute vérification de symétrie de sortie qui apparie par numéro est fausse.
- **La symétrie de la carte n'est pas un vecteur unique** : (0 ; +40) en analogique, (0 ; +17)
  aux selfs, (−22 ; 0) aux sorties. Aucun contrôle par translation globale.
- **Barre de liaison** : 10 × 60 mm, 600 mm², **pied élargi ≥ 1200 mm² non négociable**. M3 en
  (285, 163) et (285, 187). **Zones interdites** : `x` ∈ [280, 291], `y` ∈ [160, 190] ; et
  `x` ∈ [280, 300], `y` ∈ [169, 181].
- **Contour arrêté à 200 × 150 mm**, `(100,100)`–`(300,250)` ; resserrable après E1.5.
- **Un déplacement IPC peut perdre une propriété de pastille.** `U1` a perdu
  `pad_prop_heatsink` ainsi — seule occurrence de la carte. Réparation par « Mise à Jour des
  Empreintes à partir des Librairies », options texte décochées. **À vérifier après tout
  déplacement de circuit intégré porteur d'un pad exposé.**
- **Un DRC de comparaison se lance dans le répertoire du projet**, sinon la baseline est fausse.
  Binaire : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
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
- **`docs/kicad-operations.md`** — pilotage KiCad, à lire avant toute manipulation de la carte
- `docs/architecture.md` — placement figé, symétrie mesurée, `NEEDS_DATA`, budgets thermiques
- **État KiCad** : fermé, aucun verrou `~*.lck`.

## NEXT ACTION

**E1.5, tranche « corriger les deux écarts gratuits » — écriture de carte, via
`kicad-control`/MCP.** Cinq déplacements, tous en `y`, aucun en `x`, aucune rotation :

1. **Bootstraps en miroir** : `C308` 172 → **173,5** et `C309` 168 → **169,6** ; `C306` (180,4)
   et `C307` (176,5) ne bougent pas. Les sommes deviennent exactement 350, soit le miroir parfait
   autour de `y` = 175. Entraxes 3,9 / 3,0 / 3,9 mm — le plus serré dépasse les ~2,6 mm de
   courtyard d'un 0603 en rotation 90°, **à confirmer par le DRC**, pas par ce calcul.
2. **Découplage `PVDD`** : `C311` 171,9 → **171,7**, ce qui met `C310`/`C311` au miroir exact.
3. **Référence `VMID` recentrée** : `R105` (118 ; 152) → **(118 ; 163,5)** et `R106` (118 ; 155)
   → **(118 ; 166,5)** — entraxe 3 mm conservé, centre porté à `y` = 165, le milieu exact des
   deux voies. Zone libre : seuls `C106` et `C107` occupent `x` 105..135 / `y` 158..178, à
   `x` 126 et 132.

Ne pas toucher `L301`–`L304` : leur déséquilibre intra-voie est réel mais **non gratuit**, et il
a sa propre tranche.

Valider par relecture du fichier — les 5 positions cibles au micron, **les 119 autres empreintes
intactes**, 124 empreintes, `pad_prop_heatsink` toujours à 1 — puis par `kicad-cli pcb drc
--format json` lancé depuis le répertoire du projet : `schematic_parity` reste 0, 254
non-connectés, **aucune `clearance`, aucun `courtyards_overlap`**, `lib_footprint_mismatch`
toujours 0. Le total peut varier de quelques unités en sérigraphie seulement.
