# ADR 0001 — Environnement de développement du TD 1

Date : 29 septembre 2026. Statut : retenu pour le TD 1.

## Contexte

Le cours fournit une application Express fonctionnelle avec un stockage de tâches en mémoire. Le TD 1 demande un environnement reproductible, un linter et un formatter, une configuration externe, PostgreSQL et Redis via Docker Compose avec le schéma `tasks`, ainsi qu'une documentation de démarrage.

## Options envisagées

| Option                                                            | Intérêt                                                       | Limite                                                                        |
| ----------------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Installer tous les outils et services séparément sur chaque poste | Utilisation directe des outils locaux.                        | Versions et configuration à reproduire manuellement.                          |
| Node.js local avec PostgreSQL et Redis dans Compose               | Services décrits dans le dépôt et démarrage commun.           | Node.js doit être installé dans la bonne version sur chaque poste.            |
| Dev Container Node.js avec les services Compose                   | Runtime et outils de développement décrits avec les services. | Nécessite Docker et un éditeur compatible ; premier téléchargement plus long. |

## Décision

Fournir un Dev Container avec Node.js 24 et les extensions ESLint/Prettier. Garder possible le lancement avec Node.js 24 sur l'ordinateur. Utiliser les configurations de qualité déjà présentes dans l'application.

Décrire PostgreSQL 16 et Redis 7 dans Compose avec un réseau commun, des contrôles de santé et des volumes. Initialiser `tasks` au premier démarrage PostgreSQL avec le fichier SQL du projet.

Placer la configuration locale dans `.env`, ignoré par Git, et fournir `.env.example` avec des valeurs fictives. Le serveur lit son port depuis l'environnement.

Conserver Express, l'interface et le stockage en mémoire du projet fourni. Le raccordement applicatif aux services n'est pas nécessaire pour préparer l'infrastructure demandée dans ce TD 1.

## Conséquences

- Le lancement des services et la configuration du poste de développement sont documentés et partageables avec le code.
- Docker Desktop doit fonctionner ; le premier lancement nécessite le téléchargement des images.
- Les données des services persistent dans leurs volumes. Les tâches créées dans l'interface restent temporaires tant que l'application utilise sa mémoire.
- `.env` n'est pas transmis avec le dépôt ; chaque développeur crée sa propre configuration à partir de l'exemple.
- Les images suivent les versions indiquées par leurs tags et les dépendances npm sont verrouillées dans `package-lock.json`.
- La structure actuelle avec deux dépôts imbriqués demande de vérifier que l'environnement du dossier parent accompagne le code lors du partage.
- La reproductibilité doit encore être confirmée par la réception chez un autre étudiant demandée page 32.
