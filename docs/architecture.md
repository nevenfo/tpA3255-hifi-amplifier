# Architecture électronique initiale — Amplificateur Hi-Fi stéréo

Date de gel initial : 2026-08-27. Ce document précède toute création de symbole KiCad.

## Sources fabricant primaires

- TPA3255, datasheet TI `SLASEA8A`, révision A, octobre 2016 : <https://www.ti.com/lit/ds/symlink/tpa3255.pdf>
- TPA3255EVM, guide TI `SLOU441`, juillet 2016 : <https://www.ti.com/lit/ug/slou441/slou441.pdf>
- TPA3255EVM, schéma TI `SLAR129A` : <https://www.ti.com/lit/pdf/slar129>
- OPA1612, datasheet TI `SBOS450C`, révision C, août 2014 : <https://www.ti.com/lit/ds/symlink/opa1612.pdf>
- Conversion single-ended vers différentiel pour TPA32xx, note TI `SLAA719` : <https://www.ti.com/lit/an/slaa719/slaa719.pdf>

Les datasheets ont été vérifiées comme documents officiels TI actuels au 2026-08-27. L’EVM sert d’ancre de validation ; ses choix ne sont pas copiés aveuglément.

## Schéma-bloc retenu

```text
48 VDC externe protégé
  ├─ bulk PVDD ── TPA3255 en BTL stéréo ── 4 × (15 µH + 680 nF) ── sorties L/R 4–8 Ω
  └─ buck 15 V ── LDO 12 V
                  ├─ VDD/GVDD du TPA3255
                  ├─ filtre LC ── +12V-OA ── VMID 6 V
                  │                         ├─ RCA L ─ volume 10 kΩ log ─ OPA1612 (−1 puis −1)
                  │                         └─ RCA R ─ volume 10 kΩ log ─ OPA1612 (−1 puis −1)
                  └─ LDO 3.3 V ── supervision/commande
```

Le PCB n’accepte aucun secteur. La source nominale retenue pour dimensionnement est `48 VDC`, capable de `10 A` transitoires. La puissance réellement continue dépendra de l’alimentation, du dissipateur, de la ventilation et de la température ambiante.

### Exigence opposable sur l’alimentation externe (D1.13)

**`REQ-PSU-1` — la tension de sortie de l’alimentation 48 V doit rester ≤ 53,5 V en toutes conditions**, à vide comme en charge, tolérance de fabrication, dérive thermique et régulation de charge incluses.

Ce n’est pas une préférence mais la condition à laquelle le TPA3255 reste dans ses *Recommended Operating Conditions* sur une charge de 4 Ω (`SLASEA8A`, tableau 7.3). Toute alimentation régulée 48 V à ± 5 % délivre au plus 50,4 V et laisse **3,1 V de marge** ; l’exigence est donc satisfaite par construction par les alimentations du commerce, mais elle doit être **vérifiée sur la datasheet du modèle retenu** et non supposée.

**Ce que cette exigence remplace.** Le seuil de surtension du `LM5069`, réglé à 56,4 V, **ne peut pas** faire respecter cette limite : sa dispersion spécifiée vaut ± 6 % pour une fenêtre à couvrir de ± 3 %, et l’abaisser provoquerait des coupures en service. La démonstration complète figure dans `docs/protection-48v.md`, section « D1.13 ». Le `LM5069` est donc, et reste, une protection contre un **défaut** d’alimentation ; le respect des conditions recommandées vient de `REQ-PSU-1`.

**Résidu assumé.** Une panne d’alimentation produisant 53,5 à 56,4 V laisserait la carte fonctionner hors conditions recommandées sans coupure. Ce résidu est accepté : la bande reste **12,6 V sous le maximum absolu de 69 V**, la limite franchie porte sur le courant de sortie et non sur la tenue en tension, et l’`OCP` interne du TPA3255 (17,0 A en CB3C) ainsi que l’`OTW`/`OTSD` restent actifs dans cette bande.

## Puissance et mode TPA3255

- `U_PWR = TPA3255DDV`, HTSSOP-44 `DDV`, PowerPAD supérieur destiné au couplage à un dissipateur et à la masse selon TI.
- Mode `BTL stéréo`, `M1=0`, `M2=0`.
- Canal gauche : charge entre `OUT_A` et `OUT_B`.
- Canal droit : charge entre `OUT_C` et `OUT_D`.
- `PBTL` est exclu : c’est un mode mono et ne répond pas au besoin stéréo.
- `PVDD` nominal 48 V, dans la plage TI 18–53.5 V pour 4 Ω (18 / 51 / 53.5 V min-typ-max, tableau 7.3). L’option jusqu’à 56.5 V pour charge ≥6 Ω n’est pas utilisée, puisque la carte doit accepter 4 Ω ; elle exigerait de surcroît un seuil de surintensité réduit (note 1 du même tableau), donc de modifier `OC_ADJ`. Le respect de la borne 53.5 V est assuré par `REQ-PSU-1`, pas par le `LM5069`. Maximum absolu `PVDD_X to GND` : **69 V** (tableau 7.1).
- `VDD`, `GVDD_AB` et `GVDD_CD` reçoivent 12 V. `AVDD` et `DVDD` internes n’alimentent aucune charge externe.
- `FREQ_ADJ = 22.0 kΩ`, cible de commutation nominale 450 kHz, suivant l’EVM.

TI publie 150 W/8 Ω à 1 % THD+N en BTL ; 2 × 100 W/8 Ω est donc dans l’enveloppe électrique annoncée, mais pas validé thermiquement dans notre assemblage.

## Chaîne analogique et volume

- Deux RCA stéréo single-ended, référencées à la masse de signal.
- Potentiomètre double `10 kΩ` logarithmique placé avant les buffers. La référence mécanique finale reste à sélectionner.
- Deux `OPA1612AIDR` SOIC-8 : quatre AOP au total, deux par canal.
- Alimentation simple `+12V-OA`, filtrée depuis le rail 12 V ; `VMID=6 V` produit par 10.0 kΩ/10.0 kΩ et fortement découplé.
- Par canal, deux inverseurs de gain `−1` sont montés en cascade, entrées non-inverseuses à `VMID`, avec résistances de gain `10.0 kΩ 0.1 %` et compensation `22 pF C0G`, conformément à `SLAR129A`, `SLOU441` pp. 16–18 et `SLAA719` p. 3. Le premier étage fournit `−VSE`; le second réinverse ce signal et fournit `+VSE`.
- Résultat : `Vdiff = 2 × VSE`. Les quatre sorties passent par `10 µF` de blocage DC, puis `100 Ω` série et `100 pF` anti-RF avant `INPUT_A/B/C/D`.
- L’OPA1612 SOIC-8 est pin-à-pin avec le NE5532ADR de l’EVM. Sous 0/12 V, sa plage de mode commun garantie est 2–10 V ; à `VMID=6 V` et `2 Vrms` par branche, le signal 3.17–8.83 V y reste et son swing garanti sous 10 kΩ est compatible.
- Un condensateur d’entrée avant le volume est retenu pour bloquer le DC source ; valeur initiale `4.7 µF` avec charge nominale 10 kΩ. Technologie et référence finale restent à sélectionner selon encombrement et distorsion.

Le rôle de l’OPA1612 est donc précisément : buffer faible bruit, conversion SE→différentielle, adaptation d’impédance et filtrage RF. Il ne réalise ni le réglage de volume ni l’amplification de puissance.

## Connectique de liaison au châssis

Trois liaisons quittent la carte vers des organes montés sur le châssis : `J2` et `J3`, deux points chacun, vers les embases RCA d'entrée, et `J4`, six points, vers le potentiomètre de volume déporté hors carte (voir B2.1). **Famille retenue sur arbitrage utilisateur : JST XH au pas de 2,5 mm**, en version verticale par défaut, l'orientation définitive relevant du placement en Phase E.

| Repère | Points | Empreinte | Vers |
|---|---|---|---|
| `J2` | 2 | `JST_XH_B2B-XH-A_1x02_P2.50mm_Vertical` | RCA gauche : `RCA_L`, `GND` |
| `J3` | 2 | `JST_XH_B2B-XH-A_1x02_P2.50mm_Vertical` | RCA droite : `RCA_R`, `GND` |
| `J4` | 6 | `JST_XH_B6B-XH-A_1x06_P2.50mm_Vertical` | potentiomètre : `VOL_L_IN`, `VOL_L`, `GND`, `VOL_R_IN`, `VOL_R`, `GND` |

