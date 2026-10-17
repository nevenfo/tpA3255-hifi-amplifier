# F1.4-a — stratégie de plans (lecture seule)

## Baseline

- PCB MD5 : `D1C92EA037BDE88A81731B002599E651`.
- `kicad-cli 10.0.6` : DRC 123 violations (`silk_overlap` 93, `silk_over_copper` 30), 78 non-connectés tous `GND`, parité 3 ; ERC 16 violations dans `sheets[0].violations`.
- Stack déclaré : `F.Cu` signal, `In1.Cu` power, `In2.Cu` mixed, `B.Cu` signal. Aucune zone. Ces contrôles sont une baseline, pas une preuve CEM/thermique.

## Mesures

544 segments : `F.Cu` 428 / 1 800,62 mm, `B.Cu` 116 / 2 191,65 mm ; 54 vias traversants. Occupation `B.Cu` : `INPUT_A`–`INPUT_D` 54 / 1 301,01 mm (11 vias), `PVDD` 10 / 211,28 mm (3 vias), `+3V3` 2 / 135,93 mm (2 vias), autres 50 / 543,43 mm. La migration d'une piste vers `In2.Cu` peut garder ses vias : zéro via neuf est une borne basse, à confirmer au DRC/remplissage.

Méthode reproductible : lecture seule de `HifiAmp_TPA3255.kicad_pcb`, extraction des blocs `(segment ...)` et `(via ...)`, regroupement par champ `layer` et `net`, longueur euclidienne `sqrt((x2-x1)^2+(y2-y1)^2)` de chaque segment. Les rapports source sont `reports/F1.4-a-2026-10-17-drc.json` et `reports/F1.4-a-2026-10-17-erc.json`, produits par `kicad-cli pcb drc --schematic-parity --format json` et `kicad-cli sch erc --format json`.

## Conflit E1.5

La fente analogique/puissance vers `x≈142`, à pont unique près de `U6`, est incompatible avec des `INPUT` allant de `x≈127–133` à `x=288,71`, demandés entre deux plans `GND` : la fente détourne leur retour au pont et agrandit la boucle. Un seul net `/GND` ne résout pas ce conflit. Les retours `GND` de `C321`–`C324` doivent aller à `U6` hors aire analogique ; `PVDD`/`C310`/`C311` reste court ; BTL ne va jamais à `GND`.

## Options et recommandation

| Option | Reprise | Verdict |
| --- | --- | --- |
| O1, `B.Cu` signal | 0 | Pas de plan continu ni double référence `INPUT` : rejetée. |
| O2, plans fendus | 116 / 2 191,65 mm vers `In2.Cu`, 0 via neuf minimum | Les `INPUT` traversent les fentes : rejetée. |
| O3, deux plans continus | Même transfert, dont `INPUT` 54 / 1 301,01 mm, `PVDD` 10 / 211,28 mm, `+3V3` 2 / 135,93 mm | Recommandée : `In1.Cu` et `B.Cu` `/GND` continus, `In2.Cu` mixte ; `INPUT` entre deux références, retour local, MKP vers `U6`. |

O3 transforme l'étoile en ségrégation fonctionnelle : aucun retour MKP dans l'aire analogique, aucune route bruitée au-dessus des entrées, couture `GND` à la frontière fonctionnelle et près de `U6`, sans fente sous `INPUT`. Une étoile physique impose de changer l'architecture d'entrée ou d'ajouter une couche.

## Préconditions F1.4-b

`U1` WSON-10 à pad exposé (`172,172`) et `Q301` TO-220 (`184,229`) : **NEEDS_DATA** recommandations thermiques `LM5010ASD`/`IPP330P10NM`, puissance pire cas et règles PCB (via, masque, bouchage, cuivre) avant leur matrice thermique. Aucun via sous `U6`, dont le PowerPAD supérieur va à la barre/dissipateur. Après arbitrage : connectivité, DRC sans `clearance`/`shorting_items`/`track_dangling`/`courtyards_overlap`, parité 3, pas d'îlot, continuité des deux références sous chaque `INPUT`.
