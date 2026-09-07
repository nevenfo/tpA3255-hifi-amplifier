# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **104 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage. Plus aucun `lib_footprint_mismatch`.

## Tâche actuelle

**E1.5 — revue du placement.** Tranche « symétrie mesurée » close, question des selfs tranchée.
Reste la tranche corrective, **bloquée**, puis le retour des courants et les masses.

## Dernière tâche validée

**E1.5, tranche « symétrie mesurée » = PASS**, en lecture seule. Les 18 paires analogiques sont
à **(0 ; +40 ; 0°) exact** ; selfs à (0 ; +17) ; sorties et borniers à (−22 ; 0). **Trois écarts
établis** — `VMID` décentré, bootstraps hors miroir, translation des selfs contredisant le
miroir du brochage. Détail : `docs/architecture.md`, section « E1.5 — symétrie mesurée ».

Validation :

- Couverture complète : 18 paires analogiques, 4 blocs de sortie, aucun apparié omis.
- Chaque écart distingue le **délibéré** de l'**erreur**.
- `.kicad_pcb` **identique au bit près** — `cmp` positif, `git diff` vide.

**Avant elle** : E1.4 (`698e925`), placement clos ; E1.10 ; E1.3 en quatre tranches ; E1.9,
E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

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
- **Un DRC de comparaison se lance dans le répertoire du projet**, sinon la baseline est fausse.
  Binaire : `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
- Toute édition schéma/PCB/librairie passe par `kicad-control`/MCP ; **les fichiers de
  configuration s'éditent directement**.

## Blocage actif

**L'automatisation GUI est morte dans cette session, donc l'IPC KiCad est inatteignable** — et
`move_component` n'existe qu'en IPC.

- **Symptôme** : `SetForegroundWindow`, `ui_click` et le clavier échouent — `False` avec
  `GetLastWin32Error = 0`, ou `SendInput was blocked`.
- **Cause probable** : blocage `SendInput` au niveau de la **session entière**, contexte
  `RDP-Tcp#0`.
- **Faits exclus** : pas l'élévation de KiCad — la même activation échoue sur une fenêtre
  Firefox non élevée. Pas le MCP, pas le lancement. Deux canaux ont échoué : coordonnées, puis
  UIA sémantique.
- **Prochaine tentative** : **l'utilisateur ouvre l'éditeur de PCB lui-même** depuis le
  gestionnaire de projet, déjà ouvert. L'IPC n'est pas affecté et reprend ensuite. Détail dans
  `docs/kicad-operations.md`, section « Quand l'automatisation GUI est morte ».
- **État préservé** : `.kicad_pcb` intact, MD5 `fab35539c5b160087172270bafa3e7bd`. Aucun
  éditeur ouvert, donc aucune ambiguïté IPC.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — 124 empreintes **toutes placées** ; `.kicad_dru` porte la règle
  d'isolation intra-empreinte de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad, à lire avant toute manipulation de la carte
- `docs/architecture.md` — placement figé, symétrie mesurée, `NEEDS_DATA`, budgets thermiques

## NEXT ACTION

**E1.5, tranche « corriger les deux écarts gratuits ».** Dès l'éditeur de PCB ouvert, reprendre
via `kicad-control`/MCP. Cinq déplacements, tous en `y`, aucun en `x`, aucune rotation :

| Réf. | `y` actuel | `y` cible |
| --- | --- | --- |
| `C308` | 172 | **173,5** |
| `C309` | 168 | **169,6** |
| `C311` | 171,9 | **171,7** |
| `R105` | 152 | **163,5** |
| `R106` | 155 | **166,5** |

`C308`/`C309` portent les sommes A+D et B+C à 350 exactement, soit le miroir des bootstraps
autour de `y` = 175 ; `C311` fait de même pour `PVDD` ; `R105`/`R106` recentrent `VMID` à
`y` = 165, entraxe 3 mm conservé. `C306` (180,4) et `C307` (176,5) ne bougent pas. **Ne pas
toucher `L301`–`L304`.** Zone `VMID` libre : seuls `C106`/`C107` occupent `x` 105..135 /
`y` 158..178.

**Point à surveiller** : l'entraxe le plus serré devient **3,0 mm** entre `C307` et `C308`, pour
des 0603 en rotation 90°. À confirmer par l'absence de `courtyards_overlap` au DRC, **pas** par
le calcul de courtyard.

Valider par relecture — les 5 positions au micron, **les 119 autres intactes**, 124 empreintes,
`pad_prop_heatsink` à 1 — puis par `kicad-cli pcb drc --format json` depuis le répertoire du
projet : `schematic_parity` = 0, 254 non-connectés, **aucune `clearance`, aucun
`courtyards_overlap`**, `lib_footprint_mismatch` = 0. Seul le compte de sérigraphie peut varier.
