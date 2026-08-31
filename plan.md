# PLAN — Stress-test E2E KiCad MCP — Amplificateur Hi-Fi

## Objectif final

Transformer le cahier des charges en projet KiCad stéréo OPA1612/TPA3255 documenté et aussi fabricable que les preuves disponibles le permettent, principalement via `kicad-control` et son MCP, sans édition directe de `.kicad_sch` ou `.kicad_pcb`.

## Invariants

- Répondre et documenter en français ; préserver exactement noms, erreurs, commandes et chemins.
- Alimentation DC externe uniquement ; aucun secteur 230 V sur le PCB.
- Toute décision critique vient d’une source fabricant actuelle, d’un calcul explicite ou porte `NEEDS_DATA`.
- Toute opération directe sur KiCad appartient exclusivement à `kicad-control`/MCP ; aucun contournement par édition de `.kicad_sch`/`.kicad_pcb`.
- Gate ERC validé explicitement avant tout début de PCB.
- Ne jamais assimiler ERC/DRC à une validation physique de THD+N, SNR, EMI/EMC, thermique ou stabilité réelle.
- Préserver les fichiers utilisateur ; le principal seul possède Git et les checkpoints.

## Critères globaux de réussite

- Statuts séparés : SCHÉMA, PCB, ERC, DRC, CONFORMITÉ DATASHEETS, CRÉDIBILITÉ ÉLECTRONIQUE, AUTONOMIE MCP, PRÊT À FABRIQUER.
- Livrables : projet, schéma, PCB 4 couches, footprints, BOM, ERC, DRC, design review, calculs, hypothèses, limitations, `NEEDS_DATA`, benchmark MCP.
- Métriques MCP uniquement si observées dans l’état live.

# Phase A — Inspection et architecture

## A0 — Établir l’état initial et la continuité

### Objectif

Prouver l’état du workspace, de Git, de KiCad, de l’IPC/API, du MCP et de ses outils avant toute création KiCad.

### Dépendances

Aucune.

### Tâches

- [x] A0.1 Inspecter le workspace et Git sans modification.
- [x] A0.2 Créer `plan.md`, `progress.md` et un rollback Git local.
- [x] A0.3 Sonder KiCad, IPC/API, MCP et outils exposés par des opérations read-only live.
- [x] A0.4 Consigner capacités, observabilité et limitations initiales.

### Validation

Preuves live compactes ; aucune supposition fondée sur une ancienne documentation.

## A1 — Figer l’architecture électronique initiale

### Objectif

Définir architecture, budgets, gains, interfaces, rails, protections, filtres et composants critiques à partir des sources TI et de calculs traçables.

### Dépendances

A0.3.

### Tâches

- [x] A1.1 Vérifier datasheets, EVM et recommandations TI actuelles avec pages/sections.
- [x] A1.2 Établir le schéma-bloc stéréo et le rôle exact de l’OPA1612/volume/interface différentielle.
- [x] A1.3 Calculer niveaux, gains, impédances, headroom, rails et budget de puissance.
- [x] A1.4 Définir alimentation auxiliaire, séquencement, RESET/MUTE/FAULT et protections.
- [x] A1.5 Justifier filtre LC, découplages, thermique, EMI et connectique.
- [x] A1.6 Enregistrer chaque inconnue critique comme `NEEDS_DATA` et valider la revue d’architecture.

### Validation

Chaque connexion/valeur critique est sourcée ou calculée ; architecture cohérente pour 4–8 Ω et ~2 × 100 W/8 Ω.

# Phase B — Schéma KiCad

## B1 — Créer et construire le schéma par blocs

### Dépendances

A1 validée.

### Tâches

- [x] B1.1 Créer le projet via MCP et vérifier sa réouverture.
- [x] B1.2 Ajouter alimentation DC, auxiliaires et connectique.
- [x] B1.3 Ajouter entrée/volume/analogique gauche puis inspecter les nets.
- [x] B1.4 Ajouter entrée/volume/analogique droite puis inspecter les nets.
- [x] B1.5 Ajouter TPA3255, interface différentielle et contrôles puis inspecter les nets.
- [x] B1.6 Ajouter découplages, puissance, filtres LC, sorties et protections.
- [x] B1.7 Revoir références, alimentations, nets critiques et connexions inattendues.

### Validation

Schéma complet inspecté bloc par bloc via la stack et conforme à l’architecture A1.

## B2 — Retouches schématiques issues des décisions mécaniques et de protection

### Objectif

Intégrer au schéma les décisions prises après le gate C1 : déport mécanique des entrées et du volume, et renforcement de la protection d'entrée 48 V.

### Dépendances

C1 PASS ; décisions utilisateur du 2026-08-31 ; sourcing fabricant de la protection 48 V.

### Tâches

