---
name: design-interface
description: "Génère plusieurs designs d'interface radicalement différents pour un module en utilisant des sous-agents parallèles, puis les compare. Utilise cette compétence dès que l'utilisateur veut designer une API, explorer des options d'interface, comparer des formes de module, ou mentionne "design it twice", "propose-moi plusieurs approches", "quelles sont mes options d'architecture", "compare des designs", "montre-moi des alternatives d'interface". Également pertinent quand l'utilisateur hésite entre plusieurs façons de structurer un composant ou un service."
---

# Designer une Interface

Basé sur le principe "Design It Twice" de *A Philosophy of Software Design* : ta première idée n'est probablement pas la meilleure. Génère plusieurs designs radicalement différents, puis compare.

## Déroulement

### Étape 1 — Recueillir les besoins

Avant de designer quoi que ce soit, comprends le contexte. Pose ces questions (une à la fois) :

- Quel problème ce module résout-il ?
- Qui sont les appelants ? (autres modules, utilisateurs externes, tests)
- Quelles sont les opérations clés ?
- Y a-t-il des contraintes ? (performance, compatibilité, patterns existants dans le code)
- Qu'est-ce qui doit rester caché à l'intérieur vs exposé à l'extérieur ?

Si un dépôt de code existe, explore-le pour identifier les conventions en place et les patterns déjà utilisés plutôt que de poser des questions dont la réponse est dans le code.

### Étape 2 — Générer les designs (sous-agents parallèles)

Lance 3 sous-agents ou plus en parallèle via l'outil Task. Chaque sous-agent doit produire une approche **radicalement différente**. La valeur de l'exercice vient de la diversité — si les designs se ressemblent, l'exercice ne sert à rien.

Pour forcer la divergence, assigne à chaque agent une contrainte différente :

- **Agent 1** : "Minimise le nombre de méthodes — vise 1 à 3 méthodes maximum"
- **Agent 2** : "Maximise la flexibilité — supporte un maximum de cas d'usage"
- **Agent 3** : "Optimise pour le cas d'usage le plus courant"
- **Agent 4** (optionnel) : "Inspire-toi de [paradigme/bibliothèque spécifique]"

Chaque sous-agent doit produire :
1. La signature de l'interface (types, méthodes, paramètres)
2. Un exemple d'utilisation concret (comment l'appelant s'en sert)
3. Ce que le design cache en interne
4. Les compromis de cette approche

### Étape 3 — Présenter les designs

Présente chaque design l'un après l'autre pour que l'utilisateur puisse absorber chaque approche avant la comparaison. Pour chaque design, montre :

1. **Signature de l'interface** — types, méthodes, paramètres
2. **Exemples d'utilisation** — comment les appelants s'en servent concrètement
3. **Ce que ça cache** — la complexité maintenue à l'intérieur

### Étape 4 — Comparer les designs

Après avoir montré tous les designs, compare-les sur ces axes :

- **Simplicité de l'interface** : moins de méthodes, paramètres plus simples = plus facile à apprendre et à utiliser correctement
- **Généraliste vs spécialisé** : flexibilité vs focus
- **Efficacité d'implémentation** : est-ce que la forme de l'interface permet une implémentation efficace, ou force des contorsions internes ?
- **Profondeur** : une petite interface qui cache beaucoup de complexité = module profond (bien). Une grosse interface avec une implémentation mince = module superficiel (à éviter)
- **Facilité d'utilisation correcte** vs **facilité de mauvais usage**

Discute les compromis en prose, pas en tableau. Mets en lumière les points de divergence les plus forts entre les designs.

### Étape 5 — Synthétiser

Souvent le meilleur design combine des éléments de plusieurs options. Demande :

- "Quel design correspond le mieux à ton cas d'usage principal ?"
- "Y a-t-il des éléments d'autres designs qui méritent d'être intégrés ?"

Propose un design final qui prend le meilleur de chaque approche si pertinent.

## Critères d'évaluation

Tirés de *A Philosophy of Software Design* :

**Simplicité de l'interface** — Moins de méthodes, paramètres plus simples = plus facile à apprendre et à utiliser correctement.

**Généralité** — Capable de gérer des cas d'usage futurs sans modification. Mais attention à la sur-généralisation.

**Efficacité d'implémentation** — La forme de l'interface permet-elle une implémentation efficace ? Ou force-t-elle des mécanismes internes maladroits ?

**Profondeur** — Une petite interface cachant une complexité significative = module profond (bon). Une grande interface avec une implémentation mince = module superficiel (à éviter).

## Anti-patterns

- **Designs trop similaires** — Si les sous-agents produisent des designs proches, l'exercice perd tout son intérêt. Force la divergence radicale.
- **Sauter la comparaison** — La valeur est dans le contraste entre les approches, pas dans un design isolé.
- **Implémenter** — Cette compétence porte uniquement sur la forme de l'interface. On ne code rien ici.
- **Évaluer sur l'effort d'implémentation** — Un design plus difficile à implémenter n'est pas forcément un mauvais design. Ce qui compte, c'est la qualité de l'interface pour l'appelant.
