# H1 — Revue électrique indépendante (H1.1–H1.3)

Base : `HifiAmp_TPA3255.kicad_sch` MD5 `9bcba85d947b78af82475f58438dcdcb`, PCB MD5 `4f4b36822efc7ac6eb330008314dde87`. Valeurs et nets lus dans la netlist `kicad-cli sch export netlist` (écrite hors projet), recoupés par `get_net_connections` (MCP). Lecture seule. Cette revue est distincte d'ERC/DRC.

Classes de preuve : **[K]** prouvée KiCad (netlist/champs Value) ; **[D]** documentation fabricant déjà dans `docs/` (non relue ici à la source) ; **[C]** calcul à partir de [K]+[D] ; **[NV]** non vérifié physiquement. Toute conclusion électrique est au moins [C] et [NV].

## 0. Synthèse des écarts

| Id | Classe | Écart |
|---|---|---|
| B1 | **BLOQUANT** | Étage d'entrée OPA1612 : aucun condensateur entre le curseur du volume (`VOL_L`/`VOL_R`) et `R101`/`R201`. Le DC d'entrée vaut 0 V (bas du potentiomètre à `GND`) alors que les entrées + sont à `VMID` = 6 V : les sorties saturent (§2.1). |
| A1 | À corriger / trancher | `C7` (5,6 nF en parallèle de `R39`, réseau FB du LM5010A) documenté (`power-block.md`, `architecture.md`) mais absent du schéma ; aucune justification dans `plan.md`. |
| A2 | À corriger / trancher | Pas de réseau d'amortissement/EMI de sortie (10 nF + 3,3 Ω et 1 nF de la Fig. 29 de `SLASEA8A`, d'après `architecture.md`) : seulement L + C. Déjà sous `NEEDS_DATA` EMI/stabilité, mais absent du schéma. |
| A3 | À surveiller | Rail `+12V-OA` : LC `L6` 10 µH + ≈ 30 µF, f0 ≈ 9 kHz dans la bande audio, Q non borné (§2.6). |
| A4 | À surveiller | `C39` et `C312`–`C315` (63 V) : seuil OV pire cas du LM5069 59,8 V = 95 % du calibre ; `REQ-PSU-1` (≤ 53,5 V) porte la marge. |
| A5 | À surveiller | Charge d'entrée 5 kΩ au volume maximal (§1.4) ; `power-block.md` dit encore `C13` = 100 nF (schéma : 1 µF/25 V, décision `plan.md` C2, acceptée). |
| X1 | Accepté (documenté) | 4 Ω : −2,1 dB à 20 kHz (Q = 0,43) et régime de crête (`REQ-THERM-3`) ; `Q301` 9,3 A de crête > 6,9 A convectif ; TVS 93,6 V > 88 V (instantané) ; sensibilité 1,19 Vrms pour 100 W. |

## 1. Chaîne analogique

### 1.1 Topologie réelle [K]
RCA → `C101` 4,7 µF → `VOL_L_IN` → `J4` (potentiomètre déporté 10 kΩ log, bas à `GND`) → `VOL_L` → `R101` 10,0 k 0,1 % → `U4A` inverseur (`R102` 10,0 k + `C102` 22 pF C0G en réaction, `+` à `VMID`) → `-VSE` → `R103` 10,0 k → `U4B` inverseur (`R104` 10,0 k + `C103` 22 pF) → `+VSE`. Sorties : `C104`/`C105` 10 µF → `R107`/`R108` 100 Ω → `INPUT_A`/`INPUT_B` (`U6` 5/6) avec `C106`/`C107` 100 pF à `GND`. Voie droite identique (`U5`, 2xx). `VMID` = `R105`/`R106` 10,0 k 1 % (5 kΩ), `C110`+`C210` 10 µF/25 V. Conforme à `architecture.md` pour toutes les valeurs [K].

### 1.2 Gains [C]
| Étage | Formule | Valeur | Intention |
|---|---|---|---|
| Volume | atténuation ≤ 1 | 0 à −∞ | idem |
| U4A / U5A | −R102/R101 | −1 (0 dB) | −1 ✓ |
| U4B / U5B | −R104/R103 | −1 | −1 ✓ |
| Différentiel | `+VSE − (−VSE)` | 2 × Vvolume (+6,02 dB) | ✓ |
| TPA3255 | 10^(21,5/20) [D] | 11,885 V/V | 11,89 ✓ |
| Total (volume max) | 2 × 11,885 | 23,77 V/V = 27,5 dB | — |

