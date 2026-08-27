# Benchmark MCP KiCad — checkpoint bloqué B1.1

Date : 2026-08-27.

## Résultat du jalon

KiCad 10.0.3 est installé et le réglage persistant contient `"enable_server": true`. Le projet `HifiAmp_TPA3255` et ses fichiers schéma/PCB existent, mais le schéma et le PCB restent vides. Aucun ERC ni DRC n’a été exécuté.

Le MCP a créé et vérifié `HifiAmp_TPA3255.kicad_sym` avec le symbole `LM5010ASD` à 11 connexions, conforme au boîtier TI `DPR0010A`. La bibliothèque a ensuite été enregistrée dans KiCad comme `HifiAmp_TPA3255_Local` par GUI, car aucune capacité MCP utilisable ne gérait son enregistrement/rechargement.

Le placement de `U1` reste bloqué. Selon l’instance IPC ciblée, le MCP renvoie soit `no_project_path_found`, soit `Library 'HifiAmp_TPA3255_Local' not found`, malgré le projet et la bibliothèque visibles dans KiCad. Le wrapper de sauvegarde a aussi échoué sur `GetOpenDocuments (AS_UNHANDLED)`. Trois cycles ouverture/rechargement/résolution/placement ont reproduit le défaut sans progrès.

## Capacités observées

- Fonctionnelles : catalogue/capabilities, `search_symbols`, création et vérification d’un symbole custom, lecture explicite du projet/schéma vide, lancement/ouverture UI, viewer, `server_stats`, `get_recent_calls`.
- Instables ou bloquantes : attachement au bon document IPC, `GetOpenDocuments`, résolution d’une bibliothèque de symboles spécifique au projet par le placeur, persistance schématique E2E.
- Opération GUI nécessaire : enregistrement de `HifiAmp_TPA3255.kicad_sym` comme `HifiAmp_TPA3255_Local`; aucun design n’a été modifié par GUI.

## Observabilité

- Instantané initial : `server_stats` a observé 17 appels et 1 erreur.
- Un audit read-only intermédiaire a rapporté 13 appels réussis sur 13 ; ce sous-jalon n’est plus exposé par le serveur courant.
- Instantané final après redémarrages : 0 appel, 0 erreur ; les statistiques avaient été réinitialisées après l’activité.
- Total E2E, nombre exact de retries et ventilation succès/échecs : non exposés de façon persistante, donc non annoncés.

Erreurs significatives conservées exactement : `KiCAD IPC is not reachable`, `AS_UNHANDLED`, `no_project_path_found`, `Library 'HifiAmp_TPA3255_Local' not found`.

## Niveaux d’acceptation au checkpoint

- Niveau 1 — Schéma : FAIL. Projet présent, schéma vide, aucun ERC.
- Niveau 2 — PCB : FAIL. PCB vide, aucun placement/routage/DRC.
- Niveau 3 — Ingénierie : PARTIAL. Architecture et composants auxiliaires sourcés, sans capture ni validation physique.
- Niveau 4 — Autonomie MCP : FAIL. Création de symbole réussie, placement/persistance E2E bloqués et une opération GUI nécessaire.

## Statuts demandés

**SCHÉMA:** FAIL  
**PCB:** FAIL  
**ERC:** NON EXÉCUTÉ  
**DRC:** NON EXÉCUTÉ  
**CONFORMITÉ DATASHEETS:** PARTIAL  
**CRÉDIBILITÉ ÉLECTRONIQUE:** PARTIAL  
**AUTONOMIE MCP:** FAIL  
**PRÊT À FABRIQUER:** NON

Ces statuts ne valident ni THD+N, SNR, EMI/EMC, thermique, stabilité, puissance continue ni performances acoustiques.
