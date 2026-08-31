# MCP_BUG — incorrect DocumentType routing for Eeschema

Date : 2026-08-31. Konnect `v1.1.2` (défaut) → correctif porté par la branche
`ai/documenttype-routing-v1.1.3`.

## Cause

Konnect ne transmettait aucun contexte de document au serveur IPC de KiCad. Le
client interrogeait donc toujours `DOCTYPE_PCB` en premier et retombait sur ce
type par défaut. Lancé depuis Eeschema, le serveur MCP se croyait attaché à
l'éditeur de PCB : `save_project` visait la carte, et
`GetOpenDocuments` était appelé avec un type que l'éditeur ouvert ne gère pas,
d'où les `AS_UNHANDLED` observés au jalon B1.1.

Le manifeste de plugin de KiCad 10 accepte `actions[].args` et le transmet à
`argv`, bien que ce champ soit absent du schéma JSON publié : le contexte
pouvait donc être transmis explicitement, sans heuristique.

## Impact sur le benchmark

Toute la voie schématique du benchmark était faussée depuis Eeschema :
attachement au mauvais document, `save_project` inopérant ou dirigé vers la
carte, et résolution de bibliothèque projet échouant sur un document qui n'était
pas celui que l'utilisateur regardait. Le jalon B1.1 restait bloqué, avec les
erreurs conservées telles quelles : `AS_UNHANDLED`, `no_project_path_found`,
`Library 'HifiAmp_TPA3255_Local' not found`.

## Reproduction

1. Installer Konnect `v1.1.2`, ouvrir un `.kicad_sch` dans Eeschema, API IPC
   activée.
2. Appeler `open_project` : le document est annoncé comme PCB.
3. Appeler `save_project` : la carte est écrite, pas le schéma.

Un cas minimal hors projet Hi-Fi suffit ; le défaut ne dépend ni du projet ni
de ses bibliothèques.

## Correctif

Les actions du plugin bornées par périmètre passent explicitement
`--document-type pcb|schematic`. Le mode `Auto`, utilisé quand un client MCP
externe n'a aucun indice de lanceur, n'accepte qu'un seul handler vivant : zéro
ou deux handlers échouent explicitement. Il n'existe plus aucun repli PCB, et
un contexte indéterminé refuse au lieu de deviner.

## Validations

- Gate local complet sur le correctif : 1 422 tests PASS / 38 ignorés,
  5 doctests PASS / 3 ignorés, clippy strict PASS, viewer 20 tests PASS.
- Eeschema réel (KiCad 10.0.6), `scripts/live-schematic-e2e.ps1` : 7 contrôles
  PASS. Contexte `schematic` → document `schematic` ; `save_project` répond
  « Schematic changes are already persisted to disk. » ; contexte `pcb` sans
  carte ouverte → refus explicite ; `Auto` → résout l'unique handler ; après
  arrêt de l'éditeur, `.kicad_sch` et `.kicad_pcb` sont bit-identiques ; le
  routage survit à une réouverture.
- Pcbnew réel, `scripts/live-pcb-e2e.ps1` : 3 tests live PASS, sortie 0. La
  voie PCB n'a pas régressé.

Deux préconditions d'environnement, découvertes en validant et sans rapport
avec ce défaut, sont désormais appliquées par les scripts : aucun doublon
d'identifiant de plugin sous `3rdparty` (trois copies tuent l'éditeur trois
secondes après le démarrage) et aucune autre instance KiCad ne doit détenir le
socket d'API.

## État du projet Hi-Fi

Inchangé. Le smoke-test du 2026-08-31 a ouvert `HifiAmp_TPA3255.kicad_sch` dans
Eeschema en lecture seule et vérifié que `.kicad_sch`, `.kicad_pcb`, `.kicad_pro`
et `.kicad_sym` sont bit-identiques avant et après. L'étape B1.3 n'a pas été
reprise et aucun fichier KiCad n'a été édité directement.

Les statuts du checkpoint B1.1 restent ceux de `benchmark-mcp.md` : l'échec
constaté ce jour-là appartient au benchmark et n'est pas réécrit ici.