Ce qui décide ici n'est pas l'encombrement mais le **détrompage**. `J4` porte six conducteurs dont deux masses, et un faisceau de volume rebranché à l'envers après un démontage enverrait l'entrée du potentiomètre sur la masse et son curseur sur l'entrée : panne silencieuse, sans destruction, donc difficile à diagnostiquer. Le boîtier XH est à la fois détrompé et verrouillé par ergot, ce qu'une barrette à vis ne procure pas. La contrepartie assumée est le besoin d'une pince à sertir, et une deuxième famille de connecteur à approvisionner à côté des MaiXu MX126-5.0 déjà employés pour `J1`, `J301` et `J302` — mais ces trois-là véhiculent de la puissance, pas du signal.

## Niveaux et gains

- Gain TPA3255 fixe : 21.5 dB, soit 11.89 V/V.
- 100 W/8 Ω : 28.28 Vrms à la charge.
- Entrée TPA3255 nécessaire : 2.38 Vrms différentiels.
- Avec la conversion `Vdiff=2×VSE`, niveau après volume : 1.19 Vrms SE.
- Une source 2 Vrms atteint donc 100 W avec environ −4.5 dB d’atténuation au volume, laissant une marge raisonnable.
- Impédance d’entrée TPA3255 : 20 kΩ par entrée selon TI.
- Le `VIN=7 Vpp` TI s’applique à chaque broche `INPUT_X`. Une source SE de 2 Vrms produit 2 Vrms par branche, soit 5.66 Vpp par broche et 4 Vrms différentiels ; elle respecte cette limite mais peut faire écrêter l’étage de puissance avant la pleine course du volume.

Le common-mode garanti des entrées TPA3255 n’est pas explicitement donné. Les condensateurs de liaison de l’EVM sont conservés pour laisser le circuit établir sa propre polarisation.

## Alimentation auxiliaire

Architecture initiale reprise de l’EVM :

- `LM5010ASD/NOPB` : buck PVDD vers +15 V.
- `LM2940IMP-12/NOPB` : +15 V vers +12 V.
- `TLV1117-33IDCY` : +12 V vers +3.3 V.

Les références, boîtiers et valeurs vérifiés pour la capture sont centralisés dans `docs/power-block.md`.
- Filtre du rail AOP : `L6=10 µH/0.8 A`, `C81=10 µF`, puis découplages locaux `0.1 µF + 10 µF`.

Valeurs buck d’ancrage EVM : `L1=100 µH/1.5 A`, `R2=182 kΩ`, `R39=4.99 kΩ`, `R40=1.00 kΩ`, `C1=0.047 µF`, `C2=0.1 µF/100 V`, `C3=1 µF/100 V`, `C4=2.2 µF/100 V`, `C6=4.7 µF`, `C7=5600 pF`, `C12=4700 pF`, `C13=0.1 µF`, `C39=47 µF/63 V`. Les diodes et leurs références exactes doivent être confirmées avant capture.

Ce choix évite un convertisseur négatif près de l’analogique. Une alimentation ±12 V n’est pas retenue, car elle imposerait de redessiner la polarisation VMID de l’EVM.

## Découplage et réservoir

TPA3255, minimum TI :

- `VDD` : `10 µF + 0.1 µF`.
- `VBG` : `1 µF`.
- `GVDD_AB`, `GVDD_CD` : `0.1 µF` chacun.
- `BST_A/B/C/D–OUT_A/B/C/D` : `0.033 µF` chacun.
- PVDD : `1 µF/100 V` au plus près de chaque groupe de broches, plus bulk basse ESR.

Complément sourcé le 2026-08-31 par lecture directe de `SLASEA8A` rév. A, pour les broches que la première rédaction n’avait pas couvertes :

- `AVDD` (broche 14) : `1 µF` vers `GND` — Figure 29 « Typical Differential (2N) BTL Application », p. 22.
- `DVDD` (broche 11) : `1 µF` vers `GND` — Figure 29, p. 22.
- `C_START` (broche 15) : `47 nF` vers `GND` — Figure 29, p. 22. La table Pin Functions précise « Startup ramp, requires a charging capacitor to GND ».
- `OC_ADJ` (broche 7) : `22 kΩ` vers `GND`, soit un seuil de `17.0 A` en mode `CB3C` (limitation cycle par cycle) — Table 4 « Device Protection », p. 18. La même table donne `24 k/27 k/30 k` pour `15.7/14.2/12.9 A` en CB3C, et `47 k/51 k/56 k/64 k` pour les mêmes seuils en mode `Latched OC`. Le mode CB3C est retenu : il limite sans verrouiller, comportement préférable en audio.
- `OSC_IOM` (broche 9) et `OSC_IOP` (broche 10) : laissées non connectées — table Pin Functions, « Oscillator synchronization interface. Do not connect if not used. » Le montage est autonome, sans configuration maître/esclave.
- `AVDD` et `DVDD` ne doivent alimenter aucun circuit externe : « The DVDD and AVDD pins are not recommended to be used as a voltage sources for external circuitry » (§10.2.1.2, p. 23).
- Rappel de dimensionnement PVDD : « A minimum voltage rating of 100 V is required for use with a 51-V power supply » et, pour le réservoir associé à chaque pont complet, « 1000 µF, 80 V supports most applications » (§10.2.1.2.2, p. 23).

Ancre EVM pour le bulk : quatre `1500 µF/63 V` locaux et deux `4700 µF/80 V` en entrée. Le nombre final sera ajusté à l’encombrement, au ripple admissible et au courant de l’alimentation externe, sans réduire le découplage céramique critique.

OPA1612 : `0.1 µF` faible ESR au plus près de chaque broche d’alimentation, complété par `10 µF` local par circuit.

## Filtre de sortie

Le réseau initial suit l’application BTL de la datasheet TPA3255, sans mélange avec celui de l’EVM :

- quatre inductances `15 µH`, une par demi-pont ; courant RMS ≥5 A, courant de saturation cible ≥10 A, faible DCR ; référence finale à qualifier ;
- quatre condensateurs film `680 nF`, tolérance et tension à qualifier ;
- fréquence LC idéale calculée : environ 49.8 kHz ;
- réseaux d’amortissement/EMI et valeurs finales à valider par calcul détaillé, implantation et mesures sur prototype. La Figure 29 de `SLASEA8A` fait apparaître, en aval de chaque demi-pont, un réseau `10 nF + 3.3 Ω` et un `1 nF` en plus du condensateur de filtre ; ces éléments ne sont pas figés ici et restent couverts par le `NEEDS_DATA` EMI/stabilité.

Topologie retenue : le condensateur de filtre de chaque demi-pont est référencé à `GND`, en aval de son inductance. C’est la seule lecture cohérente avec les quatre condensateurs annoncés et avec la coupure calculée à 49.8 kHz. La règle « les sorties BTL ne doivent jamais être reliées à la masse » porte sur les bornes haut-parleur, pas sur ce condensateur de filtre.

`NEEDS_DATA: confirmation visuelle de la topologie exacte de la Figure 29 (condensateur de filtre vers la masse ou entre les deux sorties d’un même pont) ; l’extraction texte du PDF ne permet pas de lever complètement l’ambiguïté, le rendu graphique de la page 22 n’ayant pas pu être produit.`

L’EVM utilise à la place `10 µH + 1 µF` avec Coilcraft `MA5172-AE`; ces deux variantes ne seront pas combinées.

## RESET, défauts et séquencement

- `RESET` actif bas, maintenu bas au démarrage et à l’arrêt.
- `TPS3802K33DCKR` repris comme superviseur de l’EVM.
- `FAULT` et `CLIP_OTW` sont open-drain, actifs bas, avec indication LED et points de test.
- Après chute de `FAULT`, attendre au moins 4 ms avant le front montant de `RESET`.
- Les défauts OTE/OC verrouillés exigent un cycle de `RESET`.
- Un bouton MUTE utilisateur agira par la séquence `RESET`; le TPA3255 n’expose pas une broche MUTE audio indépendante dans les éléments vérifiés.

