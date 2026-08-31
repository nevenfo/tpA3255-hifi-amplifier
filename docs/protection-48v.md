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

Montage retenu en Phase B : P-MOSFET série sur le rail positif, grille bridée par Zener — le montage actuel de `Q301`, grille auto-polarisée sur le drain, expose le `V_GS` à 48 V contre un maximum typique de ±20 V, et doit être corrigé.

- Candidat `IRF9540NS` (Vishay/Infineon, D2PAK/TO-263) : `V_DS` = -100 V, `I_D` continu = -23 A, `P_tot` = 3,8 W.
  `NEEDS_DATA` : `R_DS(on)` non recoupé — 117 mΩ et « < 55 mΩ » à `V_GS` = -10 V trouvés dans deux sources secondaires, le PDF officiel n'ayant pas pu être lu. À confirmer dans le tableau *Electrical Characteristics*.
- Réseau de grille, pratique de conception d'après `AND90146` (ON Semi) et un guide de conception PMOS, **non spécifique à 48 V** : Zener maintenant le `V_GS` entre -9 et -15 V, résistance de grille de 100 Ω à quelques kΩ pour limiter le courant Zener sans ralentir la décharge de grille en cas d'inversion.
- Variante N-MOSFET côté masse mentionnée par `AND90146` : meilleur `R_DS(on)` à coût égal, mais impose une pompe de charge ou un contrôleur, et surtout **rompt la masse système directe** — écarté ici, le référencement analogique d'un étage audio en dépend.

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
