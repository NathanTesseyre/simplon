---
title: "Container Registry"
tags:
  - docker
  - registry
  - ci-cd
  - conteneurs
section: 04-Environnements
domaine: Docker
statut: actif
liens_connexes:
  - [[Docker]]
  - [[Dockerfile]]
  - [[docker_compose]]
  - [[github_actions]]
  - [[gitlab_ci]]
  - [[kubernetes_fondamentaux]]
  - [[cloud_aws_fondamentaux]]
---

# 📘 Fiche : Container Registry

---

## 1. Pourquoi un container registry ?

Un **container registry** est un serveur qui stocke et distribue des images Docker. Sans registry, il est impossible de :
- partager une image entre développeurs,
- déployer une image sur un serveur distant,
- versionner les images dans un pipeline CI/CD.

```
CI/CD pipeline :
  Code → Build image → Push vers registry → Pull depuis registry → Déploiement
```

---

## 2. Registries disponibles

| Registry | Type | Usage |
|---|---|---|
| **Docker Hub** | Public / privé | Standard, images officielles, gratuit avec limites |
| **GitHub Container Registry (ghcr.io)** | Public / privé | Intégré à GitHub, lié aux dépôts |
| **GitLab Container Registry** | Public / privé | Intégré à GitLab CI, par projet |
| **AWS ECR** | Privé | Registry managé AWS, intégré à ECS/EKS |
| **Azure Container Registry (ACR)** | Privé | Registry managé Azure |
| **Harbor** | Auto-hébergé | Open source, scan de vulnérabilités intégré |

---

## 3. Commandes de base

### Login
```bash
# Docker Hub
docker login

# GitHub Container Registry
docker login ghcr.io -u USERNAME -p TOKEN

# GitLab Registry
docker login registry.gitlab.com -u USERNAME -p TOKEN

# AWS ECR
aws ecr get-login-password --region eu-west-3 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.eu-west-3.amazonaws.com
```

### Tag et push
```bash
# Tagger une image locale pour un registry distant
docker tag monapp:latest monuser/monapp:1.0.0
docker tag monapp:latest monuser/monapp:latest

# Pousser vers Docker Hub
docker push monuser/monapp:1.0.0
docker push monuser/monapp:latest

# Pousser vers GitHub Container Registry
docker tag monapp:latest ghcr.io/monuser/monapp:1.0.0
docker push ghcr.io/monuser/monapp:1.0.0

# Pousser vers GitLab Registry
docker tag monapp:latest registry.gitlab.com/groupe/projet/monapp:1.0.0
docker push registry.gitlab.com/groupe/projet/monapp:1.0.0
```

### Pull
```bash
docker pull monuser/monapp:1.0.0
docker pull ghcr.io/monuser/monapp:latest
```

---

## 4. Convention de tags

Un bon système de tags permet de savoir précisément quelle version est déployée.

```bash
# Tag par SHA de commit (traçabilité exacte)
docker tag monapp:latest monuser/monapp:abc1234

# Tag par version sémantique
docker tag monapp:latest monuser/monapp:1.2.3
docker tag monapp:latest monuser/monapp:1.2
docker tag monapp:latest monuser/monapp:1

# Tag par branche (environnements)
docker tag monapp:latest monuser/monapp:main
docker tag monapp:latest monuser/monapp:staging
```

> ⚠️ `latest` ne devrait jamais être utilisé en production — il ne garantit pas une version précise.

---

## 5. Intégration dans un pipeline CI/CD

### GitHub Actions
```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

- name: Build et push
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: |
      monuser/monapp:latest
      monuser/monapp:${{ github.sha }}
```

### GitLab CI
```yaml
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
```

---

## 6. AWS ECR — Créer un registry et pousser

```bash
# Créer un repository ECR
aws ecr create-repository \
  --repository-name monapp \
  --region eu-west-3

# Récupérer l'URL du registry
aws ecr describe-repositories --region eu-west-3

# Login, tag, push
aws ecr get-login-password --region eu-west-3 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.eu-west-3.amazonaws.com

docker tag monapp:latest 123456789.dkr.ecr.eu-west-3.amazonaws.com/monapp:latest
docker push 123456789.dkr.ecr.eu-west-3.amazonaws.com/monapp:latest
```

---

## 7. Utiliser un registry privé avec Kubernetes

Kubernetes doit s'authentifier pour puller des images depuis un registry privé.

```bash
# Créer un secret d'authentification
kubectl create secret docker-registry regcred \
  --docker-server=ghcr.io \
  --docker-username=monuser \
  --docker-password=MON_TOKEN \
  --docker-email=mon@email.com
```

```yaml
# Dans le Deployment
spec:
  imagePullSecrets:
    - name: regcred
  containers:
    - name: app
      image: ghcr.io/monuser/monapp:1.0.0
```

---

## 8. Bonnes pratiques

- Ne jamais utiliser `latest` en production — toujours un tag précis (SHA ou version).
- Stocker les credentials de registry dans les **secrets CI/CD**, jamais dans le code.
- Nettoyer régulièrement les vieilles images pour limiter les coûts de stockage.
- Activer le **scan de vulnérabilités** sur les images (Docker Hub, ECR, Harbor).
- Pour les projets GitLab : utiliser le **GitLab Container Registry** intégré — pas besoin de credentials supplémentaires en CI.

---

## 9. ✅ À retenir

- Un registry stocke et distribue les images Docker entre le CI et les cibles de déploiement.
- Workflow : `docker build` → `docker tag` → `docker push` (CI) → `docker pull` (déploiement).
- Ne jamais déployer avec le tag `latest` — utiliser le SHA du commit ou une version sémantique.
- Credentials = secrets CI/CD, jamais dans le code.
- Options principales : **Docker Hub** (public), **ghcr.io** (GitHub), **GitLab Registry** (GitLab), **ECR** (AWS).
