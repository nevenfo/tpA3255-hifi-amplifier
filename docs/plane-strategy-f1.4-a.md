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

### `U1` — `LM5010ASD/NOPB`, boîtier `DPR0010A`

Source : TI SNVS376F (rév. mai 2016), §6.3, §6.5, §6.6, Eq. 2, §10.1, dessins `DPR0010A` 4218856/B.

- **L'empreinte du PCB est fausse.** Le `DPR0010A` mesure 4 × 4 mm, au pas de 0,8 mm, avec un pad exposé de 2,6 × 3 mm. Le PCB porte `Package_SON:WSON-10-1EP_2.5x2.5mm_P0.5mm_EP1.2x2mm`, pastilles à ± 1,2125 mm au pas de 0,5 mm. Le composant ne serait pas soudable. Correction : F1.7, préalable à F1.4-b.
- Pas de NEEDS_DATA bloquant pour `U1`.
  - Données thermiques : RθJA 36 °C/W (métrique JEDEC, SPRA953), RθJC(bot) 3 °C/W, ψJB 13,5 °C/W ; TJ max recommandée 125 °C ; RDS(on) max 0,8 Ω (TJ ≤ 125 °C).
  - Fréquence, Eq. 2, à VIN ≈ 47,5 V (`PVDD` après `D3`), `R_ON` 182 kΩ, VOUT 15 V : FS ≈ 673 kHz.
  - Budget : (125 − 50) / 36 = 2,08 W admissibles à Ta = 50 °C. La conduction au courant maximal du composant (1 A, D = 0,316) vaut 1² × 0,8 × 0,316 = 0,25 W. La datasheet ne chiffre pas les pertes de commutation (méthode PIN − POUT, §10.1), mais elles devraient dépasser 1,8 W pour atteindre la limite : la marge est large.
- Recommandations de layout :
  - Pad exposé soudé au plan `GND`, vias vers les plans de l'autre face, « autant que possible » (§10.1). L'exemple TI utilise des vias Ø 0,2 mm.
  - Les règles du projet imposent 0,6/0,3 mm : matrice 2 × 3 au pas de 1 mm dans le pad, vers `In1.Cu` et `B.Cu`, bouchées côté `B.Cu` pour limiter la remontée de soudure.
  - Le retour `C2`→`C1` (`COUT`→`CIN`) et `SGND`/`RTN` restent directs sans via (§10.1).

### `Q301` — `IPP330P10NM`, TO-220

Source : Infineon Final Data Sheet rév. 2.0, 2021-05-10, Tables 2–4, Diagram 9.

- Données : RthJC 0,5 °C/W ; RthJA 40 °C/W avec 6 cm² de cuivre 70 µm sur une couche (FR4 40 × 40 × 1,5 mm, vertical, air calme) et 62 °C/W sur empreinte minimale ; RDS(on) max 33 mΩ à 25 °C ; Tj max 175 °C ; RDS(on) normalisé ≈ 1,38 à 78 °C et ≈ 1,58 à 100 °C (Diagram 9).
- Calcul itératif `Tj = Ta + I² × 33 mΩ × k(Tj) × RthJA` :
  - 4,6 A, Ta = 40 °C, 40 °C/W : Tj ≈ 79 °C, P ≈ 0,96 W.
  - 4,95 A (4 Ω), Ta = 50 °C, 40 °C/W : Tj ≈ 101 °C, P ≈ 1,28 W.
  - Même cas, empreinte minimale (62 °C/W) : Tj ≈ 160 °C, marge insuffisante. **Le cuivre de drain est donc nécessaire.**
- Cuivre retenu, conforme à O3 (`B.Cu` reste `/GND` continu) :
  - Zone `PVDD_FUSED` `x` 175..200, `y` 230..248,5 sur `F.Cu` et `In2.Cu` (≈ 9 cm² chacune), cousue par vias. 2 × 9 cm² × 35 µm ≥ 6 cm² × 70 µm en conductance de feuille.
  - Équivalence estimée et non prouvée (couche interne, carte horizontale) : à confirmer par mesure de température.
  - Les `INPUT` (`y` ≤ 198,2) passent à plus de 30 mm.
- Aucun via sous `U6`, dont le PowerPAD supérieur va à la barre/dissipateur.

### Validation après application

Connectivité, DRC sans `clearance`/`shorting_items`/`track_dangling`/`courtyards_overlap`, parité 3, pas d'îlot, continuité des deux références sous chaque `INPUT`.
