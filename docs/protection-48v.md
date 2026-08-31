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

`NEEDS_DATA` : la vérification chiffrée de la SOA contre le profil de charge du bulk n'a pas été faite — elle exige le graphe SOA du datasheet complet du MOSFET finalement retenu.

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

`NEEDS_DATA` : aucun fusible traversant de format classique (5 × 20 mm ou 6,3 × 32 mm) explicitement calibré 48–63 VDC entre 10 et 20 A n'a été identifié sur datasheet.

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
- Équation 13 : **`C_TIMER` = 3,9 µF**.
- Équation 14 : `t_flt` = 122 ms au minimum, 184 ms typique, 318 ms au maximum.

Critère de non-coupure au démarrage, vérifié en croisant les pires cas des deux côtés (et non typique contre typique comme dans l'exemple TI) : `t_flt,min` = 122 ms > `t_start,max` = 94 ms, **marge × 1,30**.

## Exigence SOA imposée au MOSFET — point dur de cette conception

Méthode de la section 9.2.1.2.5 : en défaut, le MOSFET subit `V_DS` = `V_IN,MAX` et `I_D` = `P_LIM` / `V_IN,MAX` pendant `t_flt`.

**Pire cas : 56 V et 5,34 A pendant 318 ms.** Avec la marge de 1,3 × recommandée par TI, le MOSFET doit tenir **6,95 A sous 56 V pendant 318 ms, soit 389 W en mode linéaire**.

C'est une contrainte sévère, et elle est **structurelle, non un défaut de réglage** : `P_LIM × t_flt` suit l'énergie de charge du bulk, que la limite de puissance soit réglée haut ou bas. Charger 15 400 µF sous 56 V stocke 24,1 J, que le MOSFET doit dissiper à l'identique ; les dispersions du temporisateur (51 à 120 µA) et de la limite de puissance (± 24 %) portent l'exposition pire cas bien au-delà.

C'est pourquoi `P_LIM` est réglé **près du plafond** : à énergie constante, une limite de puissance élevée raccourcit `t_flt`, et la SOA d'un MOSFET s'améliore beaucoup plus vite quand la durée diminue que lorsque le courant diminue.

Effet du bulk sur la durée d'exposition, à `P_LIM` maximal :

| Bulk | `t_start` pire cas | `C_TIMER` | Exposition SOA |
|---|---|---|---|
| 15 400 µF (actuel) | 94 ms | 3,9 µF | 389 W pendant **318 ms** |
| 8 200 µF | 52 ms | 2,2 µF | 377 W pendant **179 ms** |
| 4 700 µF | 29 ms | 1,2 µF | 390 W pendant **98 ms** |

`NEEDS_DATA` — **référence exacte du MOSFET de hot-swap `Q302`**, à choisir sur courbe SOA constructeur vérifiant 6,95 A / 56 V / 318 ms. Un MOSFET à conduction seule ne convient pas : il faut une SOA garantie en mode linéaire. La vérification devra appliquer le déclassement en température de l'équation 19, la SOA étant spécifiée à 25 °C de boîtier alors que le boîtier est chaud pendant l'événement.

`NEEDS_DATA` — `R_DS(on)` et résistance thermique du MOSFET retenu, pour l'équation 4 (température de boîtier en régime établi, à maintenir sous 125 °C).

## Composants figés à ce stade

| Repère | Valeur | Équation |
|---|---|---|
| `R1` / `R2` / `R3` (seuils) | 191 kΩ / 5,11 kΩ / 9,09 kΩ, 1 % | 21 à 24 |
| `R_SNS` | 4 mΩ | 1 |
| `R_PWR` | 147 kΩ | 9 |
| `C_TIMER` | 3,9 µF | 13 |
| `U8` | `LM5069-2`, VSSOP-10 | — |

`NEEDS_DATA` — correspondance broche/numéro du boîtier VSSOP-10, à relever sur le schéma de brochage avant la capture du symbole.
