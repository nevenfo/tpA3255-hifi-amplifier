# Revue de conception G1.2 (lecture seule)

Base : PCB `70e315fa…`, DRU `cb9016bf…`, DRC G1.1 (`drc-g1.1.json`), HEAD `5eaf144`.
Méthode : extraction au fichier (pads, pistes, vias, zones remplies) et formule IPC-2221 du projet
(35 µm, échauffement cible 10 °C ; couche interne k = 0,024, externe k = 0,048). Aucune fiche
technique n'est dans le dépôt (aucun PDF composant) : les valeurs fabricant citées viennent de
`docs/architecture.md`, `docs/plane-strategy-f1.4-a.md`, `docs/protection-48v.md`.

## Synthèse

| Classe | Nombre | Constats |
| --- | --- | --- |
| Bloquant (avant sorties de fabrication H2) | 1 | B1 |
| À corriger | 6 | C1 à C6 |
| Accepté | 11 | A1 à A11 |

Aucun bloquant pour la connectivité : DRC 0 clearance / court-circuit / non-connecté, parité 3.

## Bloquant

### B1 — Vias dans les pads et EP de U1 : remplissage non spécifié
- **Preuve :** 9 vias sont dans un pad ou sous une ouverture de pâte : `C303.2` (293,95 ; 166), `C206.2` (295,775 ; 168,63) et les 6 vias thermiques de `U1` (x 171,5/172,5, y 171/172/173), plus aucun autre. Le `setup` du PCB : `tenting` oui, `covering/plugging/capping/filling` = non. Les trous de 0,3 mm restent ouverts dans le cuivre de soudure.
- **Risque :** aspiration de la brasure (EP de `U1` sous 4 ouvertures de pâte ; pad 0,9 × 0,94 mm de `C206.2` avec seulement 0,15 à 0,17 mm de cuivre autour du via).
- **Correction proposée :** exiger au fabricant des vias bouchés/remplis et métallisés en surface (IPC-4761 type VII) pour ces 9 vias, ou à défaut déplacer `C303.2`/`C206.2` en dog-bone (voir A6). À consigner dans les notes de fabrication et, si possible, dans les propriétés de via (`filling`/`capping`). Aucune donnée fabricant disponible : `NEEDS_DATA` capacité de via-in-pad (min. pad, remplissage) du fabricant retenu.

## À corriger