Niveau d'entrée pour pleine puissance (au curseur du volume, SE) :
- 100 W/8 Ω : V = √(100·8) = 28,28 Vrms (40,0 Vpk, 5,00 A crête, 3,54 A rms) ; Vdiff = 28,28/11,885 = 2,380 Vrms ; **Vvolume = 1,190 Vrms** ; depuis 2 Vrms : −4,51 dB ✓ (docs −4,5 dB).
- 100 W/4 Ω : 20,0 Vrms (28,3 Vpk, 7,07 A crête) ; Vvolume = 0,841 Vrms ; depuis 2 Vrms −7,5 dB.
- 150 W/4 Ω (ligne 300 W total de la courbe) : 24,5 Vrms, 8,66 A crête.
- Sensibilité à volume max : 1 Vrms → 23,8 Vrms = 70,6 W/8 Ω ; −10 dBV (0,316 Vrms) → 7,1 W. **Les 100 W exigent ≥ 1,19 Vrms de source** (non explicité dans les docs ; accepté, X1).
- Limite d'entrée TPA3255 [D] : 7 Vpp par broche = 2,47 Vrms par branche ; le TPA3255 écrête avant l'AOP (confirme `architecture.md`).

### 1.3 Headroom AOP [C]
Rail `+12V-OA` ≈ 12 V (U2 − L6), `VMID` 6 V. Par branche à 2 Vrms : 6 ± 2,83 V = 3,17–8,83 V, soit **3,17 V de marge aux deux rails** (docs ✓). La plage de mode commun d'entrée (2–10 V) est sans objet : les entrées + sont fixées à 6 V (montage inverseur). Charge de `-VSE` : `R102`‖`R103`‖(`R107`+20 kΩ) ≈ 4,0 kΩ ; de `+VSE` ≈ 6,7 kΩ ; les docs ne justifient le swing que sous 10 kΩ. `NEEDS_DATA` : OPA1612 (SBOS450C) Voh/Vol à 12 V simple sous 4 kΩ et 6,7 kΩ.

### 1.4 Impédances [C]
- Impédance vue par le TPA3255 : 100 Ω (`R107`) + Zout AOP (≪ 100 Ω en boucle fermée, non chiffré) ‖ 100 pF ; face à 20 kΩ [D] → erreur 0,5 % (−0,04 dB) ✓.
- Entrée de l'étage 1 : `R101` 10 kΩ (masse virtuelle). Le curseur charge le potentiomètre de 10 kΩ par 10 kΩ. Zin de la source (C101 + pot) = (1−x)Rp + (xRp‖10k) : x=1 → **5,0 kΩ** ; x=0,9 → 5,7 k ; 0,5 → 8,3 k ; 0,1 → 9,9 k. Impédance de source du curseur max 2,5 kΩ (x=0,5).
- Déformation de la loi par la charge (loi linéaire équivalente) : x=0,5 → −7,96 dB au lieu de −6,02 dB ; x=0,9 → −1,66 vs −0,92 dB ; x=0,1 → −20,8 vs −20 dB. La loi réelle dépend de la piste (`NEEDS_DATA` référence du potentiomètre).
- Écart avec la docs : « charge nominale 10 kΩ » vaut 5 kΩ au maximum (A5). `NEEDS_DATA` : impédance de sortie des sources visées.

### 1.5 Coupures [C]
| Pôle | Formule | f |
|---|---|---|
| `C101` | 1/(2π·Zin·4,7 µF) | 3,4 Hz (10 kΩ) à 6,8 Hz (5 kΩ) → −0,12 à −0,47 dB @20 Hz |
| `C104`/`C105` (et 20x) | 1/(2π·20,1 kΩ·10 µF) | 0,79 Hz (−0,007 dB @20 Hz) |
| `VMID` (`C110`+`C210`) | 1/(2π·5 kΩ·20 µF) | 1,6 Hz ✓ docs |
| Réaction `R102‖C102` (`R104‖C103`) | 1/(2π·10 kΩ·22 pF) | 723 kHz, 2 pôles = −0,007 dB @20 kHz |
| Sortie `R107·C106` | 1/(2π·100·100 pF) | 15,9 MHz |

Réponse 20 Hz–20 kHz de la chaîne analogique (hors B1) : plate à ±0,5 dB, limitée en bas par `C101` ; le correctif B1 (pôle 1,6 Hz pour 10 µF) est négligeable.

