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
- ~~`NEEDS_DATA: traitement de la broche 45 (PowerPad) du symbole.`~~ — **tranché en D1.3, comme le demandait cette note.** La broche 45 **reste câblée à `GND` au schéma**, ce qui est correct au sens de TI : le PowerPAD est bien une masse, simplement raccordée par le dissipateur et non par le PCB. L'empreinte garde ses 44 pastilles, conformes au land pattern. **Conséquence à connaître avant E1.2 : l'import vers le PCB signalera une broche sans pastille. C'est attendu et ce n'est pas un défaut à corriger** ; supprimer la broche du symbole serait au contraire une erreur, elle documente une liaison électrique réelle.

## NEEDS_DATA avant gel final

- `NEEDS_DATA: référence et caractéristiques garanties de l’alimentation externe 48 V ; nécessaires pour ripple, fusible, bulk, connecteur et puissance continue.` **Partiellement cadré en D1.13** : le volet tension est désormais borné par `REQ-PSU-1` (sortie ≤ 53,5 V en toutes conditions), qui devient un critère de sélection et non plus une donnée manquante. Restent ouverts le ripple, le courant continu garanti et le comportement au démarrage.
- `NEEDS_DATA: choix mécanique du potentiomètre double 10 kΩ logarithmique ; nécessaire pour empreinte et durée de vie.`
- ~~`NEEDS_DATA: références exactes des inductances 15 µH`~~ — **levé en D1.10.** `PA6331-AE` Coilcraft, même famille que le `MA5172-AE` de la nomenclature EVM : 15 µH, DCR 31 mΩ, `I_sat` 20 A, `I_rms` 9,8 A. Tore traversant debout ø28,6 × 12,3 mm, empreinte locale créée. Reste à reconfirmer l'approvisionnement en H2.
- `NEEDS_DATA: référence exacte des condensateurs 680 nF de sortie.` **Partiellement levé en D1.9** : le diélectrique, la tension et la **boîte sont figés** — WIMA MKP4, 18 × 8 mm sur 15 de haut, pas de 15 mm, empreinte standard attribuée. Ne manquent que les quatre derniers caractères de la référence `MKP4F036804F00`, qui codent tolérance et conditionnement. Sans effet sur l'implantation ; à clore en H2 avec la BOM.
- `NEEDS_DATA: protection 48 V inversion/surtension et TVS ; le clamp doit rester compatible avec le maximum absolu TPA3255.` **Les composants sont figés — `F301`, `D301` = `SMDJ58CA`, `Q301` = `IPP330P10NM`, `U8` = `LM5069-2`, `Q302` = `IXTK200N10L2` — mais l'exigence telle qu'elle est écrite n'est pas satisfaite et ne peut pas l'être.** Aucune TVS du commerce ne clampe sous le maximum absolu du TPA3255 : le facteur de clamp exigé vaut 1,19 à 1,23 pour 1,3 minimum offert par la technologie, et ce constat ne change pas avec le maximum absolu rectifié à 69 V — il empire, cf. `docs/protection-48v.md`. **La réponse du projet est architecturale et non composant** : la TVS est cantonnée aux transitoires rapides, la surtension soutenue est coupée activement par le `LM5069`, et le respect des conditions recommandées vient de `REQ-PSU-1`. À reformuler en exigence de vérification plutôt qu'en donnée manquante.
- `NEEDS_DATA: common-mode garanti du TPA3255 ; non spécifié explicitement, mitigé par les condensateurs de liaison EVM.`
- `NEEDS_DATA: dissipateur, pression/interface thermique, boîtier, ventilation et température ambiante.`
- ~~`NEEDS_DATA: valeur du condensateur de découplage VMID (C110)`~~ — **levé en D1.5 par le calcul, aucune source extérieure n'était nécessaire.** `VMID` vaut `+12V-OA` / 2 = 6 V, produit par `R105`/`R106` de 10,0 kΩ, donc une source de Thévenin de 5 kΩ qui polarise quatre entrées non inverseuses (`U4` broches 3 et 5, `U5` broches 3 et 5). Ce sont **le bruit thermique et la réjection de rail** qui fixent la capacité, pas une valeur de datasheet : 5 kΩ produisent 9 nV/√Hz, soit huit fois le bruit propre de l'OPA1612, et ce bruit passe en entier dans le gain non inverseur s'il n'est pas court-circuité. `C110` et `C210` sont en parallèle sur `VMID`, 10 µF chacun, pour un coude à 1,6 Hz nominal et environ 3 Hz une fois pris le déclassement sous 6 V continus — sous la bande audio dans les deux cas. La tension nominale est portée à 25 V précisément pour contenir ce déclassement. Deux condensateurs et non un seul : `U4` et `U5` sont éloignés, chacun doit avoir le sien au plus près. Le temps d'établissement qui en découle, 5 τ ≈ 250 ms, reste à croiser avec la temporisation de mute en Phase F.
- `NEEDS_DATA: réponse/EMI du filtre LC, stabilité toutes charges et performance OPA1612 dans cette topologie ; validation par simulation ciblée puis prototype/mesure.`

- `NEEDS_DATA: tension absolue maximale de la broche MR du TPS3802K33 et caractéristique de montée de PVDD au power-up ; la chaîne EVM PVDD → R6 100 kΩ → C83 1 µF → RESET-SW couple MR au rail 48 V. Le continu est bloqué par C83, mais la contrainte transitoire au démarrage n'est pas bornée sans ces deux données.`

Ces points interdisent actuellement `PRÊT À FABRIQUER = OUI`, mais n’empêchent pas la capture schématique initiale si les composants non figés sont explicitement marqués.
