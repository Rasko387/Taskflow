# TaskFlow — documentation du TD 1

TaskFlow est l'application Express fournie avec le cours **Environnement de développement & outillage**. Elle permet de créer des tâches, de les rechercher, de les filtrer par statut, de faire avancer leur statut et de les supprimer.

Le TD 1, appelé **TP 1** aux pages 31–32, porte sur Git, l'environnement de travail, la configuration externe, les services Docker et la documentation. Les trois documents présentés page 30 sont disponibles ici : ce README, [CONTRIBUTING.md](CONTRIBUTING.md) et [l'ADR 0001](docs/adr/0001-environnement.md).

## Organisation de la copie locale

```text
taskflow-starter/                 # Dossier parent : environnement du TD 1
├── .devcontainer/
├── .env.example
├── compose.yaml
├── scripts/init.sql
└── taskflow-starter/             # Application : ce README et les commandes npm
    ├── README.md
    ├── CONTRIBUTING.md
    ├── docs/adr/
    ├── public/
    ├── server.js
    ├── package.json
    └── package-lock.json
```

Les commandes npm ci-dessous se lancent dans le **dossier de l'application**, celui qui contient ce README et `package.json`. Les commandes Docker se lancent dans le **dossier parent**, celui qui contient `compose.yaml`.

