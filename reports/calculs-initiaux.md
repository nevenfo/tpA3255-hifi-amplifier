# Calculs initiaux — 2026-08-27

## Niveau de signal à 100 W/8 Ω

```text
Vout_rms = sqrt(P × R) = sqrt(100 × 8) = 28.284 Vrms
G_TPA = 10^(21.5/20) = 11.885 V/V
Vin_diff = 28.284 / 11.885 = 2.380 Vrms
Vin_SE_après_volume = 2.380 / 2 = 1.190 Vrms
Atténuation depuis 2 Vrms = 20 log10(1.190/2) = -4.51 dB
Iout_rms = sqrt(P/R) = sqrt(100/8) = 3.536 Arms
Iout_peak = sqrt(2) × Iout_rms = 5.000 A
```

Les deux inverseurs `−1` en cascade donnent deux branches opposées de 1.190 Vrms, donc 2.380 Vrms différentiels. À l’entrée nominale EVM de 2 Vrms SE, chaque broche reçoit 2 Vrms, soit 5.657 Vpp, sous la limite TI de 7 Vpp par broche ; le différentiel vaut alors 4 Vrms.

## Budget DC indicatif

Pour 200 W audio et une hypothèse de calcul de 90 % d’efficacité :

```text
Pin_puissance = 200 / 0.90 = 222.2 W
Paux_budget = 10 W
I48_moyen = (222.2 + 10) / 48 = 4.84 A
```

Une alimentation 48 V/10 A est retenue comme exigence transitoire de conception, pas comme preuve de puissance continue ni de comportement musical.

## Coupure d’entrée

Avec `Cin=4.7 µF` et `R=10 kΩ` :

```text
fc = 1/(2πRC) = 3.39 Hz
|H(20 Hz)| = 0.986, soit -0.12 dB
```

## Filtre LC idéal

Avec `L=15 µH` et `C=680 nF` :

```text
fc = 1/(2πsqrt(LC)) = 49.83 kHz
```

Ce résultat n’est pas une preuve de réponse réelle BTL, de stabilité, d’EMI ou de comportement sous charge complexe.

## Statut des calculs

- Vérifié par calcul : niveaux RMS, gain, courants idéaux, budget DC indicatif, coupures idéales.
- Non simulé : réponse complète du filtre, impédance haut-parleur, phase, amortissement, transitoires et stabilité.
- Non mesuré : THD+N, SNR, EMI/EMC, thermique et puissance continue.
