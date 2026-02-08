# Shared CI/CD Workflows - GitHub Actions

Workflows **reusable** pour tous tes projets avec **Docker Hub** comme registry.

## Pipelines disponibles

| Workflow | Stack | Fichier |
|----------|-------|---------|
| Angular | Frontend Angular | `.github/workflows/ci-angular.yml` |
| Node.js | Backend Node.js (Express/NestJS) | `.github/workflows/ci-nodejs.yml` |

## Setup rapide

### 1. Secrets GitHub

Dans ton repo → **Settings** → **Secrets and variables** → **Actions** :

```
DOCKERHUB_USERNAME    → Ton username Docker Hub
DOCKERHUB_TOKEN       → Token d'accès Docker Hub (pas le mot de passe)
```

### 2. Appeler le workflow depuis ton projet

Crée un fichier dans `.github/workflows/` de ton projet :

#### Projet simple (un seul service)

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: GerardFevill/shared-workflows/.github/workflows/ci-angular.yml@main
    secrets: inherit
```

#### Monorepo (frontend + backend)

```yaml
# .github/workflows/ci-frontend.yml
name: CI/CD Frontend

on:
  push:
    branches: [main, develop]
    paths: ['frontend/**']
  pull_request:
    branches: [main]
    paths: ['frontend/**']

jobs:
  ci:
    uses: GerardFevill/shared-workflows/.github/workflows/ci-angular.yml@main
    with:
      working-directory: frontend
      image-suffix: -frontend
    secrets: inherit
```

```yaml
# .github/workflows/ci-backend.yml
name: CI/CD Backend

on:
  push:
    branches: [main, develop]
    paths: ['backend/**']
  pull_request:
    branches: [main]
    paths: ['backend/**']

jobs:
  ci:
    uses: GerardFevill/shared-workflows/.github/workflows/ci-nodejs.yml@main
    with:
      working-directory: backend
      image-suffix: -backend
    secrets: inherit
```

### 3. Push & Go

```bash
git add .github/
git commit -m "ci: add CI/CD pipeline"
git push
```

## Inputs disponibles

### ci-angular.yml

| Input | Description | Default |
|-------|-------------|---------|
| `working-directory` | Dossier des sources (monorepo) | `.` |
| `node-version` | Version Node.js | `20` |
| `image-suffix` | Suffixe image Docker (ex: `-frontend`) | `` |

### ci-nodejs.yml

| Input | Description | Default |
|-------|-------------|---------|
| `working-directory` | Dossier des sources (monorepo) | `.` |
| `node-version` | Version Node.js | `20` |
| `image-suffix` | Suffixe image Docker (ex: `-backend`) | `` |

> Le workflow Node.js inclut un service **PostgreSQL 16** pour les tests d'integration.

## Pipeline

```
Push/PR → Lint → Tests → [Security Audit] → Build Docker → Push Docker Hub → Summary
```

- **PR** : Lint + Tests uniquement (pas de push d'image)
- **Push main** : Pipeline complete + tag `main-latest`
- **Push develop** : Pipeline complete + tag `develop-latest`

## Convention de tags Docker

| Tag | Quand |
|-----|-------|
| `main-abc12345` | Chaque push sur main (SHA du commit) |
| `main-latest` | Derniere version main |
| `develop-abc12345` | Chaque push sur develop |
| `develop-latest` | Derniere version develop |

## Utilisation locale

```bash
docker pull <ton-username>/<nom-du-repo>:main-latest

# Ou dans docker-compose.yml
services:
  app:
    image: <ton-username>/<nom-du-repo>:main-latest
    ports:
      - "8080:80"
```
