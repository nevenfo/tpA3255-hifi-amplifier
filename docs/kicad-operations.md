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

## Quand l'automatisation GUI est morte, et comment le savoir vite

**Symptôme.** `SetForegroundWindow`, `SetCursorPos` et tout `ui_click` renvoient `False` avec
`GetLastWin32Error = 0`, ou `SendInput was blocked`. Les appels rapportent parfois un faux
succès ; un essai a même minimisé une fenêtre au lieu de l'ouvrir.

**Le test qui tranche en un appel** : tenter d'activer **une fenêtre quelconque sans rapport
avec KiCad** — un navigateur déjà ouvert fait l'affaire. Si elle non plus ne s'active pas, le
blocage est **au niveau de la session entière**, et il est inutile de chercher du côté de KiCad,
de son élévation ou du MCP. Ce contrôle évite de rejouer la piste `TokenElevation`, qui est une
autre cause du même symptôme et se diagnostique différemment.

**Ce qui ne contourne pas le blocage** : les coordonnées physiques, la navigation clavier
(`Tab`), l'activation de fenêtre, et le pattern UIA `Invoke` — tous repassent par `SendInput`.
Le seul canal qui répond est le pattern sémantique `SelectionItem` sur un `TreeItem`, qui passe
par COM sans `SendInput` — mais il **sélectionne** l'entrée sans l'ouvrir, et le panneau de
lancement du gestionnaire de projet est un `Pane` **sans pattern UIA** exploitable.

**Conséquence pratique.** L'IPC exige que l'éditeur de PCB soit **déjà ouvert**, et rien ne
permet de l'ouvrir sans entrée synthétique. Quand la session est dans cet état, la seule issue
est que **l'utilisateur ouvre l'éditeur de PCB lui-même** depuis le gestionnaire de projet ;
l'automatisation reprend ensuite normalement par le canal IPC, qui n'est pas affecté. Contexte
observé : session **RDP** (`RDP-Tcp#0`) — une session distante déconnectée ou verrouillée
suffit à produire ce blocage.

**Issue vérifiée.** Le contournement a fonctionné tel quel : l'éditeur de PCB ouvert par
l'utilisateur, un lot de cinq déplacements est passé par IPC en un seul appel — `atomic=true`,
`verify=auto` — suivi d'un `save_project`, **sans aucun refus**. Deux enseignements. Le blocage
`SendInput` **n'atteint pas l'IPC** : il ne coûte que l'ouverture de la fenêtre, jamais le
pilotage. Et il **ne survit pas à la session** : une session neuve avec l'éditeur déjà ouvert
n'en garde aucune trace. Avant de rouvrir un blocage GUI hérité d'une session précédente, donc,
vérifier d'abord si la fenêtre nécessaire n'est pas déjà là — `Get-Process` sur `kicad` et son
`MainWindowTitle` suffisent à le dire.

## Mettre à jour le PCB depuis le schéma

Opération **GUI par nature** : ni `kicad-cli` ni le MCP ne l'exposent. Elle s'automatise malgré
tout, dès lors que l'entrée synthétique fonctionne.

1. **Ouvrir le gestionnaire de projet par `explorer.exe "<projet>.kicad_pro"`**, jamais depuis le
   shell d'un agent — c'est ce qui garantit le jeton non élevé (`TokenElevation` = 0) sans lequel
   UIPI bloque silencieusement clics et frappes. Contrôle en un appel :
   `GetTokenInformation(token, TokenElevation=20)`.
2. Ouvrir l'**éditeur de PCB depuis le gestionnaire**. Un `pcbnew.exe` isolé ne partage pas l'IPC
   et refuse la synchronisation.
3. **Outils → « Mise à jour du PCB à partir du Schéma »**, raccourci **F8**.
4. **Piège à ne jamais oublier : « Supprimer les empreintes sans symbole associé » est COCHÉE par
   défaut.** Sur ce projet, elle **détruirait `H1` et `H2`**, les deux trous de fixation
   mécaniques volontairement sans symbole au schéma — précisément les deux `extra_footprint` de
   la baseline de parité. **La décocher avant chaque lancement**, et relire l'état de toutes les
   cases plutôt que de le supposer.
