# Revue de manufacturabilité G1.4 (lecture seule)

Base : PCB `ea6ea4b3695946c52b2deca67f622fd4`, SCH `9bcba85d947b78af82475f58438dcdcb`, DRU `cb9016bf…`,
DRC `drc-g1.3.json` (HEAD `d6ab9c2`). Méthode : extraction au fichier (pastilles en géométrie réelle, coins
arrondis compris, pistes, vias, texte), distances par géométrie plane, DRC `kicad-cli`. Aucun fabricant n'est
choisi : les seuils « standard » et « avancé » ci-dessous sont des ordres de grandeur courants de fabricants de
prototypes 4 couches, à remplacer en H2 par les capacités publiées du fabricant retenu.

## 1. Paramètres mesurés

| Paramètre | Valeur au fichier | Standard | Avancé | Verdict |
| --- | --- | --- | --- | --- |
| Couches / épaisseur | 4 (`F.Cu`, `In1.Cu` plan, `In2.Cu` mixte, `B.Cu`), `general.thickness` 1,6 mm | 1,6 mm | 0,8–2,4 mm | OK |
| Empilement | **non déclaré** (aucun bloc `stackup`) ; cuivre, prépreg et diélectrique absents | à fournir | à fournir | **À exiger dans les notes** |
| Cuivre ext. / int. | non déclaré ; les calculs IPC-2221 du projet supposent 35 µm partout | ext. 35 µm, int. 17,5 µm | int. 35 µm | **Exiger int. ≥ 35 µm** (voir §5) |
| Largeur de piste mini | 0,20 mm (92 segments : signaux `C_START`, `RESET`, `OC_ADJ`, `CLIP_OTW`, `FAULT`, …) ; 0,25 mm (220) | 0,127 mm | 0,075–0,1 mm | OK, marge ×1,6 |
| Isolation mini hors boîtiers fins | 0,225 mm (piste–piste `RESET`/`FAULT`, piste–pastille `C_START`/`R303.1`) | 0,127 mm | 0,075–0,1 mm | OK |
| Isolation dans les boîtiers fins | 0,15 mm (pastille–pastille `U8`), 0,16 mm (piste `OUT_A` – `U6.41`), 0,167–0,175 mm (`U6`, `U1`) : exceptions DRU assumées | 0,127 mm | 0,075–0,1 mm | OK, marge ×1,2 en standard |
| Isolation vias | ≥ 0,25 mm (via–via, via–piste) ; trou–trou mini 0,326 mm (vias GND `a4061ec3`/`f46ffa1e`) | 0,25 mm | 0,2 mm | OK |
| Perçage mini | via 0,3 mm (121) et 0,4 mm (13 vias `INPUT`) ; pastilles traversantes 0,8 mm mini ; 2 trous non métallisés 3,2 mm (`H1`, `H2`) | 0,3 mm | 0,2 mm | OK, rapport d'aspect 5,3:1 |
| Via mini | 0,6/0,3 (121) ; 0,8/0,4 (13) | 0,6/0,3 | 0,45/0,2 | OK |
| Anneau mini, vias | 0,15 mm (0,6/0,3) | 0,125 mm | 0,1 mm | OK, marge 0,025 mm en standard |
| Anneau mini, pastilles traversantes | 0,35 mm (`J2`, `J3`), 0,375 mm (`J4`) | 0,15 mm | 0,1 mm | OK |
| Cuivre autour d'un via dans une pastille | `C206.2` : 0,15 mm (x) et 0,175 mm (y) autour du via de 0,6 mm | — | — | Voir §2 |
| Cuivre – bord de carte | 0,525 mm (`INPUT_A`, `In2.Cu`, jusqu'au trait de contour, soit 0,50 mm à la découpe) ; plans à 0,50 mm | 0,3 mm | 0,2 mm | OK |
| Pas fin mini | `U8` 0,5 mm (pastilles 0,35 mm), `U6` 0,635 mm (0,4 mm, intervalle 0,235 mm), `U1` 0,8 mm, `U7` 0,65 mm | 0,5 mm | 0,4 mm | OK |
| Pont de masque entre ouvertures (marge 0,05 mm) | `U8` **0,05 mm** ; `U1` 0,10 mm (pin–EP) ; `U6` 0,135 mm ; autres ≥ 0,2 mm | 0,1 mm | 0,05–0,075 mm | **`U8` limite avancé** (§5, action 1) |
| Masque | `pad_to_mask_clearance` 0,05 mm, vias tentés recto-verso, pas de `covering`/`plugging`/`capping`/`filling` | — | — | Voir §2 |
| Sérigraphie, texte | 1,0 mm de haut, trait 0,15 mm (125 références visibles, 3 en `B.SilkS`) | 0,8 mm / 0,15 mm | 0,6 mm / 0,1 mm | OK |
| Sérigraphie, traits | 0,12 mm (2 852 traits d'empreintes), 0,1 mm (4) | 0,15 mm | 0,1 mm | Sous le standard : sans effet électrique, le fabricant peut épaissir ou supprimer |
| Contour | rectangle 200 × 150 mm (100–300, 100–250), trait 0,05 mm | — | — | OK |

## 2. B1 classé : vias dans une pastille ou en bord de pastille

13 vias recouvrent une pastille (mesure géométrique du cercle de cuivre du via contre la pastille). Le comptage « 9 » de
G1.2 omettait `C10.2` ; avec `C319` il y en a 10 entièrement dedans et 3 au bord. Tous ont un trou de 0,3 mm ouvert
(`filling` = non) ; la bonne mesure du risque est le cuivre restant autour du trou et la présence de pâte.

| # | Via (UUID, net, position) | Pastille | Position relative | Risque d'aspiration | Traitement exigé | Déplacement local qui supprime le besoin |
| --- | --- | --- | --- | --- | --- | --- |
| 1–6 | `f2fb0306`, `c52804f0`, `a7b9f0b0`, `f8efdd8f`, `d95f5f11`, `017c7bb9`, `/GND`, x 171,5/172,5, y 171/172/173 | `U1` EP 2,6 × 3,0 mm | dedans, à 0,5–0,8 mm du bord | **Élevé** : 4 vias (y = 171 et 173) sont entièrement sous une ouverture de pâte, 2 (y = 172) en recouvrent 20 % de deux ; pâte = 65 % de l'EP | IPC-4761 type VII (remplis + capés) pour les 6, ou à défaut vias bouchés côté `B.Cu` **et** ouvertures de pâte écartées de ≥ 0,1 mm des vias | Aucun : les vias thermiques sont requis dans l'EP |
| 7 | `6bd825bf` `/GND` (295,775 ; 168,63) | `C206.2` 0,9 × 0,95 mm | dedans, centre à 0,45 mm du bord | **Élevé** : 0,15–0,175 mm de cuivre autour du via, pastille presque entièrement trouée | IPC-4761 VII, ou déplacement | **0,96 mm** vers (295,17 ; 167,88), avec piste de 0,2 mm vers la pastille |
| 8 | `7a0c33be` `/GND` (293,95 ; 166,0) | `C303.2` 1,0 × 1,45 mm | dedans, centre à 0,50 mm du bord | Moyen : 0,35 mm de cuivre autour du trou | IPC-4761 VII, ou bouchage `B.Cu` | Aucun emplacement libre à ≤ 2,5 mm |
| 9 | `a4061ec3` `/GND` (290,512 ; 174,148) | `C319.2` 1,45 × 1,0 mm | dedans, centre à 0,35 mm du bord | Moyen | IPC-4761 VII, ou déplacement | **0,8 mm** vers (289,71 ; 174,15) ; ou décaler `C319` de +0,8 mm en x (ci-dessous) |
| 10 | `b25ba5ed` `/+3V3` (182,475 ; 138,3) | `C10.2` 1,15 × 2,7 mm | dedans, centre à 0,575 mm du bord | Faible à moyen : grande pastille, 0,4 mm de cuivre autour du trou | Bouchage `B.Cu` suffisant, ou déplacement | **1,0 mm** vers (183,48 ; 138,3) |
| 11 | `a7e953f7` `/INPUT_D` (290,6 ; 168,425) | `C207.1` 0,9 × 0,95 mm | **bord** : centre dedans à 0,26 mm du bord, cercle de cuivre déborde de 0,04 mm, trou entièrement sur la pastille | Moyen | Bouchage `B.Cu`, ou IPC-4761 VII | Aucun emplacement libre à ≤ 2,5 mm |
| 12 | `f46ffa1e` `/GND` (290,062 ; 174,583) | `C319.2` (via de la liaison `U6.13`) | **bord** : centre à 0,066 mm hors pastille, 30 % de l'annulaire recouvre la pastille | Moyen : le trou touche le bord de la pastille | Bouchage `B.Cu`, ou déplacement | Décaler `C319` (ci-dessous) |
| 13 | `1c3d68e6` `/RESET` (171,6 ; 130,8) | `U7.3` 1,02 × 0,35 mm | **bord** : centre à 0,024 mm du bord, 35 % de l'annulaire sur la pastille, trou à cheval | **Élevé** : trou à moitié sur la pastille, ouverture de masque et pâte | Déplacement recommandé (bouchage seul insuffisant sur une pastille de 0,35 mm) | **1,24 mm** vers (170,4 ; 131,1) |

- **Décaler `C319` de +0,8 mm en x** (centre (291,65 ; 175,25)) sort les deux vias GND (n° 9 et 12) de la pastille GND ;
  les dégagements restent ≥ 0,25 mm. La distance pastille DVDD ↔ `U6.11` passe de 2,31 à environ 3,1 mm. À reprendre :
  `9ea34f03`, la piste verticale `DVDD` de G1.3-a et la liaison GND.
- Déplacements : calcul avec dégagement ≥ 0,25 mm sur tout le cuivre et ≥ 0,1 mm entre l'annulaire et la pastille ; le DRC
  reste juge.
- Tenting recto-verso déjà actif : le via est fermé par le masque, mais un trou de 0,3 mm sous une pastille brasée
  aspire la brasure avant que la pâte ne refonde. Seul IPC-4761 VII (ou l'absence de via dans la pastille) l'évite.
- Vias hors pastille qui touchent une ouverture de masque : aucun autre cas (mesure sur les 134 vias).

## 3. Sérigraphie : 89 `silk_overlap` + 25 `silk_over_copper`

Aucun objet de sérigraphie n'est hors carte. Tous sont des avertissements.

| Catégorie | Nombre | Détail | Correction proposée | Automatisable au fichier |
| --- | --- | --- | --- | --- |
| A1. Référence d'un électrolytique contre le contour du voisin | 73 | `C313` sur le contour de `C312` (37) et `C315` sur celui de `C314` (36) : condensateurs ø 18 mm, références à moins de 1 mm du contour voisin | Déplacer le champ référence de `C313` et `C315` à l'extérieur de la paire, ou réduire à 0,8 mm | Oui (champ `Reference`, `(at)` relatif ; DRC ensuite) |
| A2. Référence contre le contour d'un voisin | 12 | `C39`/`C4`, `C4`/`C3`, `C302`/`C106`, `C206`/`C303`, `C207`/`C303`, `C319`/`C318`, `R202`/`C202`, `C83`/`C84`, `C327`/`U1` (×3), `C38`/`C5` | Déplacer le champ référence au côté libre | Oui, cas par cas |
| A3. Référence contre référence | 4 | `H2`/`U6`, `C206`/`C207`, `C106`/`C107`, `C328`/`U1` | Déplacer un des deux champs | Oui |
| B1. Référence sur pastille voisine | 21 | `C302`, `C206`, `C319`, `C207`, `R301`, `Q302`, `C39`, `R202` (2), `R102` (2), `C83` (2), `C327`/`U1` (3), `C328` (2), `C38` (2) | Même correction que A2 ; le fabricant retire déjà la sérigraphie sur les ouvertures de masque | Oui |
| B2. Repère de broche 1 de `L301`–`L304` sur pastille | 4 | Cercle de l'empreinte sur la pastille `OUT_*` | Aucune : repère d'empreinte, la sérigraphie sera rognée | Non nécessaire |

Effet de fabrication nul pour les 114 entrées ; seul le gain de lisibilité justifie A1–A3 et B1. Les 4 de B2 sont à
accepter. Par rapport à G1.1, 3 entrées sont nouvelles et 5 ont disparu (champs de `C9`, `C13`, `C318`, `C319`, `C326`, `C329` déplacés avec leurs composants).

## 4. Écarts résiduels assumés

| Écart | Valeur | Justification | Reprise éventuelle |
| --- | --- | --- | --- |
| C1 : respect partiel de la note T1 (TI SLASEA8A, Fig. 38–40) | Barre `F.Cu` 2,6 mm + 10 vias PVDD vers `In2.Cu` ; arrivée `F.Cu` directe sur les 6 broches mais courant partagé par les vias (≈ 0,46 A par via, ≈ 0,4 °C) | Le tronc `F.Cu` de l'option (a) exige le déport de `+12V` (`ada74fbd`) et un canal de 2,0 mm sous le boîtier | Option (a) si TI/SLOU441 l'exige |
| C3 : `C318` à 5,13 mm de `U6.14` | Piste ≈ 5,5 mm (avant : 32,4 mm) | Zone saturée par `C_START`, `INPUT_C` et les vias GND | Valeur et distance : SLOU441 non lu |
| Découpes des plans sous `U6` | 2 fentes ≈ 2,9 × 2,5 mm dans `In1.Cu` et `B.Cu`, pont de plan ≈ 1 mm ; plans −0,13 cm² chacun, d'un seul tenant | Imposées par les 8 vias PVDD et le via GND (283 ; 175) | Réduire à 6 vias PVDD si la fente gêne |
| Écart `INPUT_A`–`INPUT_B` / `INPUT_C`–`INPUT_D` | 324,7 / 325,2 mm contre 365,1 / 365,0 mm : 40,3 mm entre paires, 0,45 et 0,1 mm intra-paire | Écart de voie, λ ≫ 40 mm à 20 kHz (A5) | Aucune |
| Dog-bones de `U6` à la limite | Via (290,06 ; 180,0) à 0,265 mm de `INPUT_A` et 0,01 mm du courtyard ; `U6.3/.4`, `U6.33/.34` ; `GND` de `U6.33/.34` à 0,512 mm de `PVDD` | Pas de 0,635 mm | Marges au-dessus des capacités standard |
| Isolation de 0,15–0,16 mm dans `U8`, `U6`, `U1` | Exceptions DRU prouvées sélectives | Géométrie des boîtiers | Aucune |
| Données `NEEDS_DATA` | TPA3255 SLOU441 (valeurs et distance `AVDD`/`DVDD`) ; TLV1117-33 (condensateur d'entrée de `U3`, `C9` seul 100 nF) ; capacités du fabricant (via-in-pad rempli, pont de masque mini, cuivre interne) | Pas de PDF dans le dépôt | À lever en H2 |
| Valeurs TI relevées sur pages `document-viewer` | SLASEA8A, SNVS376F (LM5010), SNVS452G (LM5069) : aucun PDF décodé | — | Recontrôler sur PDF avant H2 |
| `Tj` de `Q301` | ≈ 102 °C estimé à 4,95 A, Ta = 50 °C (F1.4-b4, 479,5 cm²·µm de drain) contre 175 °C max | Estimation à vide | **À mesurer** en H2 (sonde thermique, charge nominale) |
| `U6` : pas de voie thermique propre | PowerPAD sur le dessus, `RθJC(top)` 0,36 °C/W, TJ max 150 °C | Dissipateur | Mesure H2 |

## 5. Notes de fabrication proposées (à reprendre dans les sorties H2)

```
CARTE HifiAmp_TPA3255 — 4 couches, 200 x 150 mm, épaisseur finie 1,6 mm.
Empilement : F.Cu / In1.Cu (plan GND) / In2.Cu (signaux + barres PVDD) / B.Cu.
Cuivre fini : couches externes 35 um (1 oz), couches INTERNES 35 um (1 oz) ; le
cuivre interne de 17,5 um n'est pas accepté (barres PVDD In2.Cu 7 mm dimensionnées
pour 35 um : à 17,5 um l'échauffement à 4,6 A passerait de 9 °C à environ 27 °C).
Prépreg / diélectrique : ≥ 0,1 mm entre couches adjacentes (tension de service 48 V,
crête 58 V). Fournir l'empilement retenu avant lancement.
Matériau : FR-4, Tg ≥ 150 °C (cuivre fort, refusion sans plomb).
Finition : ENIG recommandée (pas fin 0,5 mm sur U8, via-in-pad) ; HASL sans plomb accepté
hors pastilles U8/U1.
Masque : vert (ou noir mat), vias tentés recto-verso, marge d'ouverture 0,05 mm
(0,025 mm autour des pastilles de U8), pont de masque minimum 0,1 mm.
Perçage fini : vias 0,3 mm (0,4 mm pour 13 vias INPUT), trous traversants ≥ 0,8 mm,
tolérance +/-0,1 mm ; 2 trous de fixation 3,2 mm non métallisés.
Anneau minimum 0,15 mm sur les vias (0,6/0,3). Piste/isolement minimums 0,2 mm /
0,225 mm hors boîtiers fins ; 0,15 mm dans les cours de U1, U6, U8.
Cuivre à 0,5 mm du bord de carte. Contour sans V-cut ni languette.
VIAS DANS LES PASTILLES : 6 vias thermiques de U1 (EP) + C206.2, C303.2, C319.2, C10.2 :
IPC-4761 type VII (remplis résine, capés et métallisés en surface) ; autres vias
tentés (type IV-b) recto-verso. Ne pas appliquer de pâte sur les trous non remplis.
Contrôle : test électrique 100 % (net-list IPC-356), contrôle d'impédance non requis.
Sérigraphie : texte 1,0 mm / trait 0,15 mm ; traits d'empreinte 0,12 mm ; retirer la
sérigraphie sur les ouvertures de masque.
Marquage : date et code fabricant admis sur B.SilkS hors zone de brasure.
```

## Actions recommandées avant H2

Autonomes au fichier (chacune se prouve au DRC, parité 3, `(units`, `pad_prop_heatsink`) :

1. `U8` : marge de masque locale −0,025 mm sur les 10 pastilles (pont 0,05 → 0,10 mm).
2. `C206.2` : déplacer le via de 0,96 mm vers (295,17 ; 167,88) avec piste de liaison.
3. `U7.3` : déplacer le via `/RESET` de 1,24 mm vers (170,4 ; 131,1).
4. `C10.2` : déplacer le via `/+3V3` de 1,0 mm vers (183,48 ; 138,3).
5. Sérigraphie A1–A3 et B1 : déplacer les champs référence (110 entrées sur 114, sauf B2).

Demandent une décision :

6. Décaler `C319` de +0,8 mm (supprime 2 vias, pastille↔broche 2,31 → 3,1 mm).
7. `C303.2` et `C207.1` : aucun emplacement libre ; choisir bouchage/IPC-4761 VII ou accepter le risque.
8. Les 6 vias de l'EP de `U1` : IPC-4761 VII, ou écarter les ouvertures de pâte des vias (perte de surface de pâte).
9. Choix du fabricant (capacités via-in-pad rempli, pont de masque, cuivre interne 35 µm, empilement).
10. Option (a) de C1 si la note T1 doit être respectée à la lettre.

## Complément : épaisseur de cuivre et référence TI

Le Second Brain (page « routage et écarts référence TI ») le rappelle : TI recommande du FR-4 en cuivre de 70 µm (2 oz), et son EVM est une carte 2 couches en 2 oz. Ce projet est dimensionné en 35 µm sur toutes les couches. Avant H2, il faut faire revalider auprès du fabricant retenu l'épaisseur, le cuivre (externe et interne), le courant et la température. Une option en 70 µm externe abaisserait de moitié environ l'échauffement des pistes externes.