### 1.6 Bruit (ordre de grandeur) [C]/[NV]
Bruit thermique 10 kΩ = 12,9 nV/√Hz. Étage 1 (Rin = 10 k + curseur 2,5 k, en = 1,1 nV/√Hz déduit de `architecture.md`, gain de bruit 2) : 19,5 nV/√Hz ; étage 2 : 18,4 ; différentiel (le bruit de l'étage 1 s'additionne en phase) : 43 nV/√Hz → 72 µV rms en sortie dans 20 kHz (×11,885), soit ≈ 112 dB sous 28,28 V non pondéré, hors bruit propre du TPA3255. `NEEDS_DATA` : en/in de l'OPA1612 (SBOS450C), bruit de sortie du TPA3255 (SLASEA8A).

## 2. Points détaillés

### 2.1 B1 — saturation DC de l'étage d'entrée [K]+[C]
Netlist : `VOL_L` = {`J4.2`, `R101.1`} seulement ; `J4.3` = `GND`. `C101` est avant le potentiomètre : le curseur est donc à 0 V DC. Pour U4A : `Vout = Vmid(1 + R102/R101) − (R102/R101)·Vin = 12 V − 0 = 12 V`, soit le rail : l'AOP sature (nœud inverseur à Vout/2 ≈ 5,9 V < VMID, boucle ouverte). U4B reçoit ≈ 11,9 V sur `R103` : `Vout = 6 − (11,9 − 6) ≈ 0 V`, saturé au rail bas. Le DC est bloqué par `C104`, mais aucun signal audio n'est transmis ; même chose voie droite (`R201`). L'ERC ne peut pas le voir. Démonstration algébrique, non simulée.
Correctifs possibles (à trancher, non appliqués) : (a) condensateur 10 µF (film/non polarisé) en série entre `VOL_x` et `R101`/`R201` (gain DC unitaire, pôle 1,6 Hz) ; (b) bas du potentiomètre sur `VMID` (peu souhaitable : `VMID` sur câble déporté). L'option (a) change la netlist et exige une propagation au PCB (GUI).

### 2.2 Filtre LC de sortie [C]
L301–L304 15 µH, C321–C324 680 nF vers `GND` [K]. f0 = 1/(2π√(15 µH·680 nF)) = **49,83 kHz** ✓ (docs 49,8). Vue différentielle : L = 30 µH, C = 340 nF, Z0 = 9,39 Ω ; équivalente par branche (charge R/2, Z0 = 4,70 Ω).

| Charge | Q = R/Z0 | \|H\| 1/10/20/30/50/100 kHz (DCR 31 mΩ) | 450 kHz |
|---|---|---|---|
| 8 Ω | 0,85 | −0,07 / +0,03 / +0,26 / +0,35 / −1,47 / −11,7 dB | −38,2 dB |
| 4 Ω | 0,43 | −0,14 / −0,70 / −2,10 / −3,87 / −7,5 / −15,0 dB | −38,4 dB |

Surtension de 0,44 dB à ≈ 28 kHz sur 8 Ω (Q > 0,707) ; −2,1 dB à 20 kHz sur 4 Ω (accepté, X1). Fréquence PWM : `R301` 22,0 kΩ = intention 450 kHz [D] ; la table fs = f(R_FREQ_ADJ) de `SLASEA8A` n'est pas dans `docs/` → `NEEDS_DATA`. Atténuation = 1/(x²−1), x = 450/49,83 = 9,03.
Courant d'ondulation (hypothèse modulation BD, au repos : sortie 0↔48 V à 50 %, cap à 24 V) : ΔI_pp = 24·0,5/(15 µH·450 kHz) = 1,78 A ; Irms par condensateur 0,51 A ; ΔV_pp = 0,73 V. `NEEDS_DATA` : mode de modulation exact du TPA3255 (BD/1SPW) et courant d'ondulation admissible du WIMA MKP4 680 nF/250 V.
Selfs `PA6331-AE` [D] : Isat 20 A, Irms 9,8 A, DCR 31 mΩ. Pic 8 Ω/100 W : 5,0 + 0,9 = 5,9 A ; 4 Ω/100 W : 8,0 A ; seuil OC 17,0 A + 0,9 = 17,9 A < 20 A (marge 1,12). Cuivre : 4·I²·0,031 = 1,55 W (8 Ω) / 3,1 W (4 Ω) ; pertes fer à 450 kHz `NEEDS_DATA`.

