# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **105 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage. Aucune `clearance`, aucun `courtyards_overlap`, aucun
`lib_footprint_mismatch`.

## Tâche actuelle

**E1.5 — revue du placement.** Tranches « symétrie mesurée », « corrective » et « retour des
courants et masses » closes. Reste **la rotation de `C308`/`C309`**, puis clearances et
manufacturabilité.

## Dernière tâche validée

**E1.5, tranche « retour des courants et masses » = PASS**, en lecture seule. Fait dominant :
**un seul net de masse, `/GND`, et aucune zone dessinée** — la séparation sera géométrique,
décidée en F1, avec **35,76 mm** de marge. Trois découvertes : `C308`/`C309` **croisés** (seules
les positions avaient été mises en miroir, pas les orientations) ; **les deux borniers de sortie
croisés** ; **160 mm d'entrées analogiques** dont le filtre est à la mauvaise extrémité. Détail
et contraintes F1 : `docs/architecture.md`.

Validation :

- Les trois questions posées reçoivent une réponse **mesurée**, aucune estimée.
- Chaque croisement est établi par **test d'intersection des segments**, pas par une aire.
- `.kicad_pcb` **identique au bit près** — MD5 `a0a35c1636a56b0deb7c7934052899ff`, `git diff` vide.

**Avant elle** : E1.5 « corrective » et « symétrie mesurée » ; E1.4 (`698e925`) ; E1.10 ; E1.3 ;
E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

## Décisions actives

Placement figé, budgets thermiques et pilotage KiCad sont dans `docs/architecture.md` et
`docs/kicad-operations.md`. Restent ici celles qui gouvernent la prochaine action :

- **Le brochage de `U6` est un miroir autour de `y` = 175.** Il inverse l'ordre des nets d'une
  moitié à l'autre. **Une position mise en miroir ne suffit pas : l'orientation doit l'être
  aussi.** Deux composants appariés qui portent la même rotation sont forcément à l'envers l'un
  des deux.
- **Un croisement se prouve par intersection de segments, jamais par une aire** : sur un
  quadrilatère croisé, la formule du lacet retourne la différence des lobes et désigne le pire
  cas comme le meilleur.
- **Un écart géométrique exact peut être électriquement faux** : les selfs sont à (0 ; +17)
  exact et donnent pourtant quatre trajets de sortie inégaux. Elles ne se replaceront pas —
  **contrainte F1** : apparier au routage, **par paire de pont**, A avec B et C avec D.
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

Aucun.

## Décisions à porter à l'utilisateur

Hors périmètre du placement, aucune ne bloque la prochaine action : **permuter les broches de
`J301`/`J302` au schéma**, et **déplacer ou non `C106`/`C107`/`C206`/`C207` devant les entrées de
`U6`**. Arguments chiffrés dans `docs/architecture.md`.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes **toutes placées**, MD5
  `a0a35c1636a56b0deb7c7934052899ff` ; `.kicad_dru` porte la règle d'isolation intra-empreinte
  de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad, à lire avant toute manipulation de la carte
- `docs/architecture.md` — placement figé, symétrie mesurée, tranche corrective, retour des
  courants et masses, `NEEDS_DATA`, budgets thermiques

## NEXT ACTION

**E1.5, tranche « rotation des bootstraps croisés ».** L'éditeur de PCB étant ouvert, reprendre
via `kicad-control`/MCP. **Deux rotations, aucun déplacement** :

| Réf. | Orientation actuelle | Cible |
| --- | --- | --- |
| `C308` | 90° | **270°** |
| `C309` | 90° | **270°** |

`C306` et `C307` **ne bougent pas** : déjà dans le bon sens, les tourner les croiserait. Les
centres de `C308` (273,5 ; 173,5) et `C309` (273,5 ; 169,6) restent **identiques au micron**.

Valider par relecture — les 2 orientations, les **124 centres inchangés**, `pad_prop_heatsink`
à 1 — puis par le **test d'intersection** rejoué sur les quatre bootstraps : les quatre doivent
ressortir **simples**, aucun croisé. Enfin `kicad-cli pcb drc --format json` depuis le répertoire
du projet : `schematic_parity` = 0, 254 non-connectés, **aucune `clearance`, aucun
`courtyards_overlap`**, `lib_footprint_mismatch` = 0. Seul le compte de sérigraphie peut varier.