### C1 — Deux vias seulement pour tout le courant `PVDD` de `U6`
- **Preuve :** les 6 pastilles `PVDD` de `U6` (29-31, 36-38) rejoignent le plan `In2.Cu` par 2 vias 0,6/0,3 en (282,6 ; 172,78) et (282,6 ; 177,22), via des pistes `F.Cu` de 1,1 mm. Courant continu 4,6 A (`protection-48v.md`) : ≈ 2,3 A par via. Capacité IPC-2221 d'un via 0,6/0,3 (cuivre 25 µm) : ≈ 1,9 A à 10 °C, soit ≈ 15 °C d'échauffement estimé à 2,3 A, et 3,8 A au total pour 4,6 A.
- **Correction proposée :** ajouter 2 vias `PVDD` par côté (4 au total, ≥ 0,8 mm d'entraxe, 1,15 A chacun) ; la règle DRU exempte déjà `PVDD` dans le courtyard de `U6`. À valider par recherche de place (sorties `OUT_x` à 0,5 mm).

### C2 — Découplage `VCC` de `U1` loin de la broche
- **Preuve :** `C13` est à 7,1 mm de `U1.9` en ligne droite, 13,2 mm de piste (F.Cu 1,0 + In2.Cu 7,9 + 2 vias) pour un condensateur de découplage de `VCC` du LM5010. À comparer à `C327` (1,45 mm) sur `BUCK_VIN`.
- **Correction proposée :** rapprocher `C13` (≤ 3 mm de `U1.9`) ou ajouter un second 0,1 µF près de la broche. `NEEDS_DATA` : SNVS376F, valeur et distance recommandées pour le condensateur `VCC`/`GND` du LM5010.

### C3 — Découplages `AVDD` et `DVDD` de `U6` éloignés
- **Preuve :** `C318.1` à 13,8 mm de `U6.14` (32,4 mm de piste) ; `C319.1` à 18,0 mm de `U6.11` (38,3 mm de piste). Ces longueurs, assumées en F1.3 « faute de via », sont à réexaminer depuis que `In2.Cu` est libre.
- **Correction proposée :** rerouter les deux liaisons en visant ≤ 20 mm de piste, ou déplacer le condensateur. `NEEDS_DATA` : TPA3255 (SLASEA8A / SLASEG), section Layout : distance maximale des condensateurs `AVDD`, `DVDD`, `GVDD`, `VBG` et exigences de retour `GND`.

### C4 — Entrée `+12V` de `U3` : condensateur à 12,7 mm
- **Preuve :** pour `U3` (SOT-223, 3,3 V), le plus proche condensateur sur `+12V` est `C9.2` à 12,7 mm de `U3.3` ; le condensateur de sortie `C10.2` est à 5,3 mm de `U3.2`/`U3.4`.
- **Correction proposée :** ajouter ou déplacer un condensateur d'entrée à ≤ 5 mm de `U3.3`. `NEEDS_DATA` : fiche du régulateur de `U3`.

### C5 — Entrée du LM5069 (`U8`) : condensateur à 14,7 mm
- **Preuve :** pour `U8.2` (`PVDD_PROT`), le plus proche condensateur est `C326.1` à 14,7 mm (23,5 mm de piste) ; pour `U8.9` (`PVDD`) c'est `C39.2` à 14,9 mm (31,5 mm de piste).
- **Correction proposée :** vérifier le besoin dans la fiche, sinon rapprocher. `NEEDS_DATA` : LM5069, section Layout : condensateur local sur `VIN` et sur la sortie.

### C6 — Piste `PVDD` de 5 mm en `In2.Cu` : échauffement > 10 °C
- **Preuve :** segments `1fd0727d` et `2d1d025d` (5 mm, 38 mm au total) à 4,6 A : ≈ 15 °C ; `e74fd2fd` (4 mm, 10 mm) à 4,6 A : ≈ 22 °C, mais il ne dessert que `C317`. La barre de 7 mm (`f1745b96`) reste sous 10 °C.
- **Correction proposée :** élargir `1fd0727d` à ≥ 6,4 mm (valeur IPC-2221 interne à 10 °C), ou doubler en `F.Cu`.

## Accepté

- **A1 — Courts-circuits :** DRC `clearance` 0, `shorting_items` 0, `hole_clearance` 0, `solder_mask_bridge` 0, parité 3. Marges serrées mais conformes : via (290,06 ; 180,0) à 0,265 mm de `INPUT_A` (0,25 requis) et 0,01 mm du courtyard de `U6` ; pistes `GND` de `U6.33/.34` à 0,512 mm de `PVDD` (0,5 requis). 0,265 mm reste largement au-dessus d'une capacité de fabrication courante.
- **A2 — Masses :** non-connectés 0 ; `GND` en plans continus `In1.Cu` (290,4 cm²) et `B.Cu` (290,2 cm²), un seul polygone chacun, aucun îlot ; chaque groupe de pads `GND` a un via ou un pad traversant. Les 8 broches `GND` de `U6` sont reliées (dog-bones, ou liaison à un via existant).
- **A3 — Alimentation `OUT` :** `OUT_x_F` à 3,0 mm partout (5 A : 2,77 mm requis). Sections étroites `OUT_x` (0,4 à 0,8 mm sur 0,6 à 2,5 mm) en sortie de `U6` : déclarées en F1.2-c2 (`x` ≥ 277,2), imposées par le pas de 0,635 mm. Aucun via sur un net `OUT_*`. À surveiller à la mesure de température en H2.
- **A4 — Longueurs `OUT` :** A 29,13 / B 28,92 mm (écart 0,21), C 68,27 / D 71,89 mm (écart 3,62), inchangées depuis F1.2-c2.
- **A5 — `INPUT_A` à `INPUT_D` :** totaux 324,7 / 325,2 / 365,1 / 365,0 mm (vias à 1,6 mm). Écart intra-paire 0,45 (A–B) et 0,1 mm (C–D). L'écart de 40,3 mm entre A–B et C–D est un écart de voie, sans effet audio (λ ≫ 40 mm à 20 kHz).
- **A6 — Dog-bones de `U6` à la limite :** voir A1. Les 6 pads qui n'avaient pas de place ont reçu : `U6.3/.4` (via à la limite des clearances de classe), `U6.33/.34` (liaison `F.Cu` d'environ 6 mm vers les vias existants de `U6`), `C303.2`/`C206.2` (via dans le pad, voir B1).
- **A7 — `INPUT_B`/`INPUT_C` près du bord droit :** x = 298,8 / 298,25 mm (1,2 / 1,75 mm du bord, 0,7 / 1,25 mm du bord des plans) sur 76 et 69 mm. Les plans `In1.Cu` et `B.Cu` sont remplis sous tout le trajet `In2.Cu` des `INPUT` (7,41 mm de ligne médiane sans cuivre au total, soit 0,6 % : isolement des vias propres aux `INPUT`). Optionnel : couture `GND` près du bord.
- **A8 — Thermique :**
  - `U1` : budget de 2,08 W admissibles à Ta = 50 °C pour TJ 125 °C (RθJA 36 °C/W) contre 0,25 W de conduction à 1 A ; les pertes de commutation ne sont pas chiffrées par la fiche (F1.4-a). 6 vias thermiques dans l'EP.
  - `Q301` : Tj ≈ 102 °C à 4,95 A / Ta = 50 °C (estimation F1.4-b4, 6,54 + 7,16 cm² de drain, 479,5 cm²·µm) contre 175 °C maximum.
  - `U6` : le PowerPAD est sur le dessus ; le PCB n'a pas de voie thermique propre. Le budget (22,4 W, `RθJC(top)` 0,36 °C/W, `REQ-THERM-1`, dissipateur) est dans `docs/architecture.md`. La fiche n'étant pas dans le dépôt, ces valeurs ne sont pas re-vérifiables ici : `NEEDS_DATA` TPA3255 `RθJC(top)`, TJ max et courbes de puissance (SLASEA8A § 7.4).