- [ ] B2.1 Déporter `RV1` hors carte : ajouter un connecteur de volume, recâbler les nets et sortir `RV1` du périmètre PCB. *(Connecteur `J4` créé et nets vérifiés ; reste l'exclusion PCB, voir B2.8.)*
- [x] B2.2 Convertir les entrées `J2`/`J3` en connecteur de câblage vers RCA de châssis, avec retour de masse maîtrisé.
- [x] B2.3 Corriger `Q301` : clamp Zener de grille et résistance de grille dimensionnés pour un rail 48 V. L'anti-inversion reste une fonction distincte du hot-swap, la diode de structure d'un MOSFET N côté haut conduisant en inversion.
- [x] B2.4 Dimensionner l'étage `LM5069` sur équations de datasheet : seuils de sous-tension et de surtension avec marge sous 65 V, résistance de shunt, limitation de puissance et temporisateur de défaut.
- [x] B2.5 Vérifier la SOA du MOSFET de hot-swap pendant la charge des 15 400 µF, contre la courbe SOA de la référence retenue. *(`Q302` = `IXTK200N10L2`, SOA garantie 625 W à 75 °C contre 389 W exigés, marge 1,61 ×.)*
- [x] B2.6 Capturer le bloc `LM5069` au schéma : contrôleur, MOSFET série, shunt et réseau de programmation.
- [ ] B2.8 Exclure `RV1` du circuit imprimé (attribut `on_board` à `no`) et supprimer la propriété parasite `exclude_from_board` laissée sur ce symbole. **Aucun des 202 outils MCP n'expose ces opérations** : action manuelle dans l'interface KiCad, ou traitement au moment de la génération du PCB.
- [x] B2.7 Figer `F301` et `D301` sur des références réelles, la TVS étant explicitement cantonnée aux transitoires rapides et non à la protection en surtension. *(`F301` = `Schurter UMT-H` 12,5 A `3403.0285.11` ; `D301` = `SMDJ58CA`, bidirectionnelle pour ne pas annuler l'anti-inversion.)*

- [x] B2.9 Figer `Q301` sur une référence réelle : P-MOS bloquant l'inversion à 56 V plus marge, `I_D` ≥ 12 A continus, `R_DS(on)` faible sous `V_GS` = −15 V, boîtier dissipatif. *(`IPP330P10NM`, Infineon OptiMOS TO-220-3, −100 V / 33 mΩ.)*
- [x] B2.10 Corriger le brochage de `Q301` et `Q302` : les symboles `*_GSD` déclaraient broche 2 = Source alors que les deux composants retenus ont broche 2 = Drain. Défaut invisible à l'ERC, fatal au report PCB. *(Passés en `Q_PMOS_GDS` et `Q_NMOS_GDS` ; correctif géométriquement neutre, positions de broches identiques entre variantes.)*

### Validation

Chaque ajout est sourcé sur datasheet ou calculé explicitement ; aucun seuil de sécurité posé de mémoire ; toute grandeur dépendant d'un composant non figé reste `NEEDS_DATA`. Aucune coordonnée existante déplacée ; nets vérifiés par inspection MCP.

# Phase C — Gate schématique

## C1 — ERC et revue schématique obligatoire

### Dépendances

B1 validée.

### Tâches

- [x] C1.1 Exécuter ERC et archiver le résultat.
- [x] C1.2 Classer chaque erreur/avertissement et corriger les problèmes réels via MCP.
- [x] C1.3 Relancer ERC et vérifier alimentations, nets critiques, découplages et interfaces.
- [x] C1.4 Consigner explicitement PASS/FAIL du gate ; interdire Phase D si FAIL.

### Validation

Gate schématique explicite, reproductible, sans problème réel ERC non traité.

## C2 — Re-gate ERC après B2

### Objectif

Le gate C1 ne couvre plus le schéma une fois B2 appliquée ; le rejouer est obligatoire.

### Dépendances

B2 validée.

### Tâches

- [x] C2.1 Relancer ERC et comparer à la référence du gate C1 : 0 erreur / 14 avertissements.
- [x] C2.2 Traiter tout écart réel, puis consigner explicitement PASS/FAIL ; interdire la reprise de D1 si FAIL. **GATE C2 = PASS** (0 erreur / 15 avertissements : 5 mismatch librairie, 10 off-grid ; `reports/ERC_C2_gate_final-2026-08-31.json`).

### Validation

Gate schématique de nouveau explicite et reproductible sur le schéma modifié.

# Phase D — Footprints

## D1 — Attribuer et valider les empreintes

### Dépendances

C2 PASS.

### Tâches

- [ ] D1.1 Associer chaque composant à une empreinte compatible et disponible. *(PARTIAL. `U6` corrigé vers une empreinte à PowerPAD ; `D301`, `Q301` et `Q302` assignés. **Inventaire refait au fichier, plus large que ce qui était noté** : 16 composants restent sans empreinte — `C110`, `C210`, `C325`, `C326`, `D302`, `F301`, `J2`, `J3`, `J4`, `R305` à `R310`, `RV1`. Les `PWR_FLAG` n'en requièrent aucune. Le réseau du LM5069 et le clamp de grille, ajoutés en B2.3, B2.4 et B2.6, n'avaient jamais été assignés.)*
- [ ] D1.5 Assigner les passifs du bloc de protection ajoutés en B2 : `R305`, `R307` à `R310` et `C326` sont des passifs standard, mais **`R306` est un shunt de 4 mΩ traversé par 4,6 A en continu et jusqu'à 15,4 A en limitation**, ce qui exige une empreinte de shunt de puissance et non une empreinte générique ; `C325` vaut 3,9 µF et impose un boîtier en conséquence ; `D302` reste `NEEDS_DATA`.
- [ ] D1.4 Créer les empreintes locales manquantes : le fusible `Schurter UMT-H` 5,3 × 16 mm, aucune empreinte KiCad ne couvrant ce corps, et le connecteur d'entrée et de volume déportés. *(À revoir : B2.2 a converti `J2`/`J3` en connecteurs de câblage 2 points vers RCA de châssis, donc des empreintes standard existent peut-être ; la famille de connecteur reste à choisir.)*
- [ ] D1.2 Vérifier boîtiers fabricant, orientations, courants, connecteurs et contraintes d’assemblage.
- [ ] D1.3 Vérifier particulièrement HTSSOP TPA3255, PowerPAD et stratégie de vias thermiques.

### Validation

Toutes les empreintes sont attribuées et revues contre leurs sources fabricant.

# Phase E — PCB 4 couches

## E1 — Définir stack-up, règles et placement

### Dépendances

D1 validée.

### Tâches

- [ ] E1.1 Créer le PCB 4 couches et documenter stack-up/règles/classes de nets.
- [ ] E1.2 Placer puissance Class-D, bootstrap/découplages/bulk et thermique.
- [ ] E1.3 Placer filtres LC, sorties, alimentation et boucles de retour.
- [ ] E1.4 Placer analogique faible bruit, volume et contrôles avec séparation fonctionnelle.
- [ ] E1.5 Revoir symétrie, retour des courants, masses, clearances et manufacturabilité.

### Validation

Placement guidé par contraintes TI et revue de design via workflow disponible.

# Phase F — Routage

## F1 — Router et créer les plans

### Dépendances

E1 validée.

### Tâches

- [ ] F1.1 Router boucles de commutation, alimentation Class-D et découplages.
- [ ] F1.2 Router sorties vers filtres LC et connecteurs avec largeurs justifiées.
- [ ] F1.3 Router analogique, contrôle puis signaux non critiques.
- [ ] F1.4 Créer plans/zones et vias thermiques ; remplir les zones.
- [ ] F1.5 Vérifier l’absence de ratsnest et les longueurs/vias inutiles critiques.

### Validation

Routage terminé, zones remplies, contraintes critiques inspectées.

# Phase G — Validation PCB

## G1 — DRC et design review

### Dépendances

F1 validée.

### Tâches

- [ ] G1.1 Exécuter DRC et archiver le résultat.
- [ ] G1.2 Revoir courts-circuits, alimentations, masses, découplages, puissance et thermique.
- [ ] G1.3 Corriger via MCP, relancer DRC et le workflow de design review.
- [ ] G1.4 Établir la revue manufacturabilité et les écarts résiduels.

### Validation

DRC final et design review documentés après corrections.

# Phase H — Revue électronique et livrables

## H1 — Revue audio/électronique indépendante

### Dépendances

G1 validée ou état final PCB explicitement PARTIAL.

### Tâches

- [ ] H1.1 Vérifier par calcul gain, headroom, impédances, coupures, LC, courants et dissipation.
- [ ] H1.2 Classer preuves : KiCad, documentation, calcul/simulation, non vérifié physiquement.
- [ ] H1.3 Documenter bruit/EMI/stabilité/thermique non mesurés et toute divergence TI.

### Validation

Rapport honnête, traçable, distinct d’ERC/DRC.

## H2 — Produire BOM, rapports et benchmark MCP

### Dépendances

H1.

### Tâches

- [ ] H2.1 Générer BOM et livrables exploitables disponibles.
- [ ] H2.2 Collecter métriques MCP observables, erreurs, retries, limitations et `NEEDS_DATA`.
- [ ] H2.3 Vérifier la présence de tous les livrables demandés.
- [ ] H2.4 Émettre les huit statuts finaux et `PRÊT À FABRIQUER` avec preuves.

### Validation

Livrables présents dans le workspace et rapport final sans métrique ni performance inventée.
