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

## Puissance et mode TPA3255

- `U_PWR = TPA3255DDV`, HTSSOP-44 `DDV`, PowerPAD supérieur destiné au couplage à un dissipateur et à la masse selon TI.
- Mode `BTL stéréo`, `M1=0`, `M2=0`.
- Canal gauche : charge entre `OUT_A` et `OUT_B`.
- Canal droit : charge entre `OUT_C` et `OUT_D`.
- `PBTL` est exclu : c’est un mode mono et ne répond pas au besoin stéréo.
- `PVDD` nominal 48 V, dans la plage TI 18–53.5 V. L’option jusqu’à 56.5 V pour charge ≥6 Ω n’est pas utilisée, puisque la carte doit accepter 4 Ω.
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
- PowerPAD et masse reliés à une matrice de vias thermiques et à un dispositif dissipateur conforme au boîtier DDV.

## NEEDS_DATA avant gel final

- `NEEDS_DATA: référence et caractéristiques garanties de l’alimentation externe 48 V ; nécessaires pour ripple, fusible, bulk, connecteur et puissance continue.`
- `NEEDS_DATA: choix mécanique du potentiomètre double 10 kΩ logarithmique ; nécessaire pour empreinte et durée de vie.`
- `NEEDS_DATA: références exactes des inductances 15 µH et condensateurs 680 nF ; nécessaires pour saturation, DCR, pertes et empreintes.`
- `NEEDS_DATA: protection 48 V inversion/surtension et TVS ; le clamp doit rester compatible avec le maximum absolu TPA3255.`
- `NEEDS_DATA: common-mode garanti du TPA3255 ; non spécifié explicitement, mitigé par les condensateurs de liaison EVM.`
- `NEEDS_DATA: dissipateur, pression/interface thermique, boîtier, ventilation et température ambiante.`
- `NEEDS_DATA: valeur du condensateur de découplage VMID (C110) ; A1 exige un point milieu fortement découplé mais aucune source ne fixe la capacité.`
- `NEEDS_DATA: réponse/EMI du filtre LC, stabilité toutes charges et performance OPA1612 dans cette topologie ; validation par simulation ciblée puis prototype/mesure.`

- `NEEDS_DATA: tension absolue maximale de la broche MR du TPS3802K33 et caractéristique de montée de PVDD au power-up ; la chaîne EVM PVDD → R6 100 kΩ → C83 1 µF → RESET-SW couple MR au rail 48 V. Le continu est bloqué par C83, mais la contrainte transitoire au démarrage n'est pas bornée sans ces deux données.`

Ces points interdisent actuellement `PRÊT À FABRIQUER = OUI`, mais n’empêchent pas la capture schématique initiale si les composants non figés sont explicitement marqués.