### 2.3 Courants et protection 48 V [C]/[D]
PVDD (rendements de la fig. 10 de `SLASEA8A` repris dans `architecture.md`) : 2×100 W/8 Ω : 222,4 W → **4,63 A** moyen (rendement 89,9 %) ; 4 Ω : 237,4 W → 4,95 A (84,2 %) ; 2×150 W/4 Ω : 353,9 W → 7,37 A. La crête de **9,3 A** de la doc n'est pas reproductible depuis les données du dossier : acceptée comme enveloppe, origine non tracée. Crête par canal : 8 Ω 5,0 A, 4 Ω 7,07 A, < OC 17,0 A.
Réglage `OC_ADJ` : `R304` 22 k [K] → 17,0 A en CB3C [D, Table 4] ✓.
LM5069-2 (`R306` 4 mΩ, `R310` 147 k, `C325` 4,7 µF, `R307/308/309` 191k/5,11k/9,09k) [K] :
- Limite de courant : 48,5/55/61,5 mV ÷ 4 mΩ = 12,1/13,75/15,4 A ✓ ; disjoncteur 20,0/26,3/32,5 A ✓.
- UV : 2,5·205,2/14,2 = 36,1 V (coupure) ; reprise + 21 µA·191 k = 40,1 V ; OV : 2,5·205,2/9,09 = **56,4 V**, reprise 52,3 V ; pire cas OVLOTH 2,6 V : 58,7 V (docs 59,8 V avec 1 %) ✓, ≥ 9 V sous 69 V.
- Timer : t_flt = 4,7 µF·VTMRH/I = 147/221/383 ms ✓ (132–422 ms avec ±10 %). Énergie du bulk (15 400 µF = C312–C315 + C316–C317 ✓) : ½CV² = 17,7 J à 48 V, 24,1 J à 56 V ; 17,7 J/299 W = 59 ms ≈ doc 69 ms typ. P_LIM = 299 W : équation 9 non reproduite dans `docs/` → **non recalculée**.
- SOA `Q302` : 389 W exigés (6,95 A/56 V/422 ms) vs 625 W garantis à 75 °C [D] : marge 1,61 ; suppose le boîtier maintenu à 75 °C (radiateur) [NV].
- Dissipations : shunt 4 mΩ : 0,09 W (4,63 A), 0,35 W (9,3 A), 0,95 W (15,4 A) < 1 W à 70 °C. `Q302` (≤ 11 mΩ) : 0,24/0,95/2,6 W. `Q301` (33 mΩ max, 25 °C) : 0,71 W (4,63 A), 0,81 W (4,95 A), 2,85 W (9,3 A, au-delà des 6,9 A convectifs : dépend du radiateur).
- Topologie [K] : `F301` → `PVDD_FUSED` (`D301`, `Q301` drain) → `Q301` source `PVDD_PROT` (`D302` cathode, `U8` VIN, `R306`, diviseur) → `R306` → `PVDD_SENSE` (`U8` SENSE, `Q302` drain) → `Q302` source = `PVDD` = `U8` OUT ✓ conforme à `protection-48v.md`.
- Réserve : `NEEDS_DATA` stabilité de la boucle de puissance avec Qg = 540 nC (déjà ouverte).

