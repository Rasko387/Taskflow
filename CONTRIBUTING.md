# Contribuer à TaskFlow

Cette documentation applique les conventions de contribution présentées au TD 1. Le démarrage est décrit dans [README.md](README.md).

## Préparer une modification

1. Utiliser Node.js 24 ou le Dev Container.
2. Vérifier le dépôt actif avec `git rev-parse --show-toplevel` et les changements avec `git status`.
3. Créer une branche au nom descriptif, par exemple `git switch -c fix/filtre-statut`.
4. Réaliser une modification cohérente, puis mettre à jour la documentation si elle change une commande ou une configuration.
5. Exécuter `npm run check` depuis le dossier de l'application et vérifier le comportement concerné.
6. Examiner `git diff`, ajouter explicitement les fichiers concernés et créer un commit.
7. Pousser la branche et ouvrir une pull request avec le problème corrigé, le résultat attendu et les vérifications réalisées. Faire relire avant fusion.

La copie locale contient un dépôt dans le dossier parent et un autre dans l'application. Ils ont des index distincts : un commit de l'application n'emporte pas Compose ni la configuration située dans le parent. Vérifier la présence de tous les fichiers du TD 1 dans la version transmise.

## Nommage et style

- Variables et fonctions JavaScript en `camelCase` ; modules avec des noms descriptifs.
- Variables d'environnement en `MAJUSCULES_AVEC_UNDERSCORES` et colonnes SQL en `snake_case`.
- Deux espaces d'indentation, points-virgules, guillemets simples JavaScript et fins de ligne LF, selon la configuration Prettier existante.
- Conserver les configurations ESLint et Prettier communes au projet.
- Messages et documentation destinés à l'utilisateur en français.

Utiliser des messages de commit explicites, par exemple `fix: corriger le filtre de statut`, `docs: préciser le démarrage` ou `chore: ajuster la configuration`. Ce sont les conventions proposées ; un contrôle automatique des messages fait partie du travail sur les hooks du TD 2.

## Configuration et dépendances

Ne pas versionner `.env`, `node_modules/` ni les logs. Partager les variables attendues dans `.env.example` avec des valeurs fictives et mettre à jour leur description dans le README.

Après une modification des dépendances, conserver ensemble `package.json` et `package-lock.json`. Utiliser l'installation verrouillée pour vérifier qu'un autre développeur retrouve les mêmes dépendances.

Documenter une décision technique importante dans un ADR sous `docs/adr/` : contexte, options envisagées, décision retenue et conséquences. L'[ADR 0001](docs/adr/0001-environnement.md) décrit l'environnement du TD 1.

## Validation

Les commandes de vérification et la procédure de réception par un autre étudiant se trouvent dans le README. Ne déclarer un contrôle réussi qu'après l'avoir exécuté. Les tests, les hooks avancés et la CI du TD 2 se complètent au fil du cours.
