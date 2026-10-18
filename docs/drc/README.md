# DRC final G1.1

Rapport archivé : [`drc-g1.1.json`](drc-g1.1.json).

## Contexte

| Élément | Valeur |
| --- | --- |
| Date du rapport | 2026-10-18 (JSON `2026-10-18T08:55:39`) |
| Branche / HEAD | `ai/g1-drc-review` / `5eaf144` (phase F close) |
| `HifiAmp_TPA3255.kicad_pcb` MD5 | `70e315fad1997ce7b3616006aac5cd1b` |
| `HifiAmp_TPA3255.kicad_dru` MD5 | `cb9016bfa1ef3b3c612531ec1e105f7a` |
| Version | `kicad-cli` 10.0.6 |
| Commande | `kicad-cli pcb drc --schematic-parity --severity-all --format json --output docs/drc/drc-g1.1.json HifiAmp_TPA3255.kicad_pcb` |
| Écriture du PCB | aucune : pas de `--save-board`, pas de `--refill-zones` ; MD5 du PCB identique avant et après |

Un second passage `--format report` (texte) donne les mêmes 116 entrées ; il n'est pas archivé
(sortie dans le scratchpad de session).

## Résultat

| Catégorie | Type | Sévérité | Nombre |
| --- | --- | --- | --- |
| `violations` | `silk_overlap` | warning | 90 |
| `violations` | `silk_over_copper` | warning | 26 |
| `unconnected_items` | — | — | 0 |
| `schematic_parity` | `net_conflict` | warning | 1 |
| `schematic_parity` | `extra_footprint` | warning | 2 |

- Aucun `clearance`, `shorting_items`, `track_dangling`, `via_dangling`, `courtyards_overlap`,
  `hole_clearance`, `solder_mask_bridge`, `starved_thermal`, `isolated_copper`, `copper_sliver`,
  `lib_footprint_issues`, `lib_footprint_mismatch`.
- **Parité : 3 écarts connus** et expliqués.
  - `net_conflict` `U6` pin 45 `/GND` : le PowerPAD est sur le dessus du boîtier, il n'a pas de pastille cuivre.
  - `extra_footprint` `H1` et `H2` : trous de fixation mécaniques volontairement sans symbole.
- Zones : `/GND` `In1.Cu` et `B.Cu`, `/PVDD_FUSED` `F.Cu` et `In2.Cu`, remplies dans le fichier.
  Le rejeu avec `--refill-zones` sur une copie (F1.4-b5) donne des polygones identiques.

## Contrôles ignorés par la configuration du projet

Le JSON liste cinq contrôles ignorés (`ignored_checks`) : `missing_courtyard`,
`track_not_centered_on_via`, `tuning_profile_track_geometries`, `footprint_filters_mismatch`,
`footprint_type_mismatch`. Vérification complémentaire : les 126 empreintes ont bien un courtyard
(`F.CrtYd` ou `B.CrtYd`), donc `missing_courtyard` ne masque rien.

Les sévérités `warning` du projet (dont `track_dangling`, `via_dangling`, `isolated_copper`)
n'ont aucune entrée : le critère « 0 » s'applique aussi à elles.

# DRC final G1.3

Rapport archivé : [`drc-g1.3.json`](drc-g1.3.json).

| Élément | Valeur |
| --- | --- |
| Date du rapport | 2026-10-18 (JSON `2026-10-18T10:05:00`) |
| Branche / HEAD | `ai/g1-drc-review` / `4fb6e89` (G1.3-b commitée) |
| `HifiAmp_TPA3255.kicad_pcb` MD5 | `ea6ea4b3695946c52b2deca67f622fd4` |
| `HifiAmp_TPA3255.kicad_sch` MD5 | `9bcba85d947b78af82475f58438dcdcb` |
| `HifiAmp_TPA3255.kicad_dru` MD5 | `cb9016bfa1ef3b3c612531ec1e105f7a` (inchangé depuis G1.1) |
| Version | `kicad-cli` 10.0.6 |
| Commande | `kicad-cli pcb drc --schematic-parity --severity-all --format json --output docs/drc/drc-g1.3.json HifiAmp_TPA3255.kicad_pcb` |
| Écriture du PCB | aucune : MD5 du PCB identique avant et après |

## Résultat

