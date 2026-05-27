---
title: "GitLab CI/CD"
tags:
  - gitlab
  - ci-cd
  - automatisation
  - devops
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[github_actions]]
  - [[02_setup_gitea_jenkins]]
  - [[trunk-based-development]]
  - [[container_registry]]
  - [[tests_unitaires]]
  - [[fiche_git]]
---

# 📘 Fiche : GitLab CI/CD

---

## 1. Qu'est-ce que GitLab CI/CD ?

GitLab CI/CD est la plateforme d'intégration et de déploiement continus intégrée à GitLab. La configuration se fait dans un fichier `.gitlab-ci.yml` à la racine du dépôt.

| GitHub Actions | GitLab CI/CD |
|---|---|
| `.github/workflows/*.yml` | `.gitlab-ci.yml` (un seul fichier) |
| Jobs → Steps → Actions | Stages → Jobs → Scripts |
| Runners GitHub-hosted | Runners GitLab.com ou self-hosted |
| Secrets dans GitHub Settings | Variables dans GitLab CI/CD Settings |

---

## 2. Concepts clés

| Concept | Définition |
|---|---|
| **Pipeline** | Ensemble de stages déclenchés par un événement |
| **Stage** | Phase du pipeline (ex: test, build, deploy) — les stages s'exécutent en séquence |
| **Job** | Unité d'exécution dans un stage — les jobs d'un même stage tournent en parallèle |
| **Runner** | Agent qui exécute les jobs (GitLab.com ou self-hosted) |
| **Artifact** | Fichier produit par un job, transmissible aux jobs suivants |
| **Cache** | Dossiers mis en cache entre pipelines pour accélérer les builds |

---

## 3. Structure de base

```yaml
# .gitlab-ci.yml

stages:          # ordre d'exécution
  - test
  - build
  - deploy

variables:
  NODE_ENV: production

test:
  stage: test
  image: node:20-alpine
  script:
    - npm ci
    - npm test

build:
  stage: build
  image: docker:latest
  script:
    - docker build -t monapp:$CI_COMMIT_SHA .

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main                 # seulement sur la branche main
```

---

## 4. Variables prédéfinies utiles

GitLab injecte automatiquement des variables dans chaque pipeline :

| Variable | Valeur |
|---|---|
| `$CI_COMMIT_SHA` | SHA du commit (ex: `abc1234...`) |
| `$CI_COMMIT_SHORT_SHA` | SHA court (8 caractères) |
| `$CI_COMMIT_BRANCH` | Nom de la branche |
| `$CI_COMMIT_TAG` | Tag git (si pipeline déclenché par un tag) |
| `$CI_PROJECT_NAME` | Nom du projet |
| `$CI_REGISTRY` | URL du container registry GitLab |
| `$CI_REGISTRY_IMAGE` | URL complète de l'image du projet |
| `$CI_PIPELINE_ID` | ID unique du pipeline |

---

## 5. Variables et secrets

Les secrets se configurent dans **Settings → CI/CD → Variables** du projet.

```yaml
deploy:
  stage: deploy
  script:
    - echo "Déploiement vers $DEPLOY_SERVER"
    - ssh deploy@$DEPLOY_SERVER "./deploy.sh"
  environment:
    name: production
```

> Les variables marquées **Protected** ne sont disponibles que sur les branches/tags protégés. Les variables **Masked** sont cachées dans les logs.

---

## 6. Exemple complet — CI/CD Node.js + Docker + GitLab Registry

```yaml
stages:
  - test
  - build
  - deploy

variables:
  IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

test:
  stage: test
  image: node:20-alpine
  cache:
    key: $CI_COMMIT_REF_SLUG
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run lint
    - npm test
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'   # regex pour extraire la couverture
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind                           # Docker-in-Docker
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE .
    - docker push $IMAGE
  only:
    - main
    - tags

deploy:
  stage: deploy
  image: alpine
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | ssh-add -
  script:
    - ssh deploy@$DEPLOY_HOST "docker pull $IMAGE && docker compose up -d"
  environment:
    name: production
    url: https://monapp.exemple.com
  only:
    - main
```

---

## 7. Règles de déclenchement

```yaml
# Méthode moderne : rules (remplace only/except)
deploy:
  script: ./deploy.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"      # seulement sur main
    - if: $CI_COMMIT_TAG                   # ou sur un tag
      when: manual                         # déclenchement manuel

# Exécution manuelle
release:
  script: ./release.sh
  when: manual
  allow_failure: false
```

---

## 8. Artifacts et cache

```yaml
test:
  script:
    - npm test
  artifacts:
    paths:
      - coverage/          # fichiers à conserver
    expire_in: 1 week      # durée de rétention
    reports:
      junit: junit.xml     # rapport de tests affiché dans GitLab

build:
  cache:
    key: $CI_COMMIT_REF_SLUG    # clé par branche
    paths:
      - node_modules/
      - .npm/
```

---

## 9. Comparaison GitHub Actions vs GitLab CI

| | GitHub Actions | GitLab CI |
|---|---|---|
| Fichier config | `.github/workflows/*.yml` | `.gitlab-ci.yml` |
| Structure | Workflow → Job → Step | Pipeline → Stage → Job |
| Parallélisme | Jobs indépendants par défaut | Jobs du même stage en parallèle |
| Registry intégré | GitHub Container Registry | GitLab Container Registry |
| Self-hosted runner | GitHub Runner | GitLab Runner |
| Pipeline visualization | Basic | Avancée (DAG, graph) |
| Environnements | Environments | Environments + Review Apps |

---

## 10. Bonnes pratiques

- Utiliser `rules:` plutôt que `only:/except:` (plus expressif).
- Définir des `stages:` explicites — l'ordre est important.
- Mettre en cache `node_modules/` ou `.gradle/` pour accélérer les builds.
- Utiliser le **GitLab Container Registry** intégré plutôt que Docker Hub pour les projets privés.
- Vérifier les pipelines avec `gitlab-ci-lint` avant de pusher.
- Protéger les branches `main` et `production` pour que seuls les pipelines validés puissent déployer.

---

## 11. ✅ À retenir

- `.gitlab-ci.yml` à la racine = configuration complète du pipeline.
- **Stages en séquence**, **jobs en parallèle** dans un même stage.
- Variables prédéfinies : `$CI_COMMIT_SHA`, `$CI_REGISTRY_IMAGE`... très utiles pour les tags d'images.
- `docker:dind` = Docker-in-Docker pour builder des images dans un pipeline.
- `rules:` contrôle finement quand un job s'exécute.
