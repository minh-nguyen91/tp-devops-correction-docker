# simple-api-devops

API REST Spring Boot qui gère des étudiants et des départements, stockés dans une base PostgreSQL.
C'est le backend du TP DevOps : il tourne derrière un reverse proxy Apache (`http-server`), avec la base `database`, via le `docker-compose.yaml` à la racine du dépôt.

## Stack

- Java 21 (Amazon Corretto)
- Spring Boot 3.4 (Web, Data JPA, Actuator)
- PostgreSQL 14
- Maven
- Tests : JUnit 5, MockMvc, Testcontainers (PostgreSQL)
- Couverture : JaCoCo, analyse qualité : SonarCloud

## Endpoints

### Greeting

| Méthode | Route | Description |
|---|---|---|
| GET | `/?name=Bob` | Renvoie `{"id": 1, "content": "Hello, Bob!"}` (`name` vaut `World` par défaut) |

### Étudiants

| Méthode | Route | Description |
|---|---|---|
| GET | `/students` | Liste tous les étudiants |
| GET | `/students/{id}` | Un étudiant (404 s'il n'existe pas) |
| POST | `/students` | Crée un étudiant (201 + header `Location`, 400 si `lastname` ou `department` manque) |
| PUT | `/students/{id}` | Met à jour un étudiant (404 s'il n'existe pas) |
| DELETE | `/students/{id}` | Supprime un étudiant (404 s'il n'existe pas) |

Exemple de corps pour `POST` / `PUT` :

```json
{
  "firstname": "Eli",
  "lastname": "Copter",
  "department": { "id": 1 }
}
```

### Départements

| Méthode | Route | Description |
|---|---|---|
| GET | `/departments` | Liste tous les départements |
| GET | `/departments/{name}` | Un département par son nom, ex. `IRC` (404 s'il n'existe pas) |
| GET | `/departments/{name}/students` | Les étudiants d'un département |
| GET | `/departments/{name}/count` | Le nombre d'étudiants d'un département |

### Actuator

Exposés sous `/actuator` : `health`, `info`, `env`, `metrics`, `beans`, `configprops`.

## Configuration

La connexion à la base se configure par variables d'environnement (voir [application.yml](src/main/resources/application.yml)) :

| Variable | Rôle |
|---|---|
| `DATABASE_HOST` | Hôte PostgreSQL (`database` dans le compose) |
| `DATABASE_PASSWORD` | Mot de passe de l'utilisateur `usr` |

La base s'appelle `db`, l'utilisateur `usr`, le port est `5432`. Le schéma et les données initiales sont créés par les scripts de [database/sql](../database/sql) au premier démarrage du conteneur PostgreSQL ; l'application ne génère pas le schéma (`generate-ddl: false`).

## Lancer le projet

### Avec Docker Compose (recommandé)

Depuis la racine du dépôt :

```bash
docker compose up --build
```

L'API est alors accessible sur http://localhost via le reverse proxy Apache (port 80), par exemple http://localhost/students.

### En local

Avec une base PostgreSQL accessible :

```bash
DATABASE_HOST=localhost DATABASE_PASSWORD=pwd mvn spring-boot:run
```

L'API écoute sur http://localhost:8080.

## Tests

```bash
mvn clean verify
```

- Tests unitaires (`*Test`) : services, avec Mockito.
- Tests d'intégration (`*TestIT`) : contrôleurs avec MockMvc, contre une vraie base PostgreSQL lancée par Testcontainers. **Docker doit tourner** sur la machine.
- Le rapport de couverture JaCoCo est généré dans `target/site/jacoco`.

## Image Docker

Le [Dockerfile](Dockerfile) est en deux étapes : build du jar avec Maven (tests ignorés), puis exécution sur une image `amazoncorretto:21` qui ne contient que le jar.

```bash
docker build -t simple-api .
```

## CI/CD

Deux workflows GitHub Actions dans [.github/workflows](../.github/workflows) :

- **CI** ([main.yml](../.github/workflows/main.yml)) : sur chaque push sur `main` / `develop` et sur chaque pull request. Build et tests (`mvn clean verify`) puis analyse SonarCloud.
- **CD** ([cd.yml](../.github/workflows/cd.yml)) : se déclenche quand la CI réussit après un push sur `main`. Construit et pousse sur Docker Hub les images `tp-devops-simple-api`, `tp-devops-database` et `tp-devops-httpd` (tag `latest`).

Secrets GitHub nécessaires :

| Secret | Utilisé par |
|---|---|
| `SONAR_TOKEN`, `PROJECT_KEY`, `ORGANIZATION_KEY` | CI (SonarCloud) |
| `DOCKER_HUB_USERNAME`, `DOCKER_HUB_TOKEN` | CD (environnement `docker_hub`) |