## Protections et connectique

- Entrée PVDD : bornier verrouillable, fusible remplaçable ou empreinte de fusible, protection d’inversion par MOSFET et limitation de surtension à définir.
- Sorties : deux borniers 2 pôles dimensionnés pour le courant haut-parleur, sans référence commune entre bornes BTL.
- Les sorties BTL ne doivent jamais être reliées à la masse.
- Les réseaux de protection haut-parleur contre DC en cas de panne ne sont pas intégrés sans architecture sourcée ; le TPA3255 fournit ses protections internes OC/OT/UV.

## Implantation 4 couches prévue

- L1 : composants et signaux/puissance critiques courts.
- L2 : plan GND continu, sans fente arbitraire entre analogique et puissance.
- L3 : distribution PVDD/12 V/3.3 V et signaux non critiques si nécessaire.
- L4 : signaux, plans locaux et dissipation.
- Boucles PVDD–demi-pont–condensateurs minimales ; bootstrap au plus près.
- Entrées et retours analogiques courts, éloignés des nœuds de commutation et des inductances.
- Filtres LC près des sorties de puissance, borniers haut-parleur en bord de carte.
- **Correction du 2026-08-31, établie par lecture directe du dessin de boîtier `DDV0044D` dans `SLASEA8A` :** le PowerPAD du `DDV` est sur la **face supérieure** du composant (« The package type contains a PowerPAD that is located on the top side of the device for convenient thermal coupling to the heat sink », §6 ; note 5 du dessin de boîtier : « The exposed thermal pad is designed to be attached to an external heatsink »). Le `LAND PATTERN EXAMPLE` ne comporte que **44 pastilles de 1.45 × 0.4 mm au pas 0.635 mm, sans aucune pastille thermique côté PCB**.
- En conséquence, **aucune matrice de vias thermiques sous le TPA3255 n'est possible ni pertinente** : la chaleur sort par le dessus, vers un dissipateur pressé sur le boîtier. La rédaction initiale « PowerPAD et masse reliés à une matrice de vias thermiques » était erronée et est annulée. La dissipation dépend entièrement du dissipateur, de son interface et de sa pression mécanique — ce que le `NEEDS_DATA` dissipateur couvre déjà.
- Les vias thermiques restent pertinents ailleurs sur la carte, notamment sous les régulateurs à languette et pour la continuité du plan de masse, mais pas comme voie de dissipation du TPA3255.

### Épaisseur de cuivre — tranché avant E1.1

**Conclusion : cuivre standard de 35 µm, et rail d'entrée simplement élargi. Les 140 µm que laissait craindre `docs/protection-48v.md` ne sont pas exigés.**

Trois faits, dans cet ordre.

**1. Le courant n'impose rien.** IPC-2221 en couche externe, pour les 4,6 A du régime nominal :

| Échauffement | 35 µm | 70 µm | 105 µm | 140 µm |
|---|---|---|---|---|
| 10 °C | **2,47 mm** | 1,23 mm | 0,82 mm | 0,62 mm |
| 20 °C | 1,62 mm | 0,81 mm | 0,54 mm | 0,40 mm |

Une piste de 2,5 mm en cuivre standard suffit donc, avec un échauffement de 10 °C. Les crêtes musicales à 9,3 A demanderaient 6,5 mm en régime **établi**, ce qu'elles ne sont pas.

**2. Les « 7,5 mm en 140 µm » ne sont pas une exigence de conception mais une condition de mesure.** La datasheet Schurter l'énonce noir sur blanc : *« All measurements are carried out on a test board according to IEC 60127 with the following tracks: […] 10 A, 12.5 A: Track width 7.5 mm, Cu layer 140 µm »*. Moins de cuivre signifie un fusible qui évacue moins bien sa propre chaleur, donc un calibre effectif plus faible. Schurter ne quantifie ce déclassement **que contre la température ambiante**, jamais contre le cuivre : l'ampleur de l'effet n'est pas calculable depuis la datasheet.

**3. Ce qui est calculable, c'est la marge dont dispose la coordination.** La table *Pre-Arcing Time*, ligne 0,160 à 12,5 A, donne 60 min minimum à 1,25 × `In`. La sollicitation continue la plus sévère du rail est la crête musicale à 9,3 A. Elle ne franchit 1,25 × `In` que si le calibre effectif tombe sous **7,4 A**, soit un déclassement de **41 %**. Or la courbe de déclassement Schurter ne descend vers 60 % du calibre qu'aux ambiantes extrêmes de la plage, autour de 100 à 120 °C. **La coordination absorbe donc un effet qu'elle dépasse d'un large facteur**, et l'événement de limitation du LM5069 — 15,4 A pendant au plus 422 ms — reste hors d'atteinte : même à 2 × `In`, le fusible dispose de 120 s.

**4. Et le cuivre épais serait activement nuisible ici.** Le `U6` est un HTSSOP-44 au pas de **0,635 mm**, pastilles de 0,4 mm séparées de 0,235 mm. La gravure d'un cuivre de 140 µm ne tient pas des intervalles de cette finesse : le facteur de gravure croît avec l'épaisseur. Spécifier 140 µm sur les couches externes rendrait le composant principal irroutable. **À confirmer auprès du fabricant retenu**, mais la contrainte est structurelle, pas commerciale.

**Ce qui est donc retenu pour E1.1** : 35 µm sur les quatre couches, rail d'entrée `PVDD` tracé à **7,5 mm là où le placement le permet** — cela ne coûte que de la surface et récupère la condition de largeur du banc IEC, à défaut de son épaisseur —, et jamais moins de 2,5 mm. Le choix du fabricant reste ouvert et ne conditionne pas ce point.

### Règles et classes de nets posées en E1.1

Couches cuivre : `F.Cu` signal, `In1.Cu` **power**, `In2.Cu` **mixed**, `B.Cu` signal. Carte de 1,6 mm.

Règles globales, volontairement conservatrices puisque le fabricant n'est pas choisi : piste minimale 0,20 mm, isolation minimale 0,20 mm, via minimal ø0,60 mm, perçage traversant minimal 0,30 mm, anneau minimal 0,15 mm.

Les huit classes ci-dessous sont dimensionnées sur **IPC-2221 en couche externe, cuivre 35 µm, échauffement 10 °C** : 4,6 A demandent 2,47 mm, 5,0 A demandent 2,77 mm, 2,0 A demandent 0,78 mm, 1,0 A demande 0,30 mm.

| Classe | Piste | Isolation | Nets | Raison |
|---|---|---|---|---|
| `PWR_48V` | 2,50 mm | 0,50 mm | 4 | **Plancher, pas cible.** La cible au routage reste 7,5 mm |
| `PWR_OUT` | 3,00 mm | 0,50 mm | 12 | Sorties de puissance, avant et après filtre |
| `PWR_AUX` | 0,80 mm | 0,25 mm | 9 | 12 V, 3,3 V, 15 V et nœud de commutation du buck |
| `GATE_DRIVE` | 0,50 mm | 0,25 mm | 7 | Bootstraps et grilles |
| `SENSE` | 0,25 mm | 0,25 mm | 8 | Prises de mesure, dont `/PVDD_SENSE` |
| `ANALOG` | 0,25 mm | 0,25 mm | 20 | Chaîne faible bruit |
| `GND` | 0,50 mm | 0,25 mm | 1 | Liaisons hors plan |
| `Default` | 0,25 mm | 0,20 mm | 10 | Logique et contrôle |

Trois points ne se déduisent pas du tableau et doivent survivre au routage :

- **L'isolation de 0,50 mm des nets de puissance est dimensionnée sur 93,6 V**, le pic atteint pendant un écrêtage de la TVS `D301`, et non sur les 48 V nominaux.
- **`/PVDD_SENSE` est délibérément en `SENSE` et non en `PWR_48V`.** C'est la prise de mesure du shunt `R306` de 4 mΩ : une piste large y ajouterait du cuivre en série et fausserait le seuil de limitation.
- **`/SW` et `/OUT_A` à `/OUT_D` sont des nœuds de commutation.** Leur contrainte est d'être **courts**, ce qui relève du placement et qu'aucune classe ne peut exprimer.

