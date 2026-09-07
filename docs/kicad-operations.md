# Piloter KiCad sur ce projet

Ce document porte le savoir d'outillage — durable, appris à la dure, indépendant de
l'avancement. `progress.md` n'en garde rien : il n'y renvoie.

## La séquence qui fonctionne

Le MCP ne sait pas tout faire, et l'IPC est capricieux. Cet ordre est le seul qui tienne.

1. **Aucun des 203 outils du MCP ne sait synchroniser schéma → PCB.** `kicad-cli` non plus.
   Seule voie : **Outils → « Mise à jour du PCB à partir du Schéma »** dans l'éditeur de PCB,
   en action GUI.
2. **`move_component` / `rotate_component` ne fonctionnent qu'en IPC**, pas en mode fichier.
   `place_component` est en mode fichier mais **n'a aucun champ net**.
3. **L'éditeur de PCB doit être ouvert depuis le gestionnaire de projet.** Un `pcbnew.exe`
   lancé isolément crée un processus séparé **qui ne partage pas le canal IPC**. Lancé en
   autonome, il refuse en plus la synchronisation au schéma.
4. **Un seul document ouvert à la fois.** Schéma et PCB ensemble donnent
   `KiCad document context is ambiguous: expected exactly one PCB or schematic handler, found 2`.
   Fermer l'autre fenêtre par `WM_CLOSE` sur son handle Win32 suffit — la fenêtre du
   gestionnaire ne compte pas comme handler.
5. `kicad_common.json` porte déjà `api.enable_server = true` ; il ne manque que le processus.
6. **Toujours prouver un enregistrement par le MD5 du fichier**, jamais par le retour de
   `save_project`.
7. **Ne jamais lancer KiCad depuis le shell d'un agent : il est élevé, KiCad hérite du jeton
   administrateur, et UIPI bloque alors silencieusement tout clic et toute frappe** venant
   d'une session d'automatisation non élevée — les appels rapportent un faux succès.
   **Lancer par `explorer.exe <chemin du projet>`**, qui s'exécute avec le jeton utilisateur
   normal. Contrôler le `TokenElevation` du processus : il doit valoir `normal`.
8. Le titre de fenêtre porte un `*` tant que des modifications live ne sont pas enregistrées :
   indicateur fiable.

## Preuve indépendante du MCP

`kicad-cli.exe` — `sch erc`, `pcb drc --format json` — depuis
`C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/`. C'est la seule mesure qui ne dépend
pas de la couche qui vient d'écrire. **Les rapports d'agents sont systématiquement vérifiés par
le principal avant tout verdict.**

## État de la stack MCP

`kicad-agentic-mcp` v1.1.3. **Ne pas restaurer `v1.1.2`.** Analyses dans
`reports/MCP_BUG-documenttype-routing-eeschema.md` et `reports/MCP_BUG-setup-tokens-kicad-pcb.md`.

- `save_project` / `open_project` échouent hors GUI : `Connection refused`.
- `on_board` / `in_bom` / `dnp` inaccessibles ; `edit_schematic_component` ne gère que
  Reference/Value/Footprint/Datasheet plus des propriétés personnalisées, et accepte `uuid`
  en plus de `reference`.
- `add_power_symbol` : `power_net` désigne le nom du symbole de librairie, pas le net cible.
- Outils de `load_toolset` accessibles seulement via `kicad_invoke`. Sortie tronquée au-delà
  d'environ 72 000 caractères.
- **`set_design_rules` et `set_active_layer` sont PROSCRITS** : jetons invalides dans
  `(setup ...)`, fichier illisible par KiCad, aucun retour d'erreur.

## Où vivent les règles

- Les **règles DRC personnalisées** vivent dans `<projet>.kicad_dru`, jamais dans
  `board.design_settings.rules` du `.kicad_pro`, qui ne porte que des minima numériques.
- **`min_clearance` ne serre pas une règle personnalisée.** Hypothèse posée puis réfutée :
  une règle à 0,15 mm passe alors que `min_clearance` vaut 0,2. À ne pas re-supposer.
- **Une règle DRC se prouve sélective, jamais supposée telle.** Sans violation ailleurs pour
  le révéler, une condition trop large relâcherait la carte entière en silence. Le contrôle
  est de retirer une référence de la condition et de vérifier que ses seules violations
  reviennent.

## Relire les fichiers hors MCP

Dans le format KiCad 10, un bloc d'empreinte porte `(layer ...)` et `(uuid ...)` **avant**
`(at x y rot)` — une regex qui attend `(at` juste après `(footprint` échoue. Les pastilles
portent `(net "NOM")` **sans identifiant numérique**, et il n'y a pas de table de nets en fin
de fichier. Dans le `.kicad_sch`, `lib_symbols` précède les instances : itérer sur les blocs
de premier niveau à parenthèses équilibrées.

## Sources et mesures

**Lire les cotes par extraction du PDF fabricant**, jamais par listing distributeur ; contrôler
l'échelle **et les conditions de mesure**. **PDF sans couche texte** : rendre en PNG par
`pymupdf`. **Écrire l'extraction dans un fichier UTF-8 avant affichage** — la console Windows
casse sur les accents.