| Catégorie | Type | Sévérité | G1.1 | G1.3 |
| --- | --- | --- | --- | --- |
| `violations` | `silk_overlap` | warning | 90 | 89 |
| `violations` | `silk_over_copper` | warning | 26 | 25 |
| `unconnected_items` | — | — | 0 | 0 |
| `schematic_parity` | `net_conflict` | warning | 1 | 1 |
| `schematic_parity` | `extra_footprint` | warning | 2 | 2 |

- Aucun autre type : 0 `clearance`, `shorting_items`, `track_dangling`, `via_dangling`,
  `courtyards_overlap`, `hole_clearance`, `solder_mask_bridge`, `starved_thermal`, `isolated_copper`.
- Parité : les 3 mêmes écarts connus (`U6` pin 45, `H1`, `H2`).
- Comparaison à G1.1 (appariement type + objets) : 3 entrées silk nouvelles, 5 disparues, toutes des
  champs référence déplacés avec les composants (C9, C13, C318, C319, C326, C329) ; aucune violation
  non-silk. Le total silk baisse de 116 à 114.
- Même liste de 5 contrôles ignorés qu'en G1.1. ERC du schéma : 0 erreur, 16 avertissements connus
  (6 `lib_symbol_mismatch`, 10 `endpoint_off_grid`).
- Empreintes : 127 (126 + `C329`) ; `(units` : 125 ; `pad_prop_heatsink` : 1.
- Zones : `--refill-zones --save-board` sur une copie redonne un fichier de MD5 identique au PCB
  du dépôt (`ea6ea4b3…`), donc zones à jour. Surfaces (cm²) : `/GND` `B.Cu` 289,99 (G1.1 : 290,15),
  `/GND` `In1.Cu` 290,27 (290,39), `/PVDD_FUSED` `F.Cu` 6,54 et `In2.Cu` 7,16 (inchangées).

# G1.4 — DRC final

Rapport archivé : [`drc-g1.4.json`](drc-g1.4.json).

| Élément | Valeur |
| --- | --- |
| Date du rapport | 2026-10-18 (JSON `2026-10-18T10:17:23`) |
| Branche / HEAD | `ai/g1-drc-review` / G1.4 commitée |
| `HifiAmp_TPA3255.kicad_pcb` MD5 | `4f4b36822efc7ac6eb330008314dde87` |
| `HifiAmp_TPA3255.kicad_sch` MD5 | `9bcba85d947b78af82475f58438dcdcb` |
| `HifiAmp_TPA3255.kicad_dru` MD5 | `cb9016bfa1ef3b3c612531ec1e105f7a` (inchangé) |
| Version | `kicad-cli` 10.0.6 |
| Commande | `kicad-cli pcb drc --schematic-parity --severity-all --format json --output docs/drc/drc-g1.4.json HifiAmp_TPA3255.kicad_pcb` (sans `--save-board`, MD5 PCB identique avant et après) |

## Résultat

| Catégorie | Type | Sévérité | G1.1 | G1.3 | G1.4 |
| --- | --- | --- | --- | --- | --- |
| `violations` | `silk_overlap` | warning | 90 | 89 | 0 |
| `violations` | `silk_over_copper` | warning | 26 | 25 | 4 |
| `unconnected_items` | — | — | 0 | 0 | 0 |
| `schematic_parity` | `net_conflict` | warning | 1 | 1 | 1 |
| `schematic_parity` | `extra_footprint` | warning | 2 | 2 | 2 |

- Les 4 `silk_over_copper` restantes sont les repères de broche 1 de `L301`–`L304` (empreinte, acceptés).
  Aucune autre violation : 0 `clearance`, `solder_mask_bridge`, `track_dangling`, `via_dangling`,
  `courtyards_overlap`, `hole_clearance`, `starved_thermal`, `isolated_copper`.
- Parité : les 3 mêmes écarts connus (`U6` pin 45, `H1`, `H2`).
- Comparaison à G1.3 : 114 → 4 entrées silk (110 champs référence déplacés, 22 champs) ; aucune violation
  non-silk ni modification électrique ; vias `C206.2`, `U7.3`, `C10.2` sortis de leur pastille ; marge de masque
  `U8` à 0,025 mm (pont 0,10 mm). 125 `(units`, `pad_prop_heatsink` = 1.
- ERC du schéma (`--severity-all`) : 0 erreur, 16 avertissements identiques aux 16 connus
  (6 `lib_symbol_mismatch`, 10 `endpoint_off_grid`).
- Zones : `--refill-zones --save-board` sur une copie redonne un fichier de MD5 identique au PCB du dépôt
  (`4f4b3682…`), donc zones à jour.