**Réserve d'outillage sur ce point.** Aucun outil MCP n'expose l'épaisseur de cuivre par couche, et le `.kicad_pcb` ne porte donc aucun bloc `stackup` explicite : KiCad applique son défaut. **À rendre explicite dans Board Setup avant toute génération de fichiers de fabrication**, et à reporter sur la commande au fabricant.

## Exigence thermique du dissipateur — `REQ-THERM-1`

Le `NEEDS_DATA` dissipateur ne pouvait pas être levé sans donnée extérieure, mais il pouvait être **chiffré**. Ce qui suit transforme une case vide en critère d'achat.

### Ce que `U6` dissipe réellement

Lu sur la **figure 10 de `SLASEA8A`**, *System Power Loss vs Output Power*, BTL, `PVDD` = 51 V, `T_C` = 75 °C, THD+N = 10 %. C'est un tracé : les valeurs sont obtenues par **extraction vectorielle** des courbes du PDF, recalées sur les graduations. Le tracé **est à l'échelle**, contrôlé sur deux paires de graduations indépendantes par axe : 0,26492 contre 0,26500 pt/W en abscisse, 1,2240 contre 1,2250 pt/W en ordonnée. Les trois courbes s'identifient sans ambiguïté par leur extension en abscisse — 632 W, 461 W et 359 W —, ce que recoupe la figure 7.

| Puissance de sortie, deux canaux | 4 Ω | 6 Ω | 8 Ω |
|---|---|---|---|
| 100 W | 21,6 W | 17,0 W | **14,4 W** |
| 200 W | 37,4 W | 28,2 W | **22,4 W** |
| 300 W | 53,9 W | 34,9 W | 25,2 W |

**Recoupement qui valide la lecture** : à 200 W sur 8 Ω, 22,4 W de pertes donnent un rendement de **89,9 %**, soit exactement les « 90 % » que ce document postulait pour établir les 4,6 A du rail. La courbe confirme donc une hypothèse posée bien plus tôt sans preuve.

**Ce que le recoupement révèle aussi** : sur 4 Ω, le rendement tombe à **84,2 %** et le courant d'entrée monte à **4,95 A**. Sans conséquence — le fusible est à 12,5 A, `Q301` tient 6,9 A —, sauf une nuance de routage : à 4,95 A, le plancher de 2,50 mm de la classe `PWR_48V` produit 11,5 °C d'échauffement au lieu de 9,8. La cible de tracé étant 7,5 mm, où l'échauffement tombe à 1,9 °C, le point est sans portée pratique.

### Le point dur n'est pas le dissipateur, c'est l'interface

Le PowerPAD du `DDV0044D` mesure 7,01 × 4,14 mm, soit **29,02 mm²**. C'est très peu, et la résistance d'interface est inversement proportionnelle à cette surface :

| Interface | Conductivité | Épaisseur | Résistance |
|---|---|---|---|
| Pad silicone standard | 1 W/m·K | 0,25 mm | **8,61 °C/W** |
| Pad chargé céramique | 3 W/m·K | 0,25 mm | 2,87 °C/W |
| **Graisse thermique** | 3 W/m·K | 0,05 mm | **0,57 °C/W** |
| Feuille de graphite | 25 W/m·K | 0,05 mm | 0,07 °C/W |
| Feuille d'indium | 80 W/m·K | 0,05 mm | 0,02 °C/W |

**Un pad silicone standard consommerait à lui seul, 8,61 °C/W, plus que la totalité du budget thermique.** Sur 29 mm², l'interface n'est pas un détail de montage : c'est le premier poste. Le reste de ce paragraphe suppose de la **graisse thermique**, soit 0,57 °C/W.

### Budget

Chaîne : `T_J` = `T_A` + `P` × (`RθJC(top)` + `R_TIM` + `RθSA`), avec **`RθJC(top)` = 0,36 °C/W** (`SLASEA8A` § 7.4, colonne JEDEC 4 couches). Rappel de D1.3 : `RθJC(bot)` est `n/a`, il n'existe pas d'autre voie.

Deux critères, et ils ne disent pas la même chose :

- **`T_J` ≤ 125 °C**, seuil d'`OTW`. C'est la limite dure : au-delà, `CLIP_OTW` passe bas.
- **`T_C` ≤ 75 °C**, condition dans laquelle TI a caractérisé *toutes* les courbes de puissance, la figure 33 comprise. C'est le critère à retenir, plus strict, car il préserve la validité de tout ce qui a été utilisé pour dimensionner cette carte.

`RθSA` maximal admissible, **interface de 0,57 °C/W déjà déduite** :

| Cas | `T_A` = 25 °C | `T_A` = 40 °C |
|---|---|---|
| 2 × 100 W sur 8 Ω (22,4 W) | 1,66 °C/W | **0,99 °C/W** |
| 2 × 100 W sur 4 Ω (37,4 W) | 0,77 °C/W | **0,37 °C/W** |

### `REQ-THERM-1`

**Le dissipateur de `U6` doit présenter `RθSA` ≤ 1,0 °C/W en convection naturelle**, monté à la graisse thermique ou mieux, pour l'usage nominal 2 × 100 W sur 8 Ω à 40 °C d'ambiante interne.

**`REQ-THERM-2` — l'interface doit valoir 0,6 °C/W ou moins sur les 29 mm² du PowerPAD.** Graisse thermique, feuille de graphite ou indium. **Un pad silicone standard est explicitement exclu.**

### `REQ-THERM-3` — le 4 Ω est un régime de crête

**Réserve levée sur arbitrage utilisateur.** L'usage continu à pleine puissance sur 4 Ω exige `RθSA` ≤ 0,37 °C/W à 40 °C, hors d'atteinte en convection naturelle dans un volume raisonnable. Trois issues étaient ouvertes — ventilation forcée, ambiante interne maintenue basse, ou acceptation que 4 Ω soit un régime de crête. **La troisième est retenue.**

**`REQ-THERM-3` — le régime nominal continu est 2 × 100 W sur 8 Ω. Une charge de 4 Ω est admise en régime musical, où le rapport crête/moyenne maintient la dissipation moyenne bien en deçà des 37,4 W, mais pas en sinus continu pleine puissance.**

Ce que cet arbitrage change, et ce qu'il ne change pas :

- **`REQ-THERM-1` reste à 1,0 °C/W** et devient le seul critère d'achat du dissipateur. Sans cet arbitrage il aurait fallu viser 0,37, soit un tout autre objet.
- **Le dimensionnement électrique reste inchangé.** Il a été établi sur 4 Ω dans tous les cas : fusible 12,5 A, `Q301` à 6,9 A, classe `PWR_48V`, bulk. Le 4 Ω de crête reste donc entièrement couvert côté courant. C'est bien un plafond **thermique** et non électrique.
- **Le garde-fou est matériel, pas déclaratif.** Si l'utilisateur final maintient malgré tout un sinus 4 Ω pleine puissance, la protection thermique propre au TPA3255 agit : `OTW` à 125 °C de jonction puis coupure. La conséquence d'un dépassement est une mise en sécurité, pas une destruction.
- **Ce n'était pas un défaut de conception mais une limite physique** : 37 W à évacuer par 29 mm² de contact.

**À reporter en H1.3** parmi les limites non mesurées, et en H2 dans la documentation de la carte : une carte dont le régime nominal est conditionnel doit le dire.

### Rectification de `REQ-THERM-1` — la résistance d'étalement manquait au budget

**Trouvée en E1.7, en cherchant comment fixer le dissipateur.** Le budget de E1.6 enchaîne `RθJC(top)`, l'interface et `RθSA`, et s'arrête là. Il manque un terme, et il n'est pas petit.

Les 22,4 W entrent dans le dissipateur par les **29,02 mm² du PowerPAD**, c'est-à-dire par un point. Un `RθSA` de catalogue, lui, est le plus souvent mesuré **base chauffée uniformément**. Entre les deux se trouve la **résistance d'étalement**, le prix à payer pour répartir un flux ponctuel sur toute la base.

