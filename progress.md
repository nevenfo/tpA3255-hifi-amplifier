# PROGRESS

## Phase actuelle

**Phase F — Routage. F1.1 et F1.2 closes (sorties Class-D complètes), F1.3 à ouvrir.** 81
segments, **8 vias**. DRC courant : **107 violations toutes de sérigraphie** (86 `silk_overlap`,
21 `silk_over_copper`), **216 non-connectés**, aucun sur `OUT`, `OUT_x_F` ni `BST_A`–`D`,
`schematic_parity` = **3**, aucune `clearance`, aucun `shorting_items`, aucun `track_dangling`,
aucun `courtyards_overlap`. **124 empreintes, 119 blocs `(units`, `pad_prop_heatsink` = 1.**

## Tâche actuelle

**F1.3 — router analogique, contrôle puis signaux non critiques.** À découper en sous-unités.

## Dernière tâche validée

**F1.2-c2 = PASS — placement O1 appliqué, étage de sortie rerouté.** Tores `L302` (260,45 ; 184,5),
`L304` (228,35 ; 184,5), `L301` (276,5 ; 204,3), `L303` (244,4 ; 204,3) ; bootstraps en miroir,
`C306` à 180°. Détail dans `docs/architecture.md`, § F1.2-c.

Validation :

- **Vérification du principal** : DRC `kicad-cli --schematic-parity` propre, non-connectés 216,
  parité 3 ; **seules les 8 empreintes prévues changent**, toutes avec leur `(units` ; tout sur
  `F.Cu`, aucune via sur un `OUT`.
- Longueurs A 29,13 / B 28,92 (écart 0,21), C 68,27 / D 71,89 (écart 3,62) ; largeur < 3,00 mm
  seulement en `x` ≥ 277,2 (pas fin, vias `BST`) ; `OUT_x_F` intégralement à 3,00 mm.
- **Sélectivité DRU rejouée** : copie sans `U6` dans la condition → 11 `clearance`, toutes
  piste ↔ pastille `U6`.

## Décisions actives

Règles durables : `docs/kicad-operations.md` ; placement et mesures : `docs/architecture.md`.

- **Les `OUT` à 5 A restent intégralement sur `F.Cu`, sans via.** Carte non simple-face
  (`C310`/`C311` au dos).
- **Bande `x` 100..155 réservée à l'analogique bas niveau** ; zone d'interdiction de la barre
  `x` [280 ; 291], `y` [160 ; 190] pour les composants (décisions E1).
- **Sérigraphie à traiter en bloc** ; **masses et rails auxiliaires longs au plan de F1.4**.

## Blocage actif

Aucun.

## Fichiers / zones utiles

- `HifiAmp_TPA3255.kicad_pcb` — MD5 `d00e2fb846c4744d688642f534342e53`
- **`docs/kicad-operations.md`** — **à lire avant toute manipulation**
- `docs/architecture.md` — décisions E1, E1.5, F1.2-b, F1.2-c

## NEXT ACTION

**F1.3 — inventorier les 216 chevelus par catégorie (analogique, contrôle, alimentation
auxiliaire, non critique) et découper F1.3 en sous-unités à IDs stables dans `plan.md`**, avec
ordre, contraintes et validation de chacune ; validation : inventaire dont la somme fait 216,
sous-unités écrites dans le plan.
