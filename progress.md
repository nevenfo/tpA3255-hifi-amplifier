# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **105 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage. Aucune `clearance`, aucun `courtyards_overlap`, aucun
`lib_footprint_mismatch`.

## Tâche actuelle

**E1.5 — revue du placement.** Tranches « symétrie mesurée » et « corrective » closes. Reste
**le retour des courants et les masses**, en lecture seule.

## Dernière tâche validée

**E1.5, tranche « corriger les deux écarts gratuits » = PASS.** Cinq déplacements par IPC, tous
en `y` : `C308` → 173,5 ; `C309` → 169,6 ; `C311` → 171,7 ; `R105` → 163,5 ; `R106` → 166,5. Les
bootstraps somment désormais à 350, miroir exact du brochage de `U6` ; `VMID` est à mi-distance
des deux voies. Détail : `docs/architecture.md`, section « E1.5 — tranche corrective ».

Validation :

- Les 5 positions cibles **au micron**, `x` et orientation inchangés.
- **124 empreintes**, les **119 autres intactes** — `L301`–`L304`, `C306`, `C307` compris ;
  `pad_prop_heatsink` de `U1` toujours présent.
- DRC : **105 violations, toutes de sérigraphie** (86 `silk_overlap`, 19 `silk_over_copper`),
  254 non-connectés, `schematic_parity` = 0, **aucune `clearance` ni `courtyards_overlap`** —
  ce qui valide l'entraxe le plus serré, 3,0 mm entre `C307` et `C308`.

**Avant elle** : E1.5 « symétrie mesurée » ; E1.4 (`698e925`), placement clos ; E1.10 ; E1.3 en
quatre tranches ; E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

## Décisions actives

Placement figé, budgets thermiques et pilotage KiCad sont dans `docs/architecture.md` et
`docs/kicad-operations.md`. Restent ici celles qui gouvernent la prochaine action :

- **Le brochage de `U6` est un miroir autour de `y` = 175, pas un escalier.** Les paires réelles
  de l'étage de sortie sont **A↔D et B↔C**, jamais A↔C/B↔D. Toute vérification de symétrie de
  sortie qui apparie par numéro est fausse.
- **Un écart géométrique exact peut être électriquement faux.** Les selfs sont à (0 ; +17) exact,
  mais composée avec un brochage en miroir cette translation donne des trajets de sortie de
  71,7 / 91,9 / 97,9 / 91,4 mm. Elles ne se replaceront pas pour autant : courtyard
  29,10 × 13,10 mm, grille 2 × 2 imposée. **Contrainte pour F1** : apparier au routage, **par
  paire de pont** — A avec B, C avec D.
- **La symétrie de la carte n'est pas un vecteur unique** : (0 ; +40) en analogique, (0 ; +17)
  aux selfs, (−22 ; 0) aux sorties. Aucun contrôle par translation globale.
- **Zones interdites par la barre de liaison** : `x` ∈ [280, 291], `y` ∈ [160, 190] ; et
  `x` ∈ [280, 300], `y` ∈ [169, 181]. Contour 200 × 150, `(100,100)`–`(300,250)`.
- **Un déplacement IPC peut perdre une propriété de pastille** : `U1` a perdu ainsi son
  `pad_prop_heatsink`, seule occurrence de la carte. **À vérifier après tout déplacement d'un
  circuit intégré à pad exposé.**
- **Une contrainte d'encombrement se valide au DRC, pas au calcul de courtyard.**
- **Un DRC de comparaison se lance dans le répertoire du projet**, sinon la baseline est fausse.
  Binaire : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de
  configuration s'éditent directement**.

## Blocage actif

Aucun. Le blocage GUI de la session précédente est **levé et l'issue vérifiée** : l'éditeur de
PCB étant ouvert, l'IPC a exécuté le lot de cinq déplacements sans aucun refus. Le blocage
`SendInput` n'atteint pas l'IPC et ne survit pas à la session. Voir `docs/kicad-operations.md`,
section « Quand l'automatisation GUI est morte ».

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes **toutes placées**, MD5
  `a0a35c1636a56b0deb7c7934052899ff` ; `.kicad_dru` porte la règle d'isolation intra-empreinte
  de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad, à lire avant toute manipulation de la carte
- `docs/architecture.md` — placement figé, symétrie mesurée, tranche corrective, `NEEDS_DATA`,
  budgets thermiques

## NEXT ACTION

**E1.5, tranche « retour des courants et masses »**, en **lecture seule** — mesure au fichier,
aucune écriture, `.kicad_pcb` identique au bit près à la fin (`cmp` et `git diff` vides).

Trois questions à trancher, chacune par une mesure et non par une intention :

1. **Les boucles de commutation de `U6`** — pour chacun des quatre demi-ponts, la surface de la
   boucle `PVDD` → pastille → découplage → `GND`. Ce sont elles qui rayonnent ; `C311`
   vient d'être déplacé, sa boucle est donc à remesurer.
2. **L'entrelacement des condensateurs de sortie**, question laissée ouverte par la tranche
   précédente : `C321` (A) 268, `C323` (C) 246, `C322` (B) 224, `C324` (D) 202. La symétrie L/R
   est exacte, mais les retours `OUT_A_F`/`OUT_B_F` et `OUT_C_F`/`OUT_D_F` **se croisent dans la
   même bande de carte**. Établir si le croisement est réel au niveau des retours ou seulement
   apparent dans l'ordre en `x`, et s'il se corrige au placement ou au routage.
3. **La séparation des masses** — où passe la frontière entre masse de puissance et masse
   analogique, et si le placement actuel la rend réalisable en un point unique.

Livrable : une section dans `docs/architecture.md`, chaque conclusion adossée à une mesure
citée, et les contraintes de routage qui en découlent inscrites explicitement pour F1.