Plancher théorique, source circulaire équivalente de rayon `a` = √(A/π) = 3,04 mm sur un demi-espace, `R` = 1/(4·k·a) :

| Base | `k` | Étalement, **plancher** |
|---|---|---|
| Aluminium 6063 | 200 W/m·K | **0,411 °C/W** |
| Aluminium 1050 | 229 W/m·K | 0,359 °C/W |
| **Cuivre** | 390 W/m·K | **0,211 °C/W** |

C'est bien un **plancher** : le demi-espace infini est le cas le plus favorable à l'étalement. Une base réelle, d'épaisseur finie et refroidie sur une face, fait pis. Le sens de lecture est donc : *au mieux* 0,41 °C/W en aluminium.

**Forme non ambiguë de l'exigence.** Le budget total au-dessus de la jonction, à 22,4 W, 40 °C d'ambiante et `T_C` ≤ 75 °C, vaut (75 − 40) / 22,4 = **1,5625 °C/W**. C'est cette somme qui est l'invariant :

> **`REQ-THERM-1` (forme rectifiée) — interface + étalement + dissipateur ≤ 1,5625 °C/W**, du dessus du boîtier de `U6` à l'air ambiant interne.

Ce que cela donne selon la manière dont le fabricant a caractérisé la pièce, interface à 0,57 °C/W déduite :

| Hypothèse de caractérisation | `RθSA` admissible |
|---|---|
| Pièce mesurée **sur une source de la taille du composant** — l'étalement est déjà dedans | **0,99 °C/W** |
| Extrusion générique, **base chauffée uniformément**, base aluminium | **0,58 °C/W** |
| Idem, mais **base ou insert cuivre** | 0,78 °C/W |

**Le « ≤ 1,0 °C/W » de E1.6 n'était donc juste que dans le premier cas.** Pour une extrusion générique, la vraie cible est **0,58 °C/W**, soit une pièce nettement plus grosse que ce que le repère de E1.6 laissait attendre.

**Conséquence de sélection, à appliquer à chaque candidat de la short-list** : lire *comment* le `RθSA` a été obtenu, et pas seulement sa valeur. Une pièce vendue pour un boîtier précis — comme celle de l'EVM, explicitement conçue pour les modules `TAS5624`/`TAS5622` en même boîtier `DDV` — est mesurée source réelle. Une extrusion de catalogue au mètre ne l'est pas.

**Et cela redonne du poids au cuivre.** Passer la base de l'aluminium au cuivre récupère 0,20 °C/W, soit environ 13 % du budget total, sans un centimètre d'encombrement supplémentaire. Un insert cuivre sous une base aluminium est le compromis usuel.

### Ce que fait l'EVM, et pourquoi ça ne suffit pas ici

`SLOU441` nomme sa solution thermique en nomenclature, ce qui donne enfin une référence de départ **et un schéma de fixation** : `H1` = **`ATS-TI1OP-519-C1-R3`**, Advanced Thermal Solutions, *Heat Sink, Vertical*, accompagné de vis **M3 × 5 mm** et d'entretoises **M3 de 25 mm** (`Keystone 24438`).

Sa fiche fabricant, relevée sur `qats.com/DataSheet/ATS-TI1OP-519-C1-R3`, donne :

| Cote | Valeur |
|---|---|
| Longueur × largeur × hauteur | **78 × 36 × 35,6 mm** |
| Fixation | **deux trous taraudés M3**, entraxe **36,8 mm**, profondeur 6 mm |
| Matière / finition | AL-6063, anodisé noir |

| Vitesse d'air | `RθSA` non canalisé |
|---|---|
| **0 (convection naturelle)** | **non publiée** |
| 1,0 m/s | 2,2 °C/W |
| 2,0 m/s | 1,6 °C/W |
| 4,0 m/s | 1,2 °C/W |

**Ce tableau tranche la question.** La fiche annonce pourtant en tête « *optimized for natural convection air cooling* », mais **ne publie aucune valeur à vitesse nulle** : la première ligne mesurée est déjà à 1 m/s d'air forcé. En convection réellement naturelle, `RθSA` est donc supérieur à 2,2 °C/W. Même en soufflant 4 m/s dessus, 1,2 °C/W reste au-dessus des 1,0 exigés.

Conséquence chiffrée, en lui prêtant généreusement ses 2,2 °C/W : `T_C` = 40 + 22,4 × (0,57 + 2,2) = **102 °C**, contre 75 visés. **Le dissipateur de l'EVM est environ trois fois trop petit pour cette carte.** Ce n'est pas une critique de TI : l'EVM est un instrument de paillasse, alimenté 5–14 A et posé à l'air libre, pas un amplificateur en boîtier fermé fonctionnant en continu.

**Ce qu'il faut malgré tout lui reprendre, c'est la mécanique.** Le montage résout le problème que pose un PowerPAD sur le dessus d'un CMS soudé : le dissipateur porte ses propres taraudages M3 et se boulonne **par le dessous, à travers le PCB**, deux vis encadrant la puce. Le circuit imprimé n'a donc pas à supporter le poids ; il fournit la contre-pression. **Contrainte à porter en E1.2 : deux perçages M3 de passage, de part et d'autre de `U6`, à l'entraxe du dissipateur retenu, et le dégagement de composants correspondant.** L'entraxe de 36,8 mm de l'EVM est un ordre de grandeur, pas une valeur à figer avant le choix du modèle.

**La figure 2 de `SLOU441` montre la contrainte que ce montage impose au placement, et elle est lourde.** Le dissipateur occupe un **rectangle entièrement vide** au centre-gauche d'une carte de **160 × 120 mm**, ailettes verticales, sans aucun composant haut sous son emprise. C'est mécaniquement inévitable : la base repose sur le dessus de la puce, donc à environ 1 mm du PCB, et **tout ce qui passe sous elle doit tenir dans cette hauteur** — des 0603 et rien d'autre. **L'emprise au sol du dissipateur est donc une zone d'interdiction de hauteur, pas seulement un dégagement.**

C'est ce qui rend le repère de taille ci-dessous inquiétant plutôt que rassurant, et il faudra le regarder en face en E1.2 : une base de 150 × 100 mm stériliserait 150 cm² sur une carte qui, à l'échelle de l'EVM, en fait 192. Trois issues existent — repousser `U6` en bord de carte et laisser le dissipateur déborder, interposer un bloc épais qui surélève le champ d'ailettes au-dessus des composants, ou choisir un profil à base étroite et ailettes larges. **Le choix du modèle et le plan de placement sont donc un seul et même problème, pas deux.**

À titre de repère d'implantation, l'EVM range ses fonctions ainsi : entrées analogiques à gauche, puce et dissipateur au centre-gauche, filtre LC et bulk à droite, sorties en bord droit.

**Ordre de grandeur visé, pour lire la suite.** À 1,0 °C/W en convection naturelle il faut de l'ordre de 0,15 m² de surface d'ailettes, soit un profil extrudé de la classe **150 × 100 × 40 mm** — environ six fois le volume de celui de l'EVM. C'est un repère de vraisemblance, pas un calcul de dimensionnement : seule une valeur publiée par un fabricant fait foi.

### E1.7 — la solution n'est pas un dissipateur, c'est le coffret

Deux constats de E1.7 se combinent pour disqualifier l'approche « gros dissipateur posé sur la puce » : **l'emprise au sol est une zone d'interdiction de hauteur**, et **la cible corrigée est 0,58 °C/W et non 1,0**. Un profil capable de 0,58 °C/W en convection naturelle fait la taille de la carte entière ; le poser dessus reviendrait à stériliser l'implantation.

**Le déblocage vient d'ailleurs, et il est arithmétique.** Un dissipateur logé *dans* le boîtier évacue vers l'air interne, posé à 40 °C par hypothèse. Un **flanc de coffret** évacue vers l'air de la pièce, à 25 °C. Ces 15 K valent, à 22,4 W, **0,67 °C/W de budget** — davantage que la moitié du budget d'origine. Le budget total passe de 1,562 à **2,232 °C/W**, et après déduction de l'interface et de l'étalement il reste **1,251 °C/W** au lieu de 0,582.

**Le coffret n'est donc pas une contrainte à subir après le dissipateur : c'est le dissipateur.**

#### Donnée fabricant

