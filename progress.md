# PROGRESS

## Phase actuelle

**Phase E — PCB 4 couches.** `GATE C2 = PASS`, baseline ERC 16 violations / 0 erreur. DRC
courant : **106 violations, 254 non-connectés, `schematic_parity` = 0** — **toutes de la
sérigraphie**, normales avant routage. Aucune `clearance`, aucun `courtyards_overlap`, aucun
`lib_footprint_mismatch`.

## Tâche actuelle

**E1.5 — revue du placement.** Quatre tranches closes. Restent **clearances et
manufacturabilité**.

## Dernière tâche validée

**E1.5, tranche « rotation des bootstraps croisés » = PASS.** `C308` et `C309` passent de 90° à
**270°**, sans déplacement : centres identiques au micron, sommes à 350 préservées. `C306`/`C307`
inchangés — les tourner les croiserait.

Validation :

- **Test d'intersection rejoué** sur les six condensateurs voisins de `U6` — quatre bootstraps
  plus `C310`/`C311` — **zéro croisement**.
- 124 empreintes, les **122 autres intactes** ; `C306`/`C307` toujours à 90° ;
  `pad_prop_heatsink` de `U1` présent.
- DRC : **106 violations, toutes de sérigraphie**, 254 non-connectés, `schematic_parity` = 0,
  **aucune `clearance` ni `courtyards_overlap`**.

**Avant elle** : E1.5 « retour des courants et masses », « corrective » et « symétrie mesurée » ;
E1.4 (`698e925`) ; E1.10 ; E1.3 ; E1.9, E1.2, E1.8, E1.7, E1.6, E1.1 = PASS.

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

**E1.5, tranche « clearances et manufacturabilité »**, en **lecture seule** — mesure au fichier,
aucune écriture, `.kicad_pcb` identique au bit près à la fin (`git diff` vide).

Quatre contrôles, chacun conclu par une mesure :

1. **Les 106 violations de sérigraphie**, seul poste non vide du DRC. Établir combien relèvent de
   références chevauchant une pastille — donc illisibles après fabrication — et combien sont sans
   conséquence. C'est le dernier poste qui pourrait masquer un vrai défaut avant routage.
2. **Isolation des tensions élevées** : le rail `PVDD` est à 48 V et les sorties commutent à cette
   amplitude. Mesurer les distances les plus courtes entre un net de puissance et un net de
   signal, et les confronter à la règle d'isolation portée par `.kicad_dru`.
3. **Distance au contour** de toutes les empreintes — contour 200 × 150, `(100,100)`–`(300,250)` —
   et respect des deux zones interdites par la barre de liaison.
4. **Manufacturabilité** : empreintes trop proches pour la pose automatique, et cohérence des
   côtés de pose.

Livrable : une section dans `docs/architecture.md`, chaque conclusion adossée à une mesure citée,
et ce qui doit passer en contrainte F1 inscrit explicitement. **E1.5 se clôt avec cette tranche**
si aucun défaut bloquant n'apparaît.