### 2.4 Alimentations auxiliaires [K]/[C]
- Buck U1 : `R39`/`R40` → Vout = Vfb·(1 + 4,99/1) = 5,99·Vfb = 14,98 V si Vfb = 2,5 V (valeur absente de `docs/` : `NEEDS_DATA` confirmation). **A1 : `C7` absent.** `C13` = 1 µF/25 V (docs : 100 nF ; décision `plan.md` C2). Diodes : `D3` K sur `BUCK_VIN`, A sur `PVDD` ; `D1` K sur `SW`, A sur `ISEN` ✓. `NEEDS_DATA` : tON(R_ON = 182 k), fréquence, ondulation, ondulation FB minimale exigée (SNVS376F), limite de courant interne, VIN absolu max (PVDD jusqu'à 59,8 V), rendement.
- LDO : `U2` LM2940-12 : P = (15 − 12)·I12 = 3 V·I12 (0,15 / 0,3 / 0,6 W pour 50 / 100 / 200 mA) ; `U3` TLV1117-33 : P = (12 − 3,3)·I3V3 (≈ 1 mA tracé en F1.3 → 9 mW + courant de repos). Courants IVDD/IGVDD du TPA3255 (`+12V` : `U6` 1, 2, 22), Iq OPA1612, Rθja des SOT-223 : `NEEDS_DATA`. Aucune dissipation chiffrable de façon fiable sans ces données ; le buck est limité par `L1` 1,5 A [D, power-block.md].
- Supervision : `VSENSE` = 15·3,3/12,4 = 3,99 V ; avec un seuil `TPS3802K33` de 2,93 V (valeur non présente dans `docs/`) coupure vers `+15V` ≈ 11,0 V : à croiser avec l'entrée minimale du LM2940-12 (`NEEDS_DATA`). Chaîne MR : `R6` 100 kΩ limite le courant de clamp à ≈ 0,45 mA (48 V/100 k) ; `NEEDS_DATA` tension absolue max de MR (déjà ouverte).

### 2.5 Dissipation et thermique [D]/[C]
U6 : 22,4 W (2×100 W/8 Ω) ; 37,4 W (4 Ω). Budget (75−40)/22,4 = 1,5625 °C/W ✓ (REQ-THERM-1) ; (75−40)/37,4 = 0,94 °C/W (4 Ω, non tenu en convection naturelle : REQ-THERM-3, accepté). Tj = 75 + P·0,36 = 83,1 °C (8 Ω) / 88,5 °C (4 Ω) < 125 °C. Selfs : 1,55/3,1 W. Rien de mesuré [NV] : dissipateur, interface (29 mm²), `Q301` et `Q302` (radiateurs), fusible (cuivre), shunt.

### 2.6 A3 — rail `+12V-OA` [K]/[C]
`L6` 10 µH → C81 10 µF + C82/C108/C208 0,1 µF + C109/C209 10 µF ≈ 30,4 µF. f0 = 9,1 kHz, Z0 = 0,57 Ω ; Q = Z0/R_série (DCR de `L6` + ESR) : 1,9 pour 0,3 Ω ; 5,7 pour 0,1 Ω. Résonance dans la bande audio sans amortissement dédié ; la PSRR de l'OPA1612 à 9 kHz n'est pas dans `docs/` → à simuler/mesurer.

## 3. Divergences avec la référence TI
| Sujet | Référence | Schéma | Statut |
|---|---|---|---|
| FB LM5010A | `C7` 5,6 nF (EVM) | absent | A1 |
| `C13` | 100 nF | 1 µF/25 V | accepté (`plan.md`) |
| Filtre de sortie | EVM 10 µH + 1 µF ; Fig. 29 15 µH + 680 nF + 10 nF/3,3 Ω + 1 nF | 15 µH + 680 nF, sans réseau RC | A2 |
| AOP | NE5532 (EVM) | OPA1612 pin-à-pin | choix documenté |
| Volume | potentiomètre sur carte (EVM) | `J4` déporté, sans DC-block de l'étage | B1 |
| Protection 48 V | absente de l'EVM | LM5069-2 + P-MOS | ajout du projet |

## 4. Non mesuré (H1.3)
- **Bruit** : tout §1.6 est un calcul ; bruit TPA3255, couplage de masse, ronflement du câble volume `J4`.
- **EMI/EMC** : émissions 450 kHz et harmoniques, `OUT_x` non amortis (A2), boucles de courant, câble haut-parleur, ESD des RCA.
- **Stabilité** : TPA3255 sur charge réelle/câble, boucle LM5069 (Qg 540 nC), boucle FB du buck sans `C7`, 4 Ω.
- **Thermique** : dissipateur, interface 0,57 °C/W sur 29 mm², `Q301`, `Q302`, shunt, selfs, LDO.
- **Alimentation externe** : ripple, courant continu garanti, `REQ-PSU-1`.

## 5. `NEEDS_DATA` (précis)
1. OPA1612 (SBOS450C) : Voh/Vol sous 4 kΩ et 6,7 kΩ à 12 V simple, en, in, Zout, PSRR à 9 kHz.
2. TPA3255 (SLASEA8A) : table fs = f(R_FREQ_ADJ) pour 22 kΩ ; IVDD, IGVDD ; modulation ; bruit de sortie ; Figure 29 (réseau RC de sortie).
3. LM5010A (SNVS376F) : Vfb, tON(R_ON), ondulation FB minimale, VIN absolu max, rendement ; rôle de `C7`.
4. LM2940-12, TLV1117-33 : dropout, tolérance, charge minimale, Rθja SOT-223.
5. TPS3802K33 : seuil VDD, tension absolue de MR.
6. LM5069 (SNVS452G) : équation 9 (P_LIM vs R_PWR) pour reproduire 299 W.
7. WIMA MKP4 680 nF : courant d'ondulation admissible ; `PA6331-AE` : pertes fer à 450 kHz.
8. Potentiomètre `J4` : référence, loi, résistance ; impédance de sortie des sources.
9. Origine du chiffre 9,3 A de crête (4 Ω).