Modushop / HiFi 2000, gamme *Pesante Dissipante*, document `PESANTE _ DISSIPANTE Thermal info.pdf` publié par le fabricant. Valeurs **par flanc**, convection naturelle, ambiante 25 °C :

| Modèle | Flanc (chacun) | `RθSA` publié |
|---|---|---|
| `02/300` — 2U | 300 × 80 × 40 mm | **0,45 °C/W** |
| `03/300` — 3U | 300 × 120 × 40 mm | 0,41 °C/W |
| `04/300` — 4U | 300 × 160 × 40 mm | 0,31 °C/W |
| `04/400` — 4U | 400 × 160 × 40 mm | **0,23 °C/W** |

Le coffret en porte **deux**. Un seul suffit ici, ce qui laisse le second disponible pour l'alimentation ou simplement inutilisé.

Cotes de la gamme, relevées sur le catalogue fabricant : hauteurs **2U = 80 mm, 3U = 120, 4U = 165, 5U = 210** hors tout ; profondeurs 300 ou 400 mm ; **largeur intérieure utile entre les deux dissipateurs de 360 mm, identique sur tous les modèles** ; façade aluminium usinée de 10 mm ; embase intérieure pré-percée disponible en option (`01/05`), solidaire des flancs.

#### Ce que donne le budget

Chaîne complète, `T_A` = 25 °C extérieurs, interface graisse 0,57 et étalement aluminium 0,411 :

| Modèle | Total | `T_C` atteint | Marge sur les 2,232 |
|---|---|---|---|
| `02/300` | 1,431 | **57,1 °C** | 0,801 °C/W |
| `03/300` | 1,391 | 56,2 °C | 0,841 |
| `04/300` | 1,291 | 53,9 °C | 0,941 |
| `04/400` | 1,211 | 52,1 °C | 1,021 |

**Même le plus petit modèle passe, et il passe largement** : 57 °C de boîtier contre 75 visés. Le critère strict de E1.6 — préserver la validité de toutes les courbes TI — est donc tenable sans rien concéder, ce qui n'était pas acquis il y a une heure.

#### Ce que la marge doit payer : la liaison

Il reste à conduire la chaleur du dessus de `U6` jusqu'au flanc. C'est là que part la marge, et une barre de liaison coûte cher :

| Barre aluminium (`k` = 200) | Résistance |
|---|---|
| 30 mm de long, section 10 × 60 mm | 0,250 °C/W |
| 50 mm, section 10 × 60 | 0,417 °C/W |
| 80 mm, section 12 × 60 | 0,556 °C/W |
| 60 mm, section 6 × 40 | **1,250 °C/W** |

**La leçon est nette : courte et épaisse, ou rien.** Les 0,801 °C/W disponibles avec le `02/300` financent une barre de 50 mm en section 10 × 60, pas une équerre mince de 60 mm en 6 × 40, qui à elle seule dépasserait le budget. Les jonctions supplémentaires, elles, sont négligeables : à la graisse sur 60 × 20 mm, une interface vaut 0,014 °C/W — la pénalité des 29 mm² du PowerPAD ne se paie qu'une fois.

Deux architectures en découlent, et il faut en choisir une **avant** de placer :

- **Carte à plat sur l'embase, `U6` relié au flanc par une barre courte.** Impose `U6` près du bord de carte, côté flanc retenu, et une barre massive. Coût thermique : 0,25 à 0,42 °C/W, finançable.
- **Carte montée verticalement contre le flanc, `U6` pressé dessus.** Chemin thermique le plus court possible, aucune barre, mais impose une hauteur de coffret supérieure à la hauteur de carte — donc 4U si la carte fait 120 mm comme l'EVM — et une maîtrise fine du plan de contact.

#### Le point qu'il ne faut pas rater : la masse

Le PowerPAD est `GND`. Le presser contre un flanc de coffret **relie la masse du signal au châssis**, et par lui à la terre de protection si le coffret y est raccordé. Ce n'est pas anodin sur un amplificateur à entrées asymétriques : c'est la boucle de masse classique. Trois issues, à trancher en même temps que l'architecture mécanique :

- **Assumer châssis = `GND`**, avec un point de masse unique et une liaison à la terre par réseau de découplage. Thermiquement gratuit.
- **Isoler par une céramique haute conductivité.** Un intercalaire **AlN** de 0,5 mm (`k` ≈ 170 W/m·K) coûte 0,101 °C/W sur 29 mm², compatible avec `REQ-THERM-2` et avec les marges ci-dessus. Le silicone reste exclu, il vaudrait 8,61.
- **Isoler la barre du flanc** plutôt que la puce de la barre, sur une surface bien plus grande donc à coût thermique quasi nul.

**Le `NEEDS_DATA` dissipateur/boîtier est levé au sens du critère de sélection** : la famille est identifiée, sourcée, et chiffrée avec marge. Restent trois arbitrages utilisateur — modèle de coffret, architecture mécanique, traitement de la masse — qui conditionnent le contour de carte et donc E1.2.

#### Réserves honnêtes

- Les `RθSA` Modushop sont des **valeurs de catalogue fabricant**, données à 25 °C avec un exemple de montage TO3-P sur mica ; le document ne publie ni la puissance d'essai ni l'élévation de référence. Crédibles, non tracées à un rapport de mesure.
- Le catalogue Boyd « board level » signale que **ses** valeurs en convection naturelle supposent **75 K d'élévation** du dissipateur. Une convection naturelle est d'autant moins efficace que l'élévation est faible ; à 32 K d'élévation, une valeur publiée à 75 K est optimiste d'environ 20 %. La marge dégagée plus haut absorbe cet ordre de grandeur, mais il faut le savoir.
- Le calcul de la barre est une conduction 1-D. Il ignore l'étalement supplémentaire à l'entrée du flanc, où une barre de 60 mm alimente un panneau de 300. Deuxième raison de garder de la marge.
- **Fischer Elektronik est resté inaccessible** — 403 sur toutes les pages produit, y compris avec un en-tête de navigateur, et les miroirs distributeurs ne servent pas de PDF exploitable. Le `SK 47/100/SA` annoncé à 0,45–1,05 °C/W par les distributeurs **n'a pas pu être vérifié à la source primaire** et n'est donc pas retenu comme candidat.

### Arbitrages rendus, et contour qui en découle

Trois choix utilisateur closent E1.7 et ouvrent E1.2 :

1. **Coffret Modushop `03/300`, 3U.** Flancs 300 × 120 × 40 mm à **0,41 °C/W** chacun. Hors tout 120 mm de haut, 300 de profond ; **largeur intérieure entre flancs 360 mm**. Façade aluminium usinée de 10 mm, embase intérieure pré-percée `01/05` en option.
2. **Carte à plat sur l'embase, `U6` en bord de carte, relié au flanc par une barre aluminium courte et massive.** Cible : 30 à 40 mm de long, section 10 × 60 mm, soit 0,25 à 0,33 °C/W.
3. **Isolation électrique reportée à la jonction barre/flanc**, sur environ 60 × 20 mm au lieu des 29 mm² du PowerPAD. La masse signal reste ainsi flottante par rapport au châssis, pour un coût thermique de 0,07 à 0,21 °C/W selon le pad.

Chaîne complète retenue, `T_A` = 25 °C extérieurs :

| Poste | Valeur |
|---|---|
| Interface graisse sur PowerPAD | 0,570 |
| Étalement dans la barre (Al) | 0,411 |
| Barre 30–40 mm, section 10 × 60 | 0,250 à 0,333 |
| Isolation barre/flanc | 0,070 à 0,208 |
| Flanc `03/300` | 0,410 |
| **Total** | **1,711 à 1,932** |
| Budget disponible | **2,232** |

`T_C` atterrit entre **63 et 68 °C**, sous les 75 visés, avec 0,30 à 0,52 °C/W de marge. **Le montage tient, mais la marge n'est plus confortable** : c'est la barre et son isolation qui la consomment. Deux conséquences pour E1.2 — **la barre doit être aussi courte que le placement le permet**, et un pad d'isolation à `k` ≥ 3 W/m·K est préférable à un pad standard.

#### Contour de carte

