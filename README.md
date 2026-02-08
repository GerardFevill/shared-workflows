# 🚀 Templates CI/CD - GitHub Actions

Templates réutilisables pour tous tes projets avec **Docker Hub** comme registry.

## 📋 Pipelines disponibles

| Template | Stack | Fichier |
|----------|-------|---------|
| Angular | Frontend Angular + Nginx | `ci-angular.yml` |
| Node.js | Backend Node.js (Express/NestJS) | `ci-nodejs.yml` |
| Spring Boot | Backend Java Spring Boot | `ci-spring-boot.yml` |
| iOS | App iOS → TestFlight | `ci-ios.yml` |

## ⚡ Setup rapide (5 min)

### 1. Secrets GitHub à configurer

Dans ton repo → **Settings** → **Secrets and variables** → **Actions** :

#### Pour tous les projets Docker (Angular, Node.js, Spring Boot) :
```
DOCKERHUB_USERNAME    → Ton username Docker Hub
DOCKERHUB_TOKEN       → Token d'accès Docker Hub (pas le mot de passe)
```

#### Pour iOS uniquement :
```
APPLE_CERTIFICATE_BASE64
APPLE_CERTIFICATE_PASSWORD
APPLE_PROVISIONING_PROFILE_BASE64
KEYCHAIN_PASSWORD
APP_STORE_CONNECT_API_KEY_ID
APP_STORE_CONNECT_ISSUER_ID
APP_STORE_CONNECT_API_KEY_BASE64
```

### 2. Copier le workflow

```bash
# Depuis la racine de ton projet
mkdir -p .github/workflows
cp ci-angular.yml .github/workflows/   # ← Adapter selon ta stack
```

### 3. Créer le Dockerfile (pour projets Docker)

Le workflow build l'image à partir du `Dockerfile` à la racine du projet.

### 4. Push & Go

```bash
git add .github/
git commit -m "ci: add CI/CD pipeline"
git push
```

## 🔄 Workflow type

```
Push/PR → Lint → Tests → Security → Build Docker → Push Docker Hub → Summary
```

- **PR** : Lint + Tests uniquement (pas de push d'image)
- **Push main** : Pipeline complète + tag `main-latest`
- **Push develop** : Pipeline complète + tag `develop-latest`

## 📦 Utilisation locale

Après un push sur `main`, récupère l'image en local :

```bash
# Pull la dernière image
docker pull <ton-username>/<nom-du-repo>:main-latest

# Ou dans ton docker-compose.yml
services:
  app:
    image: <ton-username>/<nom-du-repo>:main-latest
    ports:
      - "8080:80"
```

## 🏷️ Convention de tags

| Tag | Quand |
|-----|-------|
| `main-abc12345` | Chaque push sur main (SHA du commit) |
| `main-latest` | Dernière version main |
| `develop-abc12345` | Chaque push sur develop |
| `develop-latest` | Dernière version develop |

## 🛠️ Personnalisation

Chaque template est conçu pour être modifié facilement :
- Versions (Node, Java, Xcode) en variables `env` en haut du fichier
- Services (PostgreSQL, Redis...) ajoutables dans la section `services`
- Steps additionnels (SonarQube, Slack notification...) à ajouter selon les besoins
