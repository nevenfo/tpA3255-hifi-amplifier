# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **106 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage. Aucune `clearance`, aucun `courtyards_overlap`, aucun
`lib_footprint_mismatch`.

## Tâche actuelle

**E1 est close** — les dix tâches de l'unité sont validées. La suite est **F1, le routage**, mais
deux décisions utilisateur la conditionnent (voir plus bas).

## Dernière tâche validée

**E1.5, tranche « clearances et manufacturabilité » = PASS**, en lecture seule, **et E1.5 est
close** — donc **E1 aussi**. Les 106 violations de sérigraphie **n'en sont que deux** :
`C312`/`C313` et `C314`/`C315` produisent 73 des 87 `silk_overlap`, et 15 des 19
`silk_over_copper` sont des champs de référence sur pastille. Aucune ne touche le cuivre.
Contour : marge minimale **1,78 mm**. Isolation : les seules valeurs serrées sont **internes aux
boîtiers** (0,200 mm sur `U1`), l'inter-composants remontant à **0,950 mm**. Manufacturabilité :
**124 empreintes toutes sur `F.Cu`**, une seule face de pose, 100 CMS et 24 traversants.

Validation :

- Les quatre contrôles reçoivent une réponse **mesurée**, le compte de sérigraphie étant
  **décomposé par origine** et non simplement constaté.
- `.kicad_pcb` **identique au bit près** — MD5 `eb5273ac4252b64fd97f4bd90b058fd9`, `git diff` vide.
- **Réparation de continuité** : `E1.3` était déclarée close par sa dernière tranche mais restait
  décochée, avec un libellé annonçant « 48 des 124 empreintes » que E1.4 contredisait. Corrigée.

**Avant elle** : les cinq tranches de E1.5 ; E1.4 (`698e925`) ; E1.10 ; E1.3 en quatre tranches ;
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
- **L'IPC écrit une rotation de 270° sous la forme `-90`.** Équivalent modulo 360, mais un
  contrôle qui chercherait littéralement `270` conclurait à tort à un échec.
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
  `eb5273ac4252b64fd97f4bd90b058fd9` ; `.kicad_dru` porte la règle d'isolation intra-empreinte
  de E1.9 ; `.kicad_sym` les symboles locaux
- **`docs/kicad-operations.md`** — pilotage KiCad, à lire avant toute manipulation de la carte
- `docs/architecture.md` — placement figé, symétrie mesurée, tranche corrective, retour des
  courants et masses, `NEEDS_DATA`, budgets thermiques

## NEXT ACTION

**F1.1 — router les boucles de commutation, l'alimentation Class-D et les découplages.** C'est la
seule tranche de F1 qu'**aucune des deux décisions en attente n'affecte** : elle porte sur
`PVDD`, `GND` et les bootstraps autour de `U6`, pas sur les sorties ni sur les entrées
analogiques. Elle peut donc commencer sans arbitrage.

Ordre imposé par la revue : **les quatre boucles de bootstrap d'abord, et sans via** — E1.5 a
retiré le dernier croisement qui en aurait imposé un, ce serait le perdre que de router
autrement. Puis `PVDD`/`GND` entre `C310`/`C311` et les pastilles 29–31 et 36–38 de `U6`, puis la
liaison au bulk.

Valider par `kicad-cli pcb drc --format json` depuis le répertoire du projet : `schematic_parity`
= 0, **aucune `clearance`**, aucun `courtyards_overlap`, et le compte de non-connectés qui
**décroît** du nombre de pastilles effectivement routées — c'est la preuve arithmétique que le
routage a pris, celle qui avait déjà servi en E1.8.

**Avant F1.2 et F1.3**, les deux décisions listées plus haut doivent être tranchées : router les
sorties puis permuter les borniers, ou router l'analogique puis déplacer les condensateurs
d'entrée, serait à refaire deux fois.