5. Le rapport doit ne contenir **aucune ligne de suppression ni d'ajout d'empreinte**. L'erreur
   `U6 pad 45 non trouvé` est **attendue** : le PowerPAD du TPA3255 est sur le dessus du boîtier.
6. **Ctrl+S**, puis vérifier que le `*` a disparu du titre — et confirmer par le MD5 du fichier.

**Le blocage `SendInput` de la session RDP n'est pas une propriété du projet.** Il vient d'une
session distante **déconnectée ou verrouillée** : reconnectée, tout repasse. Avant de rouvrir un
blocage GUI hérité, refaire le test en un appel décrit plus haut — activer une fenêtre sans
rapport — plutôt que de croire la note d'une session précédente.

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

### Deux pièges qui rendent une mesure fausse en silence

**La rotation d'un pad se fait en `-θ`, pas en `+θ`.** Dans le repère du fichier, `y` pointe vers
le bas ; l'offset local d'une pastille se transforme donc en
`x' = x·cos θ + y·sin θ` et `y' = −x·sin θ + y·cos θ`. Prendre `+θ` **échange les deux pastilles**
d'un composant tourné à ±90°. Ce qui rend l'erreur redoutable, c'est qu'elle est **invisible à 0°
et à 180°**, où les deux conventions coïncident : un script faux peut valider des dizaines de
composants — tous les borniers, `U6` à 180° — avant de mentir sur le premier 0603 vertical.
**Contrôle** : sur un composant à ±90°, vérifier que le pad calculé porte le net attendu ; à
défaut, router vers ce point et lire `unconnected_items` et `track_dangling` au DRC.

**`(segment` n'est pas suivi d'un espace mais d'un saut de ligne.** `(footprint "Nom"` en met un,
`(segment` non — chercher la chaîne `"(segment "` rend **zéro résultat sans erreur**, donc « aucune
piste » sur une carte routée. Faire correspondre `(<tag>` suivi de **n'importe quel blanc**.

Ces deux-là partagent la forme du `schematic_parity` à zéro : **un compteur nul ne prouve rien
tant qu'on n'a pas vérifié que la mesure a eu lieu.** Tout script de contrôle se calibre d'abord
sur un état qu'il doit rejeter.

## Sources et mesures

**Lire les cotes par extraction du PDF fabricant**, jamais par listing distributeur ; contrôler
l'échelle **et les conditions de mesure**. **PDF sans couche texte** : rendre en PNG par
`pymupdf`. **Écrire l'extraction dans un fichier UTF-8 avant affichage** — la console Windows
casse sur les accents.

## `schematic_parity` : une preuve qui n'en était pas une

**Le champ `schematic_parity` du rapport JSON vaut `0` quand le test n'a pas tourné**, et rien ne
distingue ce zéro d'un zéro mérité. Le test de parité PCB ↔ schéma est une **option désactivée
par défaut** :

```
kicad-cli pcb drc --schematic-parity --format json --output <rapport> <carte>
```

Sans `--schematic-parity`, la commande ne charge même pas le schéma. Toutes les mesures de ce
projet antérieures à E1.11 ont omis l'option : le `schematic_parity = 0` cité comme preuve dans
`plan.md` et `progress.md` signifiait **« test non exécuté »**, pas « aucun écart ». La preuve
était vide.

**La vraie baseline est 3 écarts, tous explicables** — mesurée sur le schéma d'origine, carte
inchangée :

| Type | Objet | Explication |
| --- | --- | --- |
| `net_conflict` | `U6`, pin 45 `/GND` sans pastille | le PowerPAD du TPA3255 est **sur le dessus** du boîtier (établi en E1.7) : il n'a pas de pastille cuivre côté PCB, et le symbole déclare pourtant la broche |
| `extra_footprint` | `H1` | trou de fixation mécanique, sans symbole au schéma |
| `extra_footprint` | `H2` | idem |

Aucun des trois n'est un défaut. Mais ils étaient invisibles, et un vrai écart le serait resté :
c'est précisément ce qui rend l'omission grave. **Toute validation qui cite `schematic_parity`
doit désormais passer `--schematic-parity` et se comparer à 3, pas à 0.**