**200 × 150 mm.** Le raisonnement : l'EVM tient un jeu comparable mais plus léger sur 160 × 120, et cette carte y ajoute tout le bloc de protection 48 V — `Q302` en TO-264, `Q301` en TO-220, le fusible, le shunt et le `LM5069` — ainsi qu'un bulk plus gros. Le coffret offrant 360 × 285 mm utiles, rien n'oblige à serrer : une carte au large se route mieux et se refroidit mieux. **Le contour n'est pas contraint par le coffret, il est contraint par le placement**, et il pourra être resserré une fois E1.5 passée.

Deux points restent à fixer plus tard, faute de données :

- **Perçages de fixation** : l'embase `01/05` est pré-percée, mais son plan de perçage n'est pas publié. Trous à ajouter une fois le plan obtenu ou l'embase mesurée. `NEEDS_DATA` mineur, sans effet sur le placement.
- **Orientation dans le coffret** : entrées RCA, sorties haut-parleur et entrée 48 V sur le panneau arrière ; `J4` vers la façade, où le potentiomètre est monté. `U6` contre un flanc, donc en bord latéral — ce qui impose de séparer l'analogique bas niveau du bord opposé.

### E1.2 — le placement retourne la section de la barre

Trois faits géométriques, établis en plaçant `U6`, obligent à préciser l'orientation de la barre. Ils ne changent pas les chiffres de la chaîne thermique, mais ils changent la pièce.

**1. `U6` ne peut pas être collé au bord.** Le `HTSSOP-44` porte 22 broches de chaque côté, et les deux côtés ne sont pas interchangeables : côté `x < 0` du symbole se trouve **tout le bas niveau** — `INPUT_A`–`D`, `GVDD`, `VDD`, `DVDD`, `AVDD`, `RESET`, `FAULT`, `VBG`, `CLIP_OTW`, `OSC`, `FREQ_ADJ`, `OC_ADJ`, `C_START` — et côté `x > 0` **toute la puissance** : six `PVDD`, `OUT_A`–`D`, quatre `BST`, six `GND`. Le boîtier doit donc être posé à **rotation 180°**, puissance vers l'intérieur de la carte, ce qui laisse le bas niveau échapper vers la lisière droite. Une lisière de moins de 10 mm ne suffirait pas à sortir 22 broches : `U6` est centré en `(285, 175)`, à 10,25 mm du bord.

**2. Une barre couchée stériliserait la lisière.** La face inférieure de la barre repose sur le dessus de `U6`, donc à ≈ **1,2 mm** du circuit imprimé. Une barre de 60 mm de large posée à plat projetterait sur la carte une ombre d'environ **60 × 20 mm** à cette hauteur. Deux conséquences, et la seconde est rédhibitoire : aucun composant plus haut que 1,2 mm n'y tiendrait, et surtout **une pièce d'aluminium nu à 1,2 mm du cuivre** n'est pas acceptable au-dessus de pastilles et de vias.

**3. La section tourne de 90°, pas la surface.** La solution est de dresser la barre : **10 mm d'épaisseur dans le plan de la carte, 60 mm de hauteur**, au lieu de l'inverse. La section de conduction reste **600 mm²** et la longueur reste 30 à 40 mm, donc **les 0,250 à 0,333 °C/W de la barre sont inchangés**. Ce qui change est l'ombre portée : elle tombe à **10 mm de large**, soit `x` de 280 à 300 et `y` de 169 à 181 — une bande qui ne contient que `U6` et des échappées de pistes, **aucun site de composant**. Les pistes qui passent dessous sont couvertes par le vernis épargne et séparées de 1,2 mm d'air ; le placement, lui, tient les composants hors de la bande.

**Réserve à ne pas oublier, et elle coûte de la marge.** Le budget de E1.7 chiffre l'isolation barre/flanc **sur 60 × 20 mm**, soit 1200 mm², d'où 0,070 à 0,208 °C/W. Une barre dressée de 10 mm d'épaisseur ne présenterait au flanc que 600 mm² et **doublerait ce terme**, à 0,14–0,42 °C/W : le total passerait de 1,711–1,932 à **1,78–2,14** pour un budget de 2,232, ce qui tient encore mais ne laisse plus que 0,09 °C/W dans le cas défavorable. **Exigence à porter à la pièce : la barre doit s'élargir en pied à son extrémité côté flanc, pour y présenter au moins 1200 mm² de contact.** Un pied rapporté ou une extrémité usinée en T suffit ; c'est la seule cote de la barre qui n'est pas négociable.

**Contrainte de placement qui en découle, appliquée en E1.2** : la bande `x` ∈ [280, 300], `y` ∈ [169, 181] est une **zone d'interdiction de composants**, pas seulement de hauteur. Les découplages du côté bas niveau sont donc rangés en deux groupes, au-dessus de `y` = 166,5 et au-dessous de `y` = 183,5, et rejoignent leurs broches par des pistes qui passent sous la barre.

### Ce qui ne va pas sur ce dissipateur

- **Les quatre inductances, 3,1 W au total**, dissipent dans le PCB et l'air, pas dans le dissipateur de `U6`.
- `Q301` (0,70 W) et `Q302` peuvent partager le même dissipateur ou en avoir un propre. Leur contribution en régime établi est inférieure à 1 W et ne change pas les valeurs ci-dessus. Le besoin de `Q302` n'est d'ailleurs pas un régime établi mais **transitoire** : sa SOA suppose le boîtier à 75 °C au moment de l'événement, ce qu'un dissipateur garantit en maintenant basse la température de départ.
- ~~`NEEDS_DATA: traitement de la broche 45 (PowerPad) du symbole.`~~ — **tranché en D1.3, comme le demandait cette note.** La broche 45 **reste câblée à `GND` au schéma**, ce qui est correct au sens de TI : le PowerPAD est bien une masse, simplement raccordée par le dissipateur et non par le PCB. L'empreinte garde ses 44 pastilles, conformes au land pattern. **Conséquence à connaître avant E1.2 : l'import vers le PCB signalera une broche sans pastille. C'est attendu et ce n'est pas un défaut à corriger** ; supprimer la broche du symbole serait au contraire une erreur, elle documente une liaison électrique réelle.

### Références validées en E1.7

**`SLPX472M080H3P3`, `C316`/`C317`.** Décodée champ par champ contre la clé du catalogue Cornell Dubilier, puis recoupée sur la ligne du tableau :

| Champ | Lecture |
|---|---|
| `SLPX` | série snap-in **85 °C**, 3000 h — **et non `SLP`, qui est la série 105 °C** |
| `472` | 4700 µF |
| `M` | ±20 % |
| `080` | 80 Vdc |
| `H3` | **ø35 × 30 mm** |
| `P` | polarisé |
| `3` | manchon PVC ou PET |

La ligne du tableau donne ESR ≤ **0,071 Ω** à 25 °C et une ondulation admissible d'au moins **4,12 Arms** à 85 °C. **La cote de 30 mm portée depuis D1 est confirmée**, ce qui n'allait pas de soi : le catalogue `SLP`, ouvert en premier, ne propose en 4700 µF/80 V que du ø30 × 45 et du ø35 × 35, **tous deux plus hauts**. C'est bien la série `SLPX` qui livre le boîtier court.

**Point à porter au dossier** : `SLPX` étant une série **85 °C** et non 105 °C, sa durée de vie garantie est de 3000 h à 85 °C. À 40 °C d'ambiante interne l'extrapolation usuelle donne un ordre de grandeur très supérieur, mais **le poste mérite d'être revu en H1** si l'ambiante interne réelle s'écarte de l'hypothèse.

## NEEDS_DATA avant gel final

