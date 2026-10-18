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