La copie actuelle contient deux dépôts Git imbriqués. Un commit effectué dans l'application n'inclut pas les fichiers du dossier parent. Avant de transmettre le projet à un autre étudiant, vérifier que la version partagée contient aussi Compose, le Dev Container, `.env.example` et le SQL. Le distant configuré est [Rasko387/Taskflow](https://github.com/Rasko387/Taskflow) ; sa seule URL ne garantit pas que les changements locaux y sont déjà publiés.

## Prérequis

- Node.js **24.x**, version déclarée dans `package.json` et le fichier `.nvmrc` du dossier parent.
- Git.
- Docker Desktop démarré, avec les conteneurs Linux et Docker Compose.
- Pour le Dev Container : VS Code et l'extension **Dev Containers**.

Le premier lancement nécessite Internet pour télécharger les images et dépendances.

## Démarrer sur son ordinateur

1. Depuis le dossier parent, copier `.env.example` vers `.env` s'il n'existe pas encore. Sous PowerShell :

   ```powershell
   if (-not (Test-Path .env)) { Copy-Item .env.example .env }
   ```

2. Ouvrir `.env` et choisir un mot de passe PostgreSQL local avant le premier démarrage de la base.
3. Toujours depuis le dossier parent :

   ```sh
   docker compose up -d --wait postgres redis
   ```

4. Entrer dans le dossier de l'application, puis installer et démarrer :

   ```sh
   cd taskflow-starter
   npm ci --ignore-scripts
   npm start
   ```

5. Ouvrir [TaskFlow](http://localhost:3000). Arrêter le serveur avec `Ctrl+C`.

`npm ci` utilise les versions du fichier `package-lock.json`. L'option `--ignore-scripts` permet ici de reproduire le TD 1 sans exécuter la préparation Husky du TD 2 déjà présente dans le projet. Le travail sur les hooks suit ensuite les consignes du TD 2.

## Utiliser le Dev Container

Créer `.env` sur l'ordinateur comme indiqué ci-dessus, puis ouvrir le **dossier parent** dans VS Code. Avec Docker Desktop démarré, lancer **Dev Containers: Reopen in Container** dans la palette de commandes.

La configuration fournit Node.js 24, les extensions ESLint et Prettier, et démarre PostgreSQL et Redis. La commande `postCreateCommand` installe les dépendances dans le sous-dossier de l'application.

Dans le terminal ouvert à la racine de l'espace de travail :

```sh
cd taskflow-starter
npm start
```

VS Code transfère le port 3000. Si `PORT` est changé, transférer également le nouveau port dans l'onglet **Ports**.

## Configuration externe

Le script `npm start` demande à Node de charger `../.env`. Une variable déjà définie dans le terminal a priorité. Redémarrer le serveur après une modification.

| Variable     | Valeur d'exemple           | Utilisation                        |
| ------------ | -------------------------- | ---------------------------------- |
| `PORT`       | `3000`                     | Port HTTP du serveur.              |
| `PGUSER`     | `taskflow`                 | Utilisateur initial de PostgreSQL. |
| `PGPASSWORD` | Valeur fictive à remplacer | Mot de passe local de PostgreSQL.  |
| `PGDATABASE` | `taskflow`                 | Base créée au premier démarrage.   |
| `PGPORT`     | `5432`                     | Port PostgreSQL sur l'ordinateur.  |
| `REDIS_PORT` | `6379`                     | Port Redis sur l'ordinateur.       |

`.env` est ignoré par Git. Seul `.env.example`, avec des valeurs fictives, doit être partagé. L'ancienne clé inutilisée en dur et son affichage dans les logs ont été retirés.

## PostgreSQL, Redis et données

Compose fournit PostgreSQL 16 et Redis 7 sur un réseau commun. Les contrôles de santé permettent d'attendre leur disponibilité. Les ports sont publiés sur `127.0.0.1` et les données des services sont conservées dans des volumes Docker.

**L'application garde son stockage en mémoire.** Les tâches de l'interface ne sont pas enregistrées dans PostgreSQL et disparaissent au redémarrage du serveur. Redis est préparé comme service ; le code applicatif ne l'utilise pas encore.

Le fichier `../scripts/init.sql` crée la table `tasks` au premier démarrage d'un volume PostgreSQL vide. Il contient un identifiant automatique, un titre obligatoire, une description, un statut (`todo`, `in-progress`, `done`), un responsable et une date de création. Trois lignes d'exemple sont ajoutées.

Depuis le dossier parent, vérifier les services :

```sh
docker compose ps
docker compose exec postgres sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "SELECT count(*) FROM tasks;"'
docker compose exec redis redis-cli ping
```

Sur une base neuve, les résultats attendus sont des services `healthy`, trois lignes dans `tasks` et la réponse Redis `PONG`.

Pour arrêter les services en conservant leurs données :

```sh
docker compose stop postgres redis
```

Le script SQL ne se relance pas automatiquement sur un volume déjà initialisé. Changer `PGPASSWORD` dans `.env` ne modifie pas le mot de passe déjà enregistré dans ce volume.

## Qualité et vérifications du TD 1

Depuis le dossier de l'application :

| Commande               | Rôle                                          |
| ---------------------- | --------------------------------------------- |
| `npm run lint`         | Analyse le JavaScript avec ESLint.            |
| `npm run lint:fix`     | Applique les corrections automatiques ESLint. |
| `npm run format:check` | Vérifie le formatage avec Prettier.           |
| `npm run format`       | Applique le formatage.                        |
| `npm run check`        | Enchaîne ESLint et la vérification Prettier.  |

Vérifier aussi manuellement l'ajout d'une tâche, les filtres, l'avancement de statut et la suppression. Les scripts `test` et `test:ci`, le dossier `test/`, les hooks et le workflow GitHub Actions appartiennent au TD 2 en cours ; leur présence ne prouve pas que ces étapes sont terminées.

## Réception par un autre étudiant

La page 32 demande un échange de dépôts. Fournir la version complète du projet et demander à l'étudiant de suivre ce README sans assistance. Vérifier le démarrage, la présence des services et du schéma, l'interface, puis `npm run check`. Noter les difficultés rencontrées et corriger la documentation. Cette réception reste à effectuer ; elle n'est pas remplacée par les vérifications locales.

## Dépannage

| Problème                           | Vérification                                                                   |
| ---------------------------------- | ------------------------------------------------------------------------------ |
| `package.json` introuvable         | Entrer dans le sous-dossier de l'application avant une commande npm.           |
| `compose.yaml` introuvable         | Revenir dans le dossier parent pour les commandes Docker.                      |
| Docker ne répond pas               | Démarrer Docker Desktop et vérifier `docker version`.                          |
| Port déjà utilisé                  | Modifier le port concerné dans `.env`, puis relancer le serveur ou le service. |
| PostgreSQL refuse les identifiants | Vérifier ceux utilisés lors de la création du volume.                          |
| Table `tasks` absente              | Vérifier le montage du SQL et les logs du premier démarrage de PostgreSQL.     |

Sur ce poste Windows, Docker Desktop est installé pour l'utilisateur. Si sa commande n'est pas trouvée, ajouter son dossier au PATH du terminal courant :

```powershell
$env:Path += ";$env:LOCALAPPDATA\Programs\DockerDesktop\resources\bin"
```

Si VS Code ne trouve pas Docker, le lancer depuis ce terminal. Pour contribuer, suivre [CONTRIBUTING.md](CONTRIBUTING.md).
