# Bloc de protection d'entrée 48 V — données sourcées et analyse

Document de conception pour `F301` (fusible), `D301` (TVS) et `Q301` (anti-inversion), plus la limitation d'appel de courant décidée en Phase D.

## Contraintes d'entrée

| Grandeur | Valeur | Origine |
|---|---|---|
| Rail nominal | 48 VDC, source capable de 10 A transitoires | `docs/architecture.md` |
| PVDD recommandé TPA3255 | 18 – 53,5 V | datasheet TI |
| **PVDD maximum absolu TPA3255** | **65 V** | datasheet TI |
| Bulk total en aval | ≈ 15 400 µF (4 × 1500 µF/63 V + 2 × 4700 µF/80 V) | `docs/architecture.md` |
| Courant continu estimé | ≈ 4,6 A (200 W de sortie, rendement 90 %, à 48 V) | calcul |

## Résultat majeur : aucune TVS passive ne satisfait la contrainte

La fenêtre utile est bornée par le rail en bas et par le maximum absolu en haut. Le facteur de clamp exigé serait :

`V_C / V_RWM < 65 / 48 = 1,354`

Or les TVS silicium à avalanche présentent un facteur de clamp de **1,3 à 1,6**, quelles que soient la tension nominale, la puissance de crête ou la famille — y compris les séries automobiles conçues pour le load dump 48 V. 1,354 se situe à la borne basse extrême de la technologie.

Relevés de datasheet :

| P/N | Fabricant | `V_RWM` | `V_C` max | `I_PP` | Verdict |
|---|---|---|---|---|---|
| `SMCJ48A` | Littelfuse, 1500 W | 48 V | **77,4 V** | 19,4 A | dépasse 65 V |
| `SLD8S48A` | Littelfuse, load dump 7 kW | 48 V | **77,4 V** | 89,7 A | dépasse 65 V malgré 7 kW |
| `1.5KE51A` | Littelfuse/Diotec, 1500 W | 43,6 V | 70,1 V | 21,7 A | `V_RWM` déjà sous 48 V, conduirait en service |
| `P6KE51A` | 600 W | 43,6 V | 70,1 V | 8,9 A | idem, et clamp toujours > 65 V |

Piège de nommage écarté : `SM30T35CAY`, `SMC30J30CA` et `SMC3K30CAHM3-57` annoncent un clamp à 48,4 V, mais leur `V_RWM` réel vaut **30 V** (`V_BR` min 33,3 V). Sur un rail 48 V elles conduiraient en permanence. À ne pas retenir.

**Conclusion : une TVS seule sur ce rail protège contre les transitoires rapides mais ne borne pas la tension sous le maximum absolu du TPA3255.** Une surtension soutenue détruit le circuit malgré sa présence. La protection contre la surtension doit être active : OVP à MOSFET, crowbar à thyristor, ou contrôleur hot-swap intégrant la fonction.

## Anti-inversion de polarité

État constaté au schéma : `Q301` porte le symbole `Transistor_FET:Q_NMOS_GSD`, c'est-à-dire un **N-MOS**, et non le P-MOS que suppose un montage série sur le rail positif. Sa grille auto-polarisée expose le `V_GS` à 48 V contre un maximum typique de ±20 V.

**Topologie constatée par inspection** : grille et drain étaient court-circuités sur `PVDD_FUSED` (sortie du fusible `F301`), source sur `PVDD` (rail aval). Le transistor est donc **en série sur le rail positif**, et non sur le retour de masse. Un N-MOS ne pouvant pas conduire côté haut sans pompe de charge, le montage était faux dans son principe, et pas seulement mal polarisé.

**Correction appliquée** : passage à un **P-MOS** (`Transistor_FET:Q_PMOS_GSD`), grille séparée du drain sur un net dédié `Q301_GATE`, tirée vers `GND` par `R305` (100 kΩ) et clampée par la Zener `D302` (15 V).

### Sens de montage du P-MOS — piège à ne pas reproduire

Ce montage se câble **à l'envers d'un usage d'interrupteur**, et une première correction s'y est trompée. Seule la diode de structure décide :

- Sur un P-MOS, l'**anode** de la diode de structure est le **drain**, sa cathode la source.
- Pour protéger, cette diode doit conduire dans le sens du courant **normal**, de `PVDD_FUSED` vers `PVDD`. Son anode — donc le **drain** — doit être **en amont**.
- **Câblage correct : drain sur `PVDD_FUSED`, source sur `PVDD`.**
- Avec l'orientation inverse (source en amont), en inversion l'anode se retrouve côté charge à ≈ 0 V et la cathode côté entrée à −48 V : la diode devient passante, le courant traverse la charge en sens inverse et **la protection ne joue pas**.

La Zener `D302` clampe le `V_GS` : sa cathode doit donc être sur la **source** (`PVDD`), jamais sur le drain. En fonctionnement normal la source monte à ≈ 47,3 V, la grille est tirée vers `GND` par `R305`, le `V_GS` est clampé à ≈ 15 V et le courant de clamp vaut ≈ 33 V / 100 kΩ ≈ 330 µA, soit ≈ 5 mW dans la Zener.

**Ce défaut est invisible à l'ERC** : la netlist est parfaitement valide, seule la physique du composant est en cause. Il illustre pourquoi un gate ERC ne vaut jamais validation électronique.

