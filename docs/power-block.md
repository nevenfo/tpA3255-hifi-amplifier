# Bloc d’alimentation auxiliaire — références vérifiées

Sources fabricant : schéma TI TPA3255EVM `SLAR129A`, BOM TI `SLAR130A`, guide `SLOU441` et datasheets des composants. Ces choix couvrent la chaîne auxiliaire ; ils ne figent pas la protection amont 48 V.

| Fonction | Référence exacte | Boîtier / valeur |
|---|---|---|
| Buck PVDD → +15 V | `LM5010ASD/NOPB` | TI `DPR0010A`, WSON-10, PowerPAD broche 11 |
| Diode de roue libre | `SK310A-TP` | SMA, Schottky 100 V/3 A |
| Diode SS/ISEN | `B1100-13-F` | SMA, Schottky 100 V/1 A |
| Inductance buck | `7447714101` | 100 µH, 1.5 A, DCR 0.165 Ω |
| LDO +15 V → +12 V | `LM2940IMP-12/NOPB` | TI `MP04A`, SOT-223-4 |
| LDO +12 V → +3.3 V | `TLV1117-33IDCY` | TI `DCY0004A`, SOT-223-4 |
| Superviseur 3.3 V | `TPS3802K33DCKR` | TI `DCK0005A`, SC-70-5, RESET actif bas, délai 400 ms |

Réseau LM5010A repris de l’EVM : `R2=182 kΩ`, `C7=5.6 nF`, `R39=4.99 kΩ`, `C12=4.7 nF`, `R40=1.00 kΩ`, `C39=47 µF/63 V`, `C3=1 µF/100 V`, `C11=10 nF/100 V`, `C4=2.2 µF/100 V`, `C2=100 nF/100 V`, `C1=47 nF/25 V`, `C13=100 nF/50 V`. Le découplage aval conserve `C5=47 µF/16 V`, `C8=470 nF/25 V`, `C9=100 nF/50 V`, `C38=10 µF/16 V`, `C10=100 µF/6.3 V`, `C67=100 nF/50 V` et `R26=3.30 kΩ` autour du superviseur.

Connexions critiques LM5010A : broches `1=SW`, `2=BST`, `3=ISEN`, `4=SGND`, `5=RTN`, `6=FB`, `7=SS`, `8=RON/SD`, `9=VCC`, `10=VIN`, `11=PowerPAD` à GND. `TLV1117-33IDCY` : `1=GND`, `2+tab=OUT`, `3=IN`.

La protection amont 48 V reste hors de ce gel ; son inconnue critique est centralisée dans la section `NEEDS_DATA` de `docs/architecture.md`.

Sources :

- <https://www.ti.com/lit/df/slar129a/slar129a.pdf>
- <https://www.ti.com/lit/df/slar130a/slar130a.pdf>
- <https://www.ti.com/lit/ug/slou441/slou441.pdf>
- <https://www.ti.com/lit/ds/symlink/lm2940c.pdf>
- <https://www.ti.com/lit/ds/symlink/tlv1117.pdf>
- <https://www.ti.com/lit/gpn/tps3802>
