---
name: plan-refactoring
description: Crée un plan de refactoring détaillé avec des micro-commits via un entretien structuré avec l'utilisateur, puis le dépose sous forme d'issue GitHub. Utilise cette compétence dès que l'utilisateur veut planifier un refactoring, créer une RFC de refactoring, découper un refactoring en étapes sûres et incrémentales, ou parle de "dette technique à traiter", "restructurer le code", "nettoyer le code", "réorganiser l'architecture". Même si l'utilisateur ne dit pas explicitement "refactoring", si la demande implique de transformer du code existant sans changer le comportement fonctionnel, cette compétence doit se déclencher.
---

# Plan de Refactoring

Cette compétence guide un processus structuré pour transformer une idée de refactoring en un plan d'implémentation concret, découpé en micro-commits, et déposé comme issue GitHub.

L'objectif est d'appliquer le conseil de Martin Fowler : **"rendre chaque étape de refactoring aussi petite que possible, pour que le programme reste toujours en état de marche."**

## Déroulement

Suis les étapes ci-dessous dans l'ordre. Tu peux sauter une étape si elle n'est clairement pas pertinente pour le cas traité, mais par défaut, passe par chacune d'elles.

### Étape 1 — Comprendre le problème

Demande à l'utilisateur une description longue et détaillée du problème qu'il veut résoudre, ainsi que ses éventuelles pistes de solution.

Ne te contente pas d'un résumé vague. Pose des questions de relance pour obtenir :
- Le contexte métier derrière le refactoring (pourquoi maintenant ?)
- Les symptômes concrets du problème (lenteur, bugs récurrents, difficulté à faire évoluer le code, duplication…)
- Ce que l'utilisateur a déjà tenté ou envisagé

### Étape 2 — Explorer le dépôt

Explore le code source pour vérifier les affirmations de l'utilisateur et comprendre l'état actuel de la base de code. Regarde :
- La structure des modules/dossiers concernés
- Les dépendances entre les parties impactées
- Les patterns utilisés actuellement
- Les points de couplage fort

Résume ce que tu as trouvé et signale toute divergence avec ce que l'utilisateur a décrit.

### Étape 3 — Alternatives

Demande à l'utilisateur s'il a envisagé d'autres approches. Propose-lui toi-même des alternatives que tu as identifiées en explorant le code. L'idée est de s'assurer qu'on ne fonce pas tête baissée dans une solution alors qu'une approche plus simple ou moins risquée existe.

Pour chaque alternative, expose brièvement les avantages et inconvénients.

### Étape 4 — Entretien d'implémentation

Interroge l'utilisateur en profondeur sur les détails d'implémentation. Sois extrêmement minutieux. Les questions typiques :
- Quels modules/classes sont concernés ?
- Quelles interfaces doivent changer ? Lesquelles doivent rester stables ?
- Y a-t-il des contraintes de rétrocompatibilité ?
- Des migrations de données ou de schéma sont-elles nécessaires ?
- Quels sont les contrats d'API à respecter ?
- Y a-t-il des interactions spécifiques entre composants à préserver ?

### Étape 5 — Délimiter le périmètre

Définis précisément ce qui est dans le périmètre du refactoring et ce qui en est exclu. C'est une étape critique pour éviter le scope creep. Formule clairement :
- Ce qu'on va modifier
- Ce qu'on ne va **pas** modifier (et pourquoi)
- Les frontières exactes de l'intervention

### Étape 6 — Couverture de tests

Examine la base de code pour évaluer la couverture de tests de la zone concernée par le refactoring.

- S'il y a des tests existants : identifie-les, évalue s'ils couvrent bien le comportement externe (pas les détails d'implémentation)
- S'il n'y a pas assez de tests : alerte l'utilisateur et demande-lui sa stratégie de test. Un refactoring sans filet de tests, c'est du funambulisme sans filet.

### Étape 7 — Découper en micro-commits

C'est le cœur du plan. Découpe l'implémentation en commits aussi petits que possible. Chaque commit doit :
- Laisser la base de code dans un état fonctionnel (les tests passent)
- Être compréhensible de façon isolée
- Avoir un objectif clair et unique

Décris chaque commit en langage clair, sans snippets de code (ils seraient vite obsolètes). Privilégie des descriptions comme "Extraire la logique de calcul de prix dans une classe dédiée PriceCalculator" plutôt que "Modifier le fichier X ligne Y".

### Étape 8 — Créer l'issue GitHub

Crée une issue GitHub avec le plan de refactoring complet en utilisant le modèle ci-dessous.

## Modèle d'issue

\`\`\`markdown
## Énoncé du problème
Le problème rencontré par le développeur, décrit de son point de vue.

## Solution retenue
La solution choisie, décrite de façon synthétique.

## Plan de commits
Un plan d'implémentation LONG et détaillé, rédigé en langage clair. Chaque commit est décrit de façon à ce que la base de code reste fonctionnelle après chaque étape.

## Document de décisions
Liste des décisions d'implémentation prises au cours de l'entretien :
- Les modules qui seront créés/modifiés
- Les interfaces qui seront modifiées
- Les clarifications techniques obtenues auprès du développeur
- Les décisions d'architecture
- Les changements de schéma
- Les contrats d'API
- Les interactions spécifiques identifiées

NE PAS inclure de chemins de fichiers ni de snippets de code spécifiques — ils risquent d'être rapidement obsolètes.

## Décisions de test
Liste des décisions relatives aux tests :
- Description de ce qui fait un bon test (tester le comportement externe, pas les détails d'implémentation)
- Quels modules seront testés
- Tests existants similaires dans la base de code (références)

## Hors périmètre
Description de ce qui est explicitement exclu du refactoring et pourquoi.

## Notes complémentaires (optionnel)
Toute information supplémentaire utile concernant le refactoring.
\`\`\`

## Conseils pour bien mener l'entretien

- **Ne précipite pas les choses.** Chaque étape a son importance. Un plan de refactoring bâclé mène à un refactoring bâclé.
- **Sois concret.** Quand tu explores le code, cite ce que tu trouves. Quand tu proposes des alternatives, donne des exemples concrets.
- **Pense aux risques.** À chaque étape, demande-toi ce qui pourrait mal tourner et assure-toi que le plan en tient compte.
- **Le plus petit commit possible.** En cas de doute sur la taille d'un commit, découpe-le en deux. Il vaut toujours mieux trop petit que trop gros.