Dans les deux cas, le clamp de grille retenu est une **Zener de 15 V** avec une **résistance de grille de 100 kΩ**, qui limite le courant Zener à environ 330 µA sous 48 V (≈ 5 mW) tout en gardant une décharge de grille rapide en cas d'inversion. La fourchette de 9 à 15 V pour le `V_GS` vient de `AND90146` ; elle n'est pas spécifique à 48 V.

- Candidat `IRF9540NS` (Vishay/Infineon, D2PAK/TO-263) : `V_DS` = -100 V, `I_D` continu = -23 A, `P_tot` = 3,8 W.
  `NEEDS_DATA` : `R_DS(on)` non recoupé — 117 mΩ et « < 55 mΩ » à `V_GS` = -10 V trouvés dans deux sources secondaires, le PDF officiel n'ayant pas pu être lu. À confirmer dans le tableau *Electrical Characteristics*.

## Limitation de l'appel de courant

Charger 15 400 µF sous 48 V est le point dur. Pendant la rampe, le MOSFET série dissipe `V_DS(t) × I_D(t)`.

Un simple réseau RC de démarrage progressif sur la grille échoue typiquement par **sortie de l'aire de sécurité (SOA)** du MOSFET, pas par dépassement de courant : un boîtier D2PAK à `P_tot` = 3,8 W sans dissipation adaptée n'encaisse pas une rampe longue.

**Levé en B2.5** : la vérification chiffrée de la SOA a été faite sur la courbe constructeur de quatre candidats. Voir la section finale de ce document.

Contrôleurs dédiés relevés, plage 48 V :

| P/N | Fonction | Note |
|---|---|---|
| `LM5066` / `LM5066H` | hot-swap 10–80 V, limitation de courant programmable, UV/OV, **surveillance active de la SOA du MOSFET** | répond directement au risque identifié |
| `LM5069` | hot-swap haute tension, limitation de courant, UV/OV | plus simple |
| `LM5060` | seuil et temporisation de défaut seulement, **sans limitation de courant active** | insuffisant ici |

## Fusible

Le point d'attention est le **pouvoir de coupure en continu** : la plupart des cartouches génériques ne sont calibrées qu'en alternatif, ou en continu à tension bien plus basse (32 V typique automobile).

- `Schurter OMF 63`, montage en surface 7,4 × 3,1 mm : calibré **63 VAC / 63 VDC**, gamme 0,063 à 10 A, pouvoir de coupure 50 A à 63 VDC, `I²t` jusqu'à 54 A²s à 10 A. Compatible du besoin (≈ 4,6 A continus).
- `Littelfuse BF1 58V`, MIDI vissé : calibré **58 VDC**, mais gamme 30 à 200 A — surdimensionné, le calibre minimal dépasse déjà largement le besoin.

**Levé en B2.7** : le format traversant classique reste sans candidat, mais un fusible CMS `Schurter UMT-H` 12,5 A calibré 125 VDC répond au besoin. Voir la section finale de ce document.

## Réserves

- Disponibilité en stock non vérifiée pour aucune référence citée.
- Aucune valeur de Zener ni de résistance de grille nommément recommandée pour 48 V n'a été trouvée : les valeurs ci-dessus sont extrapolées de notes génériques.

## Sources

- Littelfuse SMCJ : https://www.littelfuse.com/assetdocs/tvs-diodes-smcj-datasheet?assetguid=37388813-0d6d-4329-969b-1aa8b7614ac1
- Séries 1.5KE et P6KE (Yageo) : https://yageogroup.com/content/datasheet/asset/file/1_5KE_1 — https://yageogroup.com/content/datasheet/asset/file/P6KE_1
- Vishay IRF9540 : https://www.vishay.com/docs/91078/91078.pdf
- ON Semi AND90146, *MOSFET Selection for Reverse Polarity Protection* : https://www.onsemi.com/download/application-notes/pdf/and90146-d.pdf
- TI LM5066 : https://www.ti.com/lit/gpn/LM5066 — LM5069 : https://www.ti.com/product/LM5069 — LM5060 : https://www.ti.com/product/LM5060
- Schurter OMF 63 : https://www.schurter.com/en/datasheet/OMF_63
- Littelfuse BF1 58V : https://www.littelfuse.com/assetdocs/littelfuse-datasheet-142-bf1-58v?assetguid=837e42d2-ca5a-4d2e-8438-0342b74c753a

---

# Dimensionnement de l'étage LM5069 (B2.4)

Source unique : datasheet TI **`SNVS452G`**, révision de janvier 2020, lue directement (`https://www.ti.com/lit/ds/symlink/lm5069.pdf`). Les numéros d'équation ci-dessous sont ceux de ce document. Aucune valeur n'est reprise d'une source secondaire.

Contrôle de cohérence effectué avant application : l'équation 9 reproduit exactement les 14,90 kΩ de l'exemple TI (équation 10), et le recoupement indépendant du paramètre `PWRLIM-1` (`SENSE-OUT` = 48 V, `R_PWR` = 150 kΩ → 25 mV typique) donne 303 W contre 300 W par la voie directe. Les équations sont donc utilisées correctement.

## Variante retenue

**`LM5069-2`**, à redémarrage automatique. La variante `LM5069-1` se verrouille définitivement après défaut et n'est réarmable que par cyclage de l'alimentation : inacceptable pour un appareil audio grand public.

## Paramètres électriques utilisés (tableau *Electrical Characteristics*)