- **A9 — Sérigraphie (90 `silk_overlap` + 26 `silk_over_copper`, tous en avertissement) :**
  - 82 `silk_overlap` : champ référence d'un condensateur contre un segment de contour, dont 36 à 37 occurrences chacun pour `C312`–`C315` (électrolytiques ø 18 mm très proches).
  - 2 `silk_overlap` : référence `R*` ou `U*` contre un segment.
  - 6 `silk_overlap` : référence contre référence, ou contre un cercle de contour (`C312`, `C314`, `H*`/`U*`).
  - 22 `silk_over_copper` : références (16 `C*`, 5 `R*`, 1 `U1`) sur pad.
  - 4 `silk_over_copper` : cercle de repère de broche 1 des inductances `L301`–`L304` sur leur pad.
  - Correction possible : déplacer ou masquer les champs référence (sans toucher aux composants) ; les repères des inductances sont de l'empreinte et sans effet de fabrication. Les 116 entrées sont cosmétiques.
- **A10 — Contrôles ignorés du projet :** `missing_courtyard` (les 126 empreintes ont un courtyard), `footprint_filters_mismatch`, `footprint_type_mismatch`, `track_not_centered_on_via`, `tuning_profile_track_geometries` : sans effet vérifié.
- **A11 — Parité 3 :** `U6` pin 45 (PowerPAD sur le dessus), `H1` et `H2` (trous de fixation).

## Points `NEEDS_DATA` à lever pour clore G1.2

