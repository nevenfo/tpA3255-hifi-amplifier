# MCP_BUG — `set_design_rules` et `set_active_layer` écrivent des jetons invalides dans `(setup ...)`

Date : 2026-09-01. Konnect `v1.1.3`. Rencontré en E1.1, sur un `.kicad_pcb` neuf
et vide.

## Symptôme

Après appel de `set_design_rules` puis de `set_active_layer`, le `.kicad_pcb`
n'est plus lisible par KiCad. `kicad-cli pcb drc` refuse le fichier :

```
Échec du chargement du circuit imprimé: Inattendu active_layer en
'HifiAmp_TPA3255.kicad_pcb', ligne 32, décalage 6.
```

## Cause

Les deux outils écrivent dans le bloc `(setup ...)` du `.kicad_pcb` des jetons
qui n'appartiennent pas au format. Bloc produit, tel quel :

```
	(setup
    (active_layer "F.Cu")
		(pad_to_mask_clearance 0.05)

  (min_clearance 0.2)
  (min_track_width 0.2)
  (min_via_drill 0.3)
  (min_via_size 0.6))
```

Deux défauts distincts s'additionnent.

1. **`set_design_rules` se trompe de fichier.** Les règles globales de conception
   ne vivent pas dans le `.kicad_pcb` mais dans le `.kicad_pro`, sous
   `board.design_settings.rules`, avec des clés JSON dont les noms diffèrent :
   `min_via_diameter` et `min_through_hole_diameter`, non `min_via_size` et
   `min_via_drill`. L'outil écrit **aux deux endroits** : le `.kicad_pro` reçoit
   correctement `min_track_width` et `min_through_hole_diameter`, mais laisse
   `min_clearance` et `min_via_diameter` à leurs valeurs par défaut, si bien que
   **deux des quatre règles demandées ne sont pas appliquées** — et le
   `.kicad_pcb` reçoit en plus les quatre jetons invalides.
2. **`set_active_layer` aggrave au lieu de réparer.** Appelé pour tenter de faire
   réécrire le bloc, il y **ajoute** `active_layer`, tout aussi invalide, sans
   nettoyer les précédents. La couche active est un état d'interface, elle n'a
   rien à faire dans un fichier de carte.

L'indentation mêlant tabulations et espaces trahit une écriture par
concaténation de texte plutôt que par sérialisation de l'arbre S-expression.

## Absence de recours

Les 203 outils des 17 toolsets ont été passés en revue : **aucun outil
`repair`, `undo` ou `restore_snapshot`** ne permet de nettoyer le bloc `setup`.
Une fois les jetons écrits, le MCP ne sait plus les retirer.

## Contournement retenu

1. `git checkout -- HifiAmp_TPA3255.kicad_pcb` pour revenir au dernier état
   commité. Sûr ici parce que le fichier était **vide de toute connectivité** :
   aucune empreinte, aucune piste, un seul net vide. Le seul contenu perdu était
   la déclaration des deux couches internes, reposée ensuite.
2. **Ne plus jamais appeler `set_design_rules` ni `set_active_layer`** sur ce
   projet.
3. Régler les règles globales directement dans le `.kicad_pro`, qui est un
   fichier de configuration JSON sans connectivité à corrompre — même catégorie
   que `sym-lib-table` et `fp-lib-table`.

## Leçon transposable

Le MCP a produit ici un fichier que KiCad refuse, sans qu'aucun retour d'outil
ne signale d'échec. C'est la deuxième fois sur ce projet qu'un rapport d'outil
optimiste masque un fichier cassé. **La règle du projet — vérifier au fichier, et
non sur le retour de l'outil — est ce qui a permis de le voir.** Ici la preuve
décisive n'était même pas la relecture du texte, qui semblait plausible, mais
`kicad-cli pcb drc` : seul le parseur de KiCad tranche ce que KiCad accepte.