> **La leçon générale** : un compteur à zéro dans un rapport ne prouve rien tant qu'on n'a pas
> vérifié que la mesure a eu lieu. Le contrôle qui l'a révélé était un test négatif — modifier le
> schéma sans propager, et constater que le rapport continuait d'afficher `0`. Un contrôle qui ne
> sait pas échouer ne vaut rien, et le seul moyen de le savoir est de le faire échouer exprès.

## Propager le schéma vers le PCB : aucune voie automatisable

Il n'existe **ni commande `kicad-cli`, ni outil MCP** qui réalise « Mettre à jour le PCB depuis le
schéma ». `kicad-cli pcb import` n'importe que des formats non-KiCad ; `kicad-cli sch export
netlist` produit bien une netlist, mais rien ne l'importe dans le `.kicad_pcb`. Côté MCP, aucune
capacité de forward-annotation n'est exposée.

**Conséquence pratique** : toute modification de netlist faite au schéma — permutation de
broches, ajout de composant, changement d'affectation — laisse la carte en retard, et seule une
action de l'utilisateur dans l'éditeur de PCB (`Outils → Mettre à jour le PCB depuis le schéma`)
la rattrape. C'est la seconde opération de ce projet à devoir passer par l'utilisateur, après
l'ouverture de l'éditeur.

**Ce qu'il faut vérifier après cette mise à jour**, parce que c'est l'opération qui a déjà causé
les deux régressions connues du projet : `pad_prop_heatsink` de `U1` (perdu en E1.10) et les
`lib_footprint_mismatch` (introduits en E1.3). Relever la baseline avant, la comparer après.

## Règles de routage établies en Phase F

- **Les classes de net se lisent dans `.kicad_pro`**, pas dans `.kicad_dru` qui ne porte que
  l'exception interne à `U6`/`U1`/`U8`. Valeurs : `PWR_AUX` = 0,25 mm d'isolation et 0,8 mm de
  piste nominale, `ANALOG` = 0,25/0,25, `PWR_48V` = 0,5, `GATE_DRIVE` et `GND` = 0,25,
  `PWR_OUT` = 0,5. **Vérifier la classe avant de calculer un couloir** : reprendre celle d'un net
  voisin a déjà coûté un tracé refait.
- **La largeur nominale de classe n'est jamais tenable en sortie de `U6`.** Pastilles
  1,575 × 0,4 mm au pas de 0,635 mm : 0,35 mm passe pour les bootstraps, 0,30 mm pour les
  auxiliaires. Recalculer la marge à chaque sortie de boîtier fin.
- **Un contrôle de dégagement ne vaut que par l'exhaustivité de sa liste d'obstacles.** L'oubli de
  `C106`/`C107`/`C206`/`C207` a coûté un lot de 14 segments et 6 `shorting_items`. Le DRC, lui,
  l'a vu immédiatement : **le juge est le DRC, jamais le script maison seul.**
- **Un compteur nul ne prouve rien tant qu'on n'a pas vérifié que la mesure a eu lieu.** Vaut pour
  `schematic_parity` (exige `--schematic-parity`) comme pour toute extraction du `.kicad_pcb`.
  Calibrer d'abord le contrôle sur un état qu'il doit rejeter.
- **Le MCP exige que l'éditeur de PCB soit ouvert**, pas seulement le gestionnaire de projet :
  sinon `route_trace` échoue en `ipc_rejected` et un lot atomique n'écrit rien. Le geste
  d'ouverture est un geste GUI, à déléguer.
- **KiCad normalise les angles à la réécriture** (`-90` devient `270` dans les champs texte) : un
  diff Git peut montrer des suppressions sans qu'aucun composant ait bougé. Vérifier le `(at)` de
  l'empreinte, pas celui de ses propriétés.
- **Un DRC de comparaison se lance dans le répertoire du projet.** Binaire :
  `C:/Users/FlowUP/AppData/Local/Programs/KiCad/10.0/bin/kicad-cli.exe`.