1. TPA3255 (SLASEA8A / SLASEG), tables « Pin Functions » et « Layout » : distance maximale des condensateurs `AVDD`/`DVDD`/`GVDD`/`VBG`, exigence de liaison directe pour les broches `GND` (12, 13, 25, 26, 33, 34, 41), fonction des pins 3 et 4 (`M1`, `M2`), `RθJC(top)` et TJ max.
2. LM5010 (SNVS376F) § Layout : valeur et distance du condensateur `VCC`.
3. LM5069 : condensateurs d'entrée/sortie recommandés et leur distance.
4. Fabricant : capacité de via-in-pad rempli et capé, clearance minimale (marge de 0,265 mm sur `U6`), épaisseur de cuivre interne.

## Données fabricant relevées (2026-10-18)

Sources : pages `document-viewer` de ti.com. Aucun PDF n'a pu être décodé, donc chaque valeur est à recontrôler sur le PDF avant H2.

- **TPA3255, SLASEA8A (oct. 2016) :**
  - Broches `GND` : 12, 13, 25, 26, 33, 34, 41 **et 42** (§6).
  - PowerPAD : « connect to grounded heat sink » (§6).
  - §11.1 : plan de masse contigu depuis les broches `GND`, « for as many of the ground pins as possible », et zone de masse locale pleine sous le boîtier.
  - Pins 3/4 : M1 (LSB) et M2 (MSB) ; M2 = 0, M1 = 0 donne le Stereo BTL (Table 1). Une ligne « Mono BTL » est ambiguë, à vérifier sur le PDF.
  - Découplages (§6) :
    - `GVDD_AB` (1) et `GVDD_CD` (22) : 0,1 µF chacun ;
    - `VDD` (2) : 10 µF + 0,1 µF ;
    - `VBG` (20) : 1 µF ;
    - `AVDD` (14) et `DVDD` (11) : pas de valeur ; le datasheet renvoie à l'EVM, SLOU441 (non lu) ;
    - aucune distance chiffrée.
  - `PVDD` (§10.2.1.2.2) : découplage 1 µF X7R ≥ 100 V par pont complet, « as close as possible », et réservoir de 1 000 µF / 80 V.
  - **Notes T1/T2 des Figures 38–40 (§11.2) :**
    - T1 : `PVDD` en pistes larges sur la couche supérieure, « direct connection to the pins, not through vias », et « no vias or traces should block the current path » ;
    - T2 : les X7R de découplage proche se placent sous le dissipateur.
  - Thermique : TJ max 150 °C (§7.1), TJ recommandée 0–125 °C (§7.3), RθJC(top) 0,36 °C/W (§7.4).
- **LM5010A, SNVS376F :**
  - `VCC` : ≥ 0,47 µF « as close as possible to the pins » (citation venant d'un résumé, à recontrôler) ;
  - boucle C1 → VIN-SW → L1 → C2 → C1 minimale, et masse C2 → C1 courte, sans via (§10.1).
- **LM5069, SNVS452G :**
  - bypass d'entrée près de `Rsns` plutôt que de la broche VIN (§10.1.1) ;
  - 1 µF céramique vers la masse près du drain du MOSFET (§9) ;
  - TVS à clamp < 30 V au-dessus de VIN, exemple SMBJ30A (§8.2.1.2.7).

### Conséquences sur les constats

- **C1/C6 requalifiés.** L'arrivée `PVDD` de `U6` passe par la barre de 7 mm en `In2.Cu` (transfert F1.4-b1) et par 2 vias, ce qui contredit la note T1.
  - Correction visée : `PVDD` en `F.Cu` jusqu'aux broches `PVDD` de `U6`, ou à défaut une barre `F.Cu` doublée et cousue.
  - Faisabilité à chiffrer en G1.3 : la couche `F.Cu` porte les `OUT` et les `BST` autour de `U6`.
- **C3** : vérifier la valeur et la distance d'`AVDD`/`DVDD` sur SLOU441 ; en attendant, rapprocher les condensateurs.
- **C5** : la règle du LM5069 porte sur la proximité avec `Rsns` et le drain, pas avec VIN. Recalculer `C326`/`C39` selon ce critère.
- **Broche 42** : vérifier qu'elle est reliée à `/GND` et dotée d'un chemin vers les plans.
- **TJ max 150 °C** pour `U6`, à reprendre dans la revue thermique (A-thermique).