| Paramètre | Min | Typ | Max |
|---|---|---|---|
| `VCL` seuil de limitation de courant (`VIN`-`SENSE`) | 48,5 mV | 55 mV | 61,5 mV |
| `VCB` seuil de disjoncteur | 80 mV | 105 mV | 130 mV |
| `tCB` temps de réponse du disjoncteur | — | 0,44 µs | 1,2 µs |
| `UVLOTH` | 2,45 V | 2,5 V | 2,55 V |
| `OVLOTH` | — | 2,5 V | 2,6 V |
| courant d'hystérésis `UVLO`/`OVLO` | 12 µA | 21 µA | 30 µA |
| `VTMRH` seuil haut du temporisateur | 3,76 V | 4 V | 4,16 V |
| courant de détection de défaut | 51 µA | 85 µA | 120 µA |

## Seuils de sous-tension et de surtension

Configuration *Option A*, trois résistances (figure 30), équations 21 à 24 puis relecture par 28 à 33.

Cibles : `V_UVH` = 40 V, `V_UVL` = 36 V, `V_OVH` = 56 V.

**Valeurs retenues, série E96 à 1 % : `R1` = 191 kΩ, `R2` = 5,11 kΩ, `R3` = 9,09 kΩ.**

| Seuil | Valeur obtenue |
|---|---|
| `V_UVH` (mise en conduction) | 40,1 V |
| `V_UVL` (coupure basse) | 36,1 V |
| `V_OVH` (**coupure haute**) | **56,4 V** |
| `V_OVL` (reprise) | 52,3 V |
| hystérésis UV / OV | 4,0 V / 4,1 V |

Vérification de la marge, seule qui compte ici : en cumulant le seuil interne `OVLOTH` à son maximum de 2,6 V et les résistances à 1 %, `V_OVH` pire cas atteint **59,9 V**, soit **5,1 V (8 %) sous le maximum absolu de 65 V** du TPA3255. La reprise à 52,3 V reste au-dessus d'une alimentation 48 V à +5 % (50,4 V), donc pas de battement en fonctionnement normal.

## Limitation de courant

Équation 1, dimensionnée sur `V_CL` **minimal** pour garantir la conduction à pleine charge :

`R_SNS ≥ 48,5 mV / 10 A = 4,85 mΩ` → valeur normalisée immédiatement inférieure retenue : **`R_SNS` = 4 mΩ**.

| Fonction | Min | Typ | Max |
|---|---|---|---|
| Limitation de courant | 12,1 A | 13,75 A | 15,4 A |
| Disjoncteur (`VCB`) | 20,0 A | 26,3 A | 32,5 A |

La conduction est donc garantie jusqu'à 12,1 A, ce qui couvre le pire cas de consommation en charge 4 Ω (≈ 9,3 A) sans risque de coupure en pleine musique.

## Limitation de puissance et temporisateur

- Plancher imposé par l'équation 8 (`V_SNS` ≥ 5 mV) : `P_LIM` ≥ 70 W.
- Plafond imposé par la datasheet : `R_PWR` ≤ 150 kΩ, soit `P_LIM` ≤ ≈ 305 W.
- **Retenu : `R_PWR` = 147 kΩ → `P_LIM` = 299 W**, volontairement proche du plafond (voir justification ci-dessous).
- Équation 12, avec `C_OUT` = 15 400 µF : `t_start` = 69 ms typique, **94 ms** en pire cas (`P_LIM` à −24 %, `I_LIM` minimal).
- Équation 13 : `C_TIMER` = 3,9 µF calculé, **arrondi à 4,7 µF** — voir ci-dessous.
- Équation 14, à 4,7 µF nominal : `t_flt` = 147 ms au minimum, 222 ms typique, 383 ms au maximum. Avec la tolérance de ± 10 % du condensateur retenu : **132 ms au minimum, 422 ms au maximum**.