- `NEEDS_DATA: référence et caractéristiques garanties de l’alimentation externe 48 V ; nécessaires pour ripple, fusible, bulk, connecteur et puissance continue.` **Partiellement cadré en D1.13** : le volet tension est désormais borné par `REQ-PSU-1` (sortie ≤ 53,5 V en toutes conditions), qui devient un critère de sélection et non plus une donnée manquante. Restent ouverts le ripple, le courant continu garanti et le comportement au démarrage.
- `NEEDS_DATA: choix mécanique du potentiomètre double 10 kΩ logarithmique ; nécessaire pour empreinte et durée de vie.`
- ~~`NEEDS_DATA: références exactes des inductances 15 µH`~~ — **levé en D1.10.** `PA6331-AE` Coilcraft, même famille que le `MA5172-AE` de la nomenclature EVM : 15 µH, DCR 31 mΩ, `I_sat` 20 A, `I_rms` 9,8 A. Tore traversant debout ø28,6 × 12,3 mm, empreinte locale créée. Reste à reconfirmer l'approvisionnement en H2.
- ~~`NEEDS_DATA: référence exacte des condensateurs 680 nF de sortie.`~~ — **levé en E1.7 à la source primaire.** Le catalogue WIMA MKP 4 porte la ligne `MKP4F036804F00_ _ _ _` à 0,68 µF, cotes **L 18 × W 8 × H 15 mm, pas 15 mm** : la boîte figée en D1.9 est confirmée au document fabricant. **Les quatre caractères manquants ne sont pas une donnée à trouver** : le catalogue précise qu'ils complètent la référence par la **tolérance de capacité** et le **conditionnement**, choisis à la commande. Il n'y a donc plus rien à décoder, seulement à trancher en H2. *Réserve* : la lettre `F` désigne la colonne 250 VDC / 160 VAC du tableau, lecture déduite de la position de colonne et non d'une clé explicite.
- `NEEDS_DATA: protection 48 V inversion/surtension et TVS ; le clamp doit rester compatible avec le maximum absolu TPA3255.` **Les composants sont figés — `F301`, `D301` = `SMDJ58CA`, `Q301` = `IPP330P10NM`, `U8` = `LM5069-2`, `Q302` = `IXTK200N10L2` — mais l'exigence telle qu'elle est écrite n'est pas satisfaite et ne peut pas l'être.** Aucune TVS du commerce ne clampe sous le maximum absolu du TPA3255 : le facteur de clamp exigé vaut 1,19 à 1,23 pour 1,3 minimum offert par la technologie, et ce constat ne change pas avec le maximum absolu rectifié à 69 V — il empire, cf. `docs/protection-48v.md`. **La réponse du projet est architecturale et non composant** : la TVS est cantonnée aux transitoires rapides, la surtension soutenue est coupée activement par le `LM5069`, et le respect des conditions recommandées vient de `REQ-PSU-1`. À reformuler en exigence de vérification plutôt qu'en donnée manquante.
- `NEEDS_DATA: common-mode garanti du TPA3255 ; non spécifié explicitement, mitigé par les condensateurs de liaison EVM.`
- `NEEDS_DATA: dissipateur, pression/interface thermique, boîtier, ventilation et température ambiante.` **Trois des cinq volets sont clos.** L'**interface** est spécifiée par `REQ-THERM-2` — graisse, graphite ou indium, pad silicone exclu ; l'absence d'isolation électrique à assurer, le PowerPAD étant `GND` et le dissipateur porté au même potentiel, autorise le contact direct et rend cette exigence tenable. La **ventilation** est close par `REQ-THERM-3` : convection naturelle, 4 Ω en crête seulement. L'**ambiante** est fixée à 40 °C internes, hypothèse de calcul de `REQ-THERM-1`. Restent le **dissipateur** et le **boîtier** eux-mêmes, dont les cotes conditionnent E1.2, et la **pression de montage**, qui ne se vérifiera qu'au prototype.
- ~~`NEEDS_DATA: valeur du condensateur de découplage VMID (C110)`~~ — **levé en D1.5 par le calcul, aucune source extérieure n'était nécessaire.** `VMID` vaut `+12V-OA` / 2 = 6 V, produit par `R105`/`R106` de 10,0 kΩ, donc une source de Thévenin de 5 kΩ qui polarise quatre entrées non inverseuses (`U4` broches 3 et 5, `U5` broches 3 et 5). Ce sont **le bruit thermique et la réjection de rail** qui fixent la capacité, pas une valeur de datasheet : 5 kΩ produisent 9 nV/√Hz, soit huit fois le bruit propre de l'OPA1612, et ce bruit passe en entier dans le gain non inverseur s'il n'est pas court-circuité. `C110` et `C210` sont en parallèle sur `VMID`, 10 µF chacun, pour un coude à 1,6 Hz nominal et environ 3 Hz une fois pris le déclassement sous 6 V continus — sous la bande audio dans les deux cas. La tension nominale est portée à 25 V précisément pour contenir ce déclassement. Deux condensateurs et non un seul : `U4` et `U5` sont éloignés, chacun doit avoir le sien au plus près. Le temps d'établissement qui en découle, 5 τ ≈ 250 ms, reste à croiser avec la temporisation de mute en Phase F.
- `NEEDS_DATA: réponse/EMI du filtre LC, stabilité toutes charges et performance OPA1612 dans cette topologie ; validation par simulation ciblée puis prototype/mesure.`

- `NEEDS_DATA: tension absolue maximale de la broche MR du TPS3802K33 et caractéristique de montée de PVDD au power-up ; la chaîne EVM PVDD → R6 100 kΩ → C83 1 µF → RESET-SW couple MR au rail 48 V. Le continu est bloqué par C83, mais la contrainte transitoire au démarrage n'est pas bornée sans ces deux données.`

- `NEEDS_DATA: plan de perçage de l'embase Modushop 01/05` — non publié par le fabricant ; à relever sur pièce.
- `NEEDS_DATA: fabricant de PCB et épaisseur de cuivre par couche.` Le MCP n'expose pas l'épaisseur de cuivre et le fichier ne porte pas de bloc `stackup` explicite : à rendre explicite dans Board Setup **et** à la commande.
- `NEEDS_DATA: décodage à la source de l'EEU-FC1J152` (`C312`–`C315`) — `industrial.panasonic.com` refuse `curl`.

Ces points interdisent actuellement `PRÊT À FABRIQUER = OUI`, mais n’empêchent pas la capture schématique initiale si les composants non figés sont explicitement marqués.


## Contraintes de placement encore actives

Elles survivent à la tâche qui les a produites et gouvernent tout placement restant.

- **`U6` se refroidit uniquement par le dessus**, aucun via thermique sous le boîtier. La
  broche 45 du symbole n'a pas de pastille dans `HTSSOP-44_…_TopEP` : l'erreur d'import à ce
  sujet est **attendue et correcte**.
- **Orientation dans le coffret** : `U6` contre le flanc droit ; analogique bas niveau au bord
  opposé ; sorties haut-parleur et entrée 48 V sur l'arête arrière ; `J4` vers la façade.
  `J2`/`J3`/`J4` sont en **JST XH déporté**, leur panneau est donc un choix de câblage et non
  une contrainte de carte.
- **Zone d'interdiction de composants sous la barre** : `x` ∈ [280, 291], `y` ∈ [160, 190] au
  pied élargi, et `x` ∈ [280, 300], `y` ∈ [169, 181] au-delà. Ce n'est pas seulement une limite
  de hauteur — une pièce d'aluminium nu à 1,2 mm du cuivre n'est pas acceptable au-dessus de
  pastilles. Les découplages bas niveau se rangent donc au-dessus de `y` = 166,5 et au-dessous
  de `y` = 183,5.
- **Hauteurs** : tores `L301`–`L304` debout ø 28,6 × 29 mm, 3,1 W de pertes cuivre ; films
  `C321`–`C324` 18 × 8 × 15 mm ; `C312`–`C315` ø 18 × 35 mm ; `C316`/`C317` ø 35 × **30 mm** ;
  `C325` 18 mm.
- **`R306` est un shunt à deux bornes, pas Kelvin** : `VIN`/`SENSE` se prennent sur les bords
  **intérieurs** des pastilles.
- `Q302` en TO-264 sur radiateur, SOA supposant le boîtier à 75 °C. `Q301` en TO-220 : 6,9 A
  avec **6 cm² de cuivre 70 µm sur son net de drain**.
- `C110`/`C210` : établissement de `VMID` en 5 τ ≈ 250 ms, à croiser avec le mute en Phase F.
- Asymétrie de nommage assumée `-VSE`/`+VSE` à gauche, `-VSE_R`/`+VSE_R` à droite, à trancher
  avant H2. Bulk maintenu à 15 400 µF.