Critère de non-coupure au démarrage, vérifié en croisant les pires cas des deux côtés (et non typique contre typique comme dans l'exemple TI) : `t_flt,min` = 132 ms > `t_start,max` = 94 ms, **marge × 1,41**. À 3,9 µF exact la marge n'aurait valu que × 1,30.

### Pourquoi 4,7 µF et non les 3,9 µF calculés

`C_TIMER` est un condensateur film, et non céramique : à 4 V de seuil sur un diélectrique X7R de 4,7 µF, le déclassement sous tension continue atteint 30 à 50 %, ce qui ferait passer `t_flt,min` **sous** `t_start,max` et provoquerait des coupures au démarrage. Or 3,9 µF est une valeur E24, absente des séries film ; la série WIMA MKS2 au pas de 5 mm suit E6 et s'arrête à 4,7 µF, qui est donc à la fois la valeur disponible et la plus proche par le haut.

Arrondir vers le haut ne dégrade rien et améliore le seul critère contraignant : `t_flt` s'allonge de 20,5 %, ce qui écarte davantage le démarrage de la coupure. En regard, la seule contre-partie est l'allongement de l'exposition SOA de `Q302`, traité à la section suivante et sans effet sur la conclusion.

La tolérance, elle, n'est pas libre. Deux variantes MKS2 4,7 µF sont réellement commercialisées :

| Référence | Tension | Tolérance | `t_flt,min` | Marge au démarrage |
|---|---|---|---|---|
| `MKS2B044701K00KSSD` | 50 V | ± 10 % | 132 ms | **× 1,41 — retenue** |
| `MKS2C044701M00KSSD` | 63 V | ± 20 % | 118 ms | × 1,25 — rejetée |

La variante à ± 20 % retombe **sous** la marge × 1,30 que cette conception s'est fixée. La tension nominale, elle, n'est pas un critère : la broche `TIMER` ne dépasse pas 4 V, et 50 V laissent déjà un facteur 12.

## Exigence SOA imposée au MOSFET — point dur de cette conception

Méthode de la section 9.2.1.2.5 : en défaut, le MOSFET subit `V_DS` = `V_IN,MAX` et `I_D` = `P_LIM` / `V_IN,MAX` pendant `t_flt`.

**Pire cas : 56 V et 5,34 A pendant 422 ms.** Avec la marge de 1,3 × recommandée par TI, le MOSFET doit tenir **6,95 A sous 56 V pendant 422 ms, soit 389 W en mode linéaire**. La puissance exigée ne dépend que de `P_LIM`, donc l'arrondi de `C_TIMER` allonge la durée sans déplacer les 389 W.

C'est une contrainte sévère, et elle est **structurelle, non un défaut de réglage**. La datasheet l'énonce directement en section 9.2.1.1 :

> When charging the output capacitor through the hot swap MOSFET, the FET's total energy dissipation equals the total energy stored in the output capacitor (½CV²). Thus, both the input voltage and output capacitance determine the stress experienced by the MOSFET.

Autrement dit `P_LIM × t_flt` suit l'énergie de charge du bulk, que la limite de puissance soit réglée haut ou bas. Charger 15 400 µF sous 56 V stocke 24,1 J, que le MOSFET doit dissiper à l'identique ; les dispersions du temporisateur (51 à 120 µA) et de la limite de puissance (± 24 %) portent l'exposition pire cas bien au-delà.

C'est pourquoi `P_LIM` est réglé **près du plafond** : à énergie constante, une limite de puissance élevée raccourcit `t_flt`, et la SOA d'un MOSFET s'améliore beaucoup plus vite quand la durée diminue que lorsque le courant diminue.

Effet du bulk sur la durée d'exposition, à `P_LIM` maximal :

| Bulk | `t_start` pire cas | `C_TIMER` | Exposition SOA |
|---|---|---|---|
| 15 400 µF (actuel) | 94 ms | 4,7 µF | 389 W pendant **422 ms** |
| 8 200 µF | 52 ms | 2,2 µF | 377 W pendant **179 ms** |
| 4 700 µF | 29 ms | 1,2 µF | 390 W pendant **98 ms** |

**Levé en B2.5** : `Q302` = `IXTK200N10L2`, dont la SOA garantie à 75 °C vaut 625 W contre 389 W exigés. `R_DS(on)` ≤ 11 mΩ et `R_thJC` = 0,12 °C/W. Méthode, comparaison des candidats et réserves : section finale de ce document.

## Composants figés à ce stade

| Repère | Valeur | Équation |
|---|---|---|
| `R1` / `R2` / `R3` (seuils) | 191 kΩ / 5,11 kΩ / 9,09 kΩ, 1 % | 21 à 24 |
| `R_SNS` | 4 mΩ | 1 |
| `R_PWR` | 147 kΩ | 9 |
| `C_TIMER` | 4,7 µF ± 10 %, film | 13 |
| `U8` | `LM5069-2`, VSSOP-10 | — |
| `Q302` | `IXTK200N10L2`, TO-264 | 19, SOA garantie |
| `F301` | `Schurter UMT-H` 12,5 A, `3403.0285.11` | coordination LM5069 |
| `D301` | `SMDJ58CA`, DO-214AB | plafond 88 V, note (3) |

## Passifs du bloc de protection — boîtiers et références (D1.5)

Ces passifs avaient été ajoutés en B2.3, B2.4 et B2.6 sans jamais recevoir d'empreinte. Le point à retenir est que **ce n'est presque jamais la puissance qui dimensionne leur boîtier, mais la tension** : le nœud `PVDD_PROT` fonctionne à 48 V, monte à 56 V en régime permanent maximal et atteint 93,6 V pendant un écrêtage de `D301`.

| Repère | Valeur | Référence | Empreinte | Ce qui dimensionne le boîtier |
|---|---|---|---|---|
| `R305` | 100 kΩ | — | `R_0805` | **78,6 V** aux bornes en écrêtage. Un 0603, tenu à 50 V, serait violé |
| `R306` | 4 mΩ 1 % | `WSL25124L000FEA` | `R_2512` | 0,95 W pendant ≤ 422 ms ; 1 W admis à 70 °C |
| `R307` | 191 kΩ 1 % | — | `R_0805` | **87,1 V** aux bornes en écrêtage. Même raison que `R305` |
| `R308` | 5,11 kΩ 1 % | — | `R_0603` | 1,4 V seulement : bas du diviseur |
| `R309` | 9,09 kΩ 1 % | — | `R_0603` | 2,5 V seulement : bas du diviseur |
| `R310` | 147 kΩ 1 % | — | `R_0603` | broche `PWR`, quelques volts |
| `C325` | 4,7 µF ± 10 % | `MKS2B044701K00KSSD` | `C_Rect_L7.2mm_W7.2mm_P5.00mm` | film obligatoire, voir plus haut |
| `C326` | 100 nF **250 V** | — | `C_1206` | 93,6 V en écrêtage sur `VIN` de `U8` |
| `D302` | 15 V, 0,5 W | `BZT52C15` | `D_SOD-123` | 12 mW dissipés au pire ; marge 40 × |

Contrôle du diviseur de seuils, qui recoupe le brochage relevé au netlist : `R307` + `R308` + `R309` = 205,2 kΩ, donc à 56 V la prise `UVLO` est à 3,88 V (seuil 2,5 V, franchi) et la prise `OVLO` à 2,48 V, **juste sous** son seuil de 2,5 V. L'ordre `PVDD_PROT` → `R307` → `UVLO` → `R308` → `OVLO` → `R309` → `GND` est donc bien celui qui produit le déclenchement en surtension légèrement au-dessus de 56 V établi plus haut.

### Contrainte de tracé créée par `R306`, à reporter en Phase E

`WSL25124L000FEA` est un shunt à **deux bornes**, pas à quatre. À 4 mΩ, quelques milliohms de cuivre dans les pastilles s'ajoutent directement à la valeur mesurée et faussent le seuil de limitation. Les liaisons vers `VIN` et `SENSE` de `U8` doivent donc partir des **bords intérieurs des deux pastilles**, en pistes fines dédiées, et le courant de puissance entrer et sortir par les bords extérieurs.

Si ce tracé se révèle impraticable au routage, la solution est un shunt Kelvin à quatre bornes de la série **WSK** — `WSK25122L000FEA` existe dans le même boîtier 2512 —, qui rend la mesure indépendante du cuivre. Non retenue à ce stade : la contrainte de tracé est tenable et le composant deux bornes est moins cher.

### Réserve ouverte

L'épaisseur de corps de `C325` est prise à 7,2 mm, valeur maximale de la série MKS2 au pas de 5 mm, ce qui conduit à l'empreinte `W7.2mm`. La cote n'a pas été lue sur le dessin coté WIMA lui-même. **À confirmer en D1.2**, où une erreur se traduirait par un composant qui n'entre pas dans son empreinte.

## Brochage VSSOP-10 (DGS), section 6 de la datasheet

| N° | Nom | N° | Nom |
|---|---|---|---|
| 1 | `SENSE` | 10 | `GATE` |
| 2 | `VIN` | 9 | `OUT` |
| 3 | `UVLO` | 8 | `PGD` |
| 4 | `OVLO` | 7 | `PWR` |
| 5 | `GND` | 6 | `TIMER` |

Raccordements de la figure 27 (*Typical Application Schematic*) : `R_PWR` de `PWR` à la masse, `C_TIMER` de `TIMER` à la masse, diviseur de seuils de `VSYS` à la masse avec les prises sur `UVLO` et `OVLO`, et un condensateur céramique de découplage au plus près de `VIN`.

Le `NEEDS_DATA` sur ce brochage est donc levé.

# Composants figés par sourcing datasheet (B2.5 et B2.7)

## Méthode de lecture des courbes SOA

Une courbe SOA est un graphique, pas un tableau : aucune valeur ne s'en extrait par lecture de texte, et deux recherches web successives ont échoué pour cette raison exacte. La méthode retenue a été l'**extraction vectorielle** des tracés du PDF, recalés sur les étiquettes d'axes, puis évaluation analytique au point 56 V.

Trois contrôles indépendants valident la méthode :

- Sur la Fig. 13 de l'`IXTH64N10L2`, la ligne horizontale supérieure du gabarit retombe à 140,2 A, contre `I_DM` = 140 A au tableau des maxima absolus.
- Sur les quatre datasheets lues, la valeur déduite de la Fig. 14 coïncide à moins de 1 % près avec la ligne **SOA garantie** du tableau *Safe Operating Area Specification*, qui est une valeur testée et non un tracé.
- Toutes les lignes SOA de cette famille ont une pente exactement −1 en log-log : la SOA y est purement limitée en puissance, sans repli par instabilité thermique. C'est la propriété qu'apporte la famille *Linear L2*, et elle rend la valeur en watts indépendante de la tension.

## `Q302` — MOSFET de hot-swap

Exigence à satisfaire, établie plus haut : **389 W, soit 6,95 A sous 56 V, pendant 422 ms**, marge 1,3 × de TI incluse ; 299 W sans cette marge.

Comparaison sur la valeur **garantie** à `T_C` = 75 °C et `t_p` = 5 s, donc plus sévère que les 422 ms réellement subies — l'impulsion est 11,8 × plus courte que le point d'essai, si bien que la comparaison reste valable *a fortiori* :

| Référence | Boîtier | `R_DS(on)` | `R_thJC` | SOA garantie à 75 °C | Verdict sur 389 W |
|---|---|---|---|---|---|
| `IXTH64N10L2` | TO-247 | 32 mΩ | 0,35 °C/W | 215 W | ÉCHEC, 0,55 × |
| `IXTH110N10L2` | TO-247 | 18 mΩ | 0,21 °C/W | 360 W | ÉCHEC, 0,93 × |
| `IXTN200N10L2` | SOT-227 | 11 mΩ | 0,15 °C/W | 500 W | passe, 1,29 × |
| **`IXTK200N10L2`** | **TO-264** | **≤ 11 mΩ** | **0,12 °C/W** | **625 W** | **passe, 1,61 ×** |

**Retenu : `IXTK200N10L2`** (Littelfuse/IXYS, *Linear L2*, 100 V / 200 A, TO-264 traversant, `V_GSS` ±30 V).

Motif du choix contre l'`IXTN200N10L2`, pourtant suffisant : le SOT-227 est un module isolé à cosses vissées, plus encombrant et plus contraignant à implanter qu'un TO-264 traversant, pour une marge inférieure.

Le fait marquant est que le candidat évident, l'`IXTH64N10L2`, échoue d'un facteur 1,8 : **la contrainte impose un die environ trois fois plus gros que ce que le courant nominal de 4,6 A laisserait supposer.** C'est la conséquence directe et chiffrée du bulk de 15 400 µF conservé.

Lecture détaillée à 56 V (Fig. 14, `T_C` = 75 °C) :

| Durée | `IXTH64N10L2` | `IXTK200N10L2` |
|---|---|---|
| 10 ms | 7,05 A / 395 W | 27,3 A / 1527 W |
| 100 ms | 4,57 A / 256 W | 15,2 A / 850 W |
| DC (`t_p` = 5 s) | 3,78 A / 212 W | 11,2 A / 625 W |

Le déclassement en température de l'équation 19 n'a pas eu à être appliqué : IXYS publie la courbe **directement à `T_C` = 75 °C** et garantit une valeur testée à cette température, ce qui est plus solide qu'un déclassement calculé depuis une courbe à 25 °C.

Vérifications secondaires :

- Régime établi, équation 4 : 4,6 A dans ≤ 11 mΩ donne 0,23 W, et 1,6 W à la limite de courant minimale de 12,1 A. La température de boîtier en régime établi est sans enjeu.
- Charge de grille `Q_g(on)` = 540 nC et `Q_gd` = 115 nC, contre un courant de source de grille du LM5069 de 10 µA min, 16 µA typique, 22 µA max. Ce courant faible **n'entame pas** la marge `t_flt` / `t_start` : la section 8.3.3 énonce que le temporisateur de défaut n'est actif que *pendant* la limitation de puissance, alors que la charge initiale de la grille la précède.
- Coupure assurée dans les deux modes : 2 mA de pull-down sur expiration du temporisateur, 230 mA sur disjoncteur.

### Contrainte d'implantation créée par ce choix

La SOA suppose le boîtier **maintenu** à 75 °C pendant l'événement. `Q302` doit donc être monté sur radiateur, sans quoi la courbe ne s'applique pas. À reporter en Phase E.

`NEEDS_DATA` — stabilité de la boucle de limitation de puissance du LM5069 face à une charge de grille de 540 nC. TI ne spécifie aucune capacité de grille maximale et aucune donnée trouvée ne permet de trancher.

## `F301` — fusible

**Retenu : `Schurter UMT-H` 12,5 A, référence `3403.0285.11`** — CMS 5,3 × 16 mm, temporisé T, 250 VAC / **125 VDC**, pouvoir de coupure **1000 A à 125 VDC**.

Le piège identifié en amont, celui des cartouches calibrées en alternatif seulement ou en continu à 32 V, est écarté : le calibre continu et son pouvoir de coupure sont énoncés séparément au tableau des variantes. À noter que la gamme ne tient pas 250 VDC sur tous ses calibres : elle retombe à 125 VDC de 10 à 16 A, puis à 72 VDC de 20 à 50 A.

Coordination avec le LM5069, vérifiée sur la table *Pre-Arcing Time* de la datasheet, ligne des calibres 0,160 A à 12,5 A :

| Sollicitation | Courant | Réponse du fusible |
|---|---|---|
| Régime nominal | 4,6 A = 0,37 × `In` | aucune |
| Crête musicale sur 4 Ω | 9,3 A = 0,74 × `In` | aucune |
| Limitation de courant LM5069, ≤ 422 ms | ≤ 15,4 A = 1,23 × `In` | **≥ 60 min avant amorçage** : ouverture impossible |
| Disjoncteur LM5069 | 20 à 32,5 A | l'électronique coupe la première |
| `Q302` défaillant en court-circuit | ≥ 125 A = 10 × `In` | 10 à 100 ms |

Le rôle de `F301` est donc précisément borné : **ultime recours contre un `Q302` en court-circuit ou un court-circuit de câblage**, jamais protection de surcharge. Cette fonction appartient au LM5069, et un calibre plus bas romprait la coordination en ouvrant sur un événement que l'électronique est conçue pour encaisser.

Deux conséquences à reporter :

- Les calibres de cette famille sont établis sur carte d'essai à pistes de **7,5 mm en cuivre de 140 µm** pour le calibre 12,5 A. Le rail d'entrée devra s'en approcher, faute de quoi un déclassement s'applique.
- **Aucune empreinte KiCad existante ne convient** : `Fuse_Schurter_UMT250` vise un corps de 3 × 10,1 mm, pastilles 2 × 3,75 mm à ± 4,25 mm, contre 5,3 × 16 mm ici. Empreinte locale à créer, à rattacher à D1.4.

## `D301` — TVS

**Retenue : `SMDJ58CA`**, bidirectionnelle 3000 W, DO-214AB. Le motif du choix bidirectionnel est exposé plus bas.

`V_RWM` = 58 V, strictement au-dessus des 56 V que le rail peut légitimement atteindre ; `V_BR` = 64,4 à 71,2 V ; `V_C` = 93,6 V à `I_PP` = 32,1 A.

Le plafond de tension n'est pas fixé par le TPA3255, que `Q302` isole en surtension via l'OVLO, mais par ce que `D301` protège réellement, c'est-à-dire l'amont : `U8` et `Q301`. La note (3) des maxima absolus du LM5069 est décisive :

> The GATE pin voltage is typically 12 V above VIN when the LM5069 is enabled. Therefore, the Absolute Maximum Ratings for VIN (100 V) applies only when the LM5069 is disabled, or for a momentary surge to that voltage because the Absolute Maximum Rating for the GATE pin is also 100 V.

Le circuit étant actif, la broche `GATE` flotte 12 V au-dessus de `VIN` : le plafond de service est **88 V**, et non 100 V. La phrase autorise bien un dépassement momentané jusqu'à 100 V, ce qui décrit exactement un écrêtage de TVS ; la lecture prudente a néanmoins été retenue.

Ce plafond départage deux boîtiers qui affichent pourtant la même `V_C` de 93,6 V, mais pas au même courant :

| Référence | Puissance | `V_C` = 93,6 V à | Résistance dynamique déduite | `V_C` estimée à 16 A |
|---|---|---|---|---|
| `SMCJ58A` | 1500 W | 16,1 A | ≈ 1,6 Ω | ≈ 93,6 V, au-dessus des 88 V |
| **`SMDJ58CA`** | **3000 W** | **32,1 A** | **≈ 0,80 Ω** | **≈ 81 V, sous les 88 V** |

Réserve explicite : la dernière colonne est une **interpolation linéaire** entre `V_BR` typique et le couple (`V_C`, `I_PP`) publiés. Aucune des deux datasheets ne spécifie `V_C` en dessous de `I_PP`. Le choix du 3000 W est donc motivé par une marge, pas par une valeur garantie.

L'alignement des colonnes de la table SMDJ, dont l'extraction texte est décalée, a été contrôlé en vérifiant que le produit `V_C` × `I_PP` redonne 3000 W sur toute la gamme : 35,5 × 84,5 pour le 22 V, 93,6 × 32,1 pour le 58 V, 103 × 29,1 pour le 64 V.

`NEEDS_DATA` — `V_C` de `SMDJ58CA` à un courant de surge inférieur à `I_PP`, non spécifiée.

### Ce que `D301` impose réellement à `Q301`, et ce qu'elle n'impose pas

Correction d'une déduction fausse posée en première rédaction. `Q301` **n'est pas** contraint à `V_DS` ≥ 100 V par l'écrêtage : pendant un écrêtage, sa grille est tenue 15 V sous sa source par `R305` et `D302`, donc il **conduit**, et son `V_DS` reste voisin de zéro. Ce que l'écrêtage impose, c'est 93,6 V sur le **nœud**, donc sur `VIN` de `U8` — c'est l'analyse du plafond de 88 V ci-dessus, qui elle reste valable.

La contrainte réelle sur `Q301` est son **blocage en inversion** : tenir la tension d'alimentation appliquée à l'envers, soit 56 V, plus une marge. Un calibre 100 V reste prudent mais n'est pas imposé par la TVS.

### Conséquence : `D301` doit être bidirectionnelle

Une TVS **unidirectionnelle**, cathode sur `PVDD_FUSED` et anode sur `GND`, entre en conduction directe dès que l'alimentation est branchée à l'envers : elle court-circuite la source et fait fondre `F301`. Cela annule la fonction même de `Q301`, dont l'objet est précisément de rendre l'inversion inoffensive.

**Retenu : `SMDJ58CA`**, variante bidirectionnelle. Elle bloque jusqu'à −58 V en inversion, laisse `Q301` faire son travail, et partage exactement les caractéristiques électriques de la variante `A`, dont elle occupe la même ligne de tableau : `V_RWM` = 58 V, `V_BR` = 64,4 à 71,2 V, `V_C` = 93,6 V à `I_PP` = 32,1 A.

Le symbole déjà en place, `Device:D_TVS`, est bidirectionnel : ce choix rétablit du même coup la cohérence entre le symbole et le composant.

## Réserves sur ce lot

- Disponibilité en stock non vérifiée. L'`IXTK200N10L2` est actif au catalogue Littelfuse et référencé chez JLCPCB sous `C3281712`, mais les niveaux de stock varient selon les distributeurs.
- Les datasheets IXYS ont été lues sur un miroir tiers, les serveurs Littelfuse refusant les requêtes automatisées. L'identité de chaque fichier a été contrôlée sur son en-tête et ses valeurs de tableau, après qu'un premier PDF récupéré chez un distributeur se soit révélé être un composant sans aucun rapport.

## Sources de ce lot

- `IXTK200N10L2` — https://www.littelfuse.com/products/power-semiconductors-control-ics/mosfets-si-sic/n-channel-linear/l2/ixtk200n10l2
- `IXTH64N10L2` — https://www.littelfuse.com/products/power-semiconductors-control-ics/mosfets-si-sic/n-channel-linear/l2/ixth64n10l2
- `IXTN200N10L2` — https://www.littelfuse.com/products/power-semiconductors/discrete-mosfets/n-channel-linear/l2/ixtn200n10l2.aspx
- Schurter UMT-H — https://www.schurter.com/en/datasheet/typ_UMT-H.pdf
- Série SMDJ — https://www.farnell.com/datasheets/2794255.pdf
- LM5069, maxima absolus et section 8.3 — https://www.ti.com/lit/ds/symlink/lm5069.pdf
- TI, *Using MOSFET Safe Operating Area Curves in Your Design* — https://www.ti.com/lit/pdf/sluaao2

# `Q301` figé et défaut de brochage corrigé (B2.9)

## `Q301` — P-MOS d'anti-inversion

Exigences réunies : bloquer 56 V en inversion avec marge, conduire 4,6 A en continu avec des crêtes musicales à 9,3 A et jusqu'à 15,4 A pendant au plus 422 ms si le LM5069 limite, tolérer un `V_GS` clampé à 15 V par `D302`, et dissiper peu.

**Retenu : `IPP330P10NM`** — Infineon OptiMOS, P-canal, TO-220-3. Valeurs lues sur la datasheet *Final Data Sheet* Rev. 2.0 du 2021-05-10, dépourvue de couche texte et donc lue par rendu de page.

| Paramètre | Valeur |
|---|---|
| `V_DS` | −100 V |
| `R_DS(on)` max à `V_GS` = −10 V | 33 mΩ |
| `I_D` à `T_C` = 25 °C | −62 A |
| `I_D` à `T_A` = 25 °C, `R_thJA` = 40 °C/W | **−6,9 A** |
| `I_D,pulse` | −248 A |
| `V_GS` | −20 à +20 V |
| `Q_G` | −189 nC |
| `E_AS` | 1960 mJ |
| `R_thJC` | 0,5 °C/W |
| `T_j` | −55 à +175 °C |

Vérifications :

- **Blocage en inversion** : 100 V contre les 56 V que la source peut appliquer à l'envers, marge 1,79 ×.
- **`V_GS`** : la Zener `D302` de 15 V clampe à 15,75 V au pire de sa tolérance de 5 %, contre ±20 V admis. Marge 1,27 ×.
- **Conduction nominale** : 4,6² × 33 mΩ = 0,70 W.
- **Limitation de courant LM5069** : 15,4² × 33 mΩ = 7,8 W pendant au plus 422 ms, soit 3,3 J. Avec `R_thJC` = 0,5 °C/W et une impédance thermique transitoire inférieure à cette valeur sur une telle durée, l'échauffement de jonction reste inférieur à 5 °C. Sans enjeu.
- **Aucune contrainte SOA** : contrairement à `Q302`, `Q301` ne travaille jamais en régime linéaire. Il est soit passant, soit bloqué.

**Contrainte d'implantation** : la datasheet plafonne le courant continu à **6,9 A** avec la seule surface de cuivre de référence, soit 6 cm² sur une couche de 70 µm donnant `R_thJA` = 40 °C/W. Les 4,6 A nominaux passent, mais la tenue des crêtes à 9,3 A repose sur leur brièveté. Prévoir au minimum cette surface, un radiateur restant préférable.

Une variante CMS existe dans le guide de sélection Infineon, `IPB320P10LM` en D²PAK à 32 mΩ, si le traversant pose problème. Chiffre issu du guide seul ; sa datasheet n'a pas été lue.

### Réserve de conception, non levée

À 100 V, un P-canal reste environ trois fois moins bon qu'un N-canal à surface de silicium égale. L'alternative moderne est un contrôleur de diode idéale, `LM74700` ou `LM5050`, pilotant un N-canal dans le rail positif : sa pompe de charge fournit précisément la commande côté haut dont l'absence avait fait rejeter le montage N-canal en B2.3. Cette voie **n'a pas été retenue** — la topologie P-MOS est tranchée et ses 0,70 W de pertes sont acceptables — mais l'arbitrage est consigné ici pour qu'il reste révisable.

## Défaut de brochage trouvé sur `Q301` et `Q302`

Défaut réel, **invisible à l'ERC**, trouvé en croisant les symboles avec les brochages constructeurs des composants une fois ceux-ci figés.

Les deux transistors portaient un symbole de la famille `*_GSD`, qui déclare **broche 2 = Source et broche 3 = Drain**. Or les deux composants retenus ont **broche 2 = Drain, qui est la semelle, et broche 3 = Source** :

| Repère | Composant | Boîtier | Brochage réel |
|---|---|---|---|
| `Q301` | `IPP330P10NM` | TO-220-3 | 1 = Gate, 2 = Drain (semelle), 3 = Source |
| `Q302` | `IXTK200N10L2` | TO-264 | 1 = Gate, 2 = Drain (semelle), 3 = Source |

Conséquence si le défaut n'était pas corrigé : au report vers le PCB, la pastille du drain physique serait câblée sur le net de source et réciproquement. Les deux transistors seraient montés à l'envers, diode de structure passante en permanence, et **ni l'anti-inversion ni le hot-swap ne fonctionneraient**. L'ERC ne voit rien de tout cela : la numérotation des broches lui est indifférente.

**Correctif appliqué** : `Q301` passe à `Transistor_FET:Q_PMOS_GDS` et `Q302` à `Transistor_FET:Q_NMOS_GDS`.

Le correctif est **géométriquement neutre**, ce qui a été vérifié dans `Transistor_FET.kicad_sym` avant de l'appliquer : les variantes `GSD` et `GDS` ont exactement les mêmes positions de broches — `G` à (−5,08 ; 0), `D` à (2,54 ; 5,08), `S` à (2,54 ; −5,08). Seuls les numéros changent. La connectivité, qui repose sur la coïncidence label/ancre, est donc intacte.

Le schéma ne contient que ces deux transistors : le défaut est entièrement circonscrit.

## Sources de ce lot

- `IPP330P10NM` — https://www.infineon.com/dgdl/Infineon-IPP330P10NM-DataSheet-v02_00-EN.pdf
- Infineon, *P-channel MOSFETs Selection guide 2023* — https://www.infineon.com/assets/row/public/documents/24/66/infineon-productselectionguide-p-channel-mosfets-productselectionguide-en.pdf
