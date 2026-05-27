---
title: "GitHub Actions — CI/CD"
tags:
  - github-actions
  - ci-cd
  - automatisation
  - devops
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[gitlab_ci]]
  - [[trunk_based_development]]
  - [[container_registry]]
  - [[tests_unitaires]]
  - [[fiche_git]]
---

# 📘 Fiche : GitHub Actions

---

## 1. Qu'est-ce que GitHub Actions ?

GitHub Actions est la plateforme CI/CD intégrée à GitHub. Elle permet d'automatiser des workflows (tests, build, déploiement) directement depuis le dépôt, sans serveur CI externe.

| Jenkins / Gitea | GitHub Actions |
|---|---|
| Serveur à installer et maintenir | Hébergé par GitHub, zéro infra |
| Config dans un Jenkinsfile | Config en YAML dans `.github/workflows/` |
| Plugins à installer | Actions communautaires sur le Marketplace |

---

## 2. Concepts clés

| Concept | Définition |
|---|---|
| **Workflow** | Fichier YAML décrivant une automatisation |
| **Trigger (on)** | Événement qui déclenche le workflow (push, PR, cron...) |
| **Job** | Ensemble d'étapes s'exécutant sur un même runner |
| **Step** | Commande ou action unitaire dans un job |
| **Action** | Bloc réutilisable publié sur le Marketplace |
| **Runner** | Machine qui exécute les jobs (GitHub-hosted ou self-hosted) |
| **Artifact** | Fichier produit par un job et stocké temporairement |

---

## 3. Structure d'un workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:                          # triggers
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:                      # nom du job
    runs-on: ubuntu-latest   # runner

    steps:
      - name: Checkout du code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Installer les dépendances
        run: npm ci

      - name: Lancer les tests
        run: npm test
```

---

## 4. Triggers courants

```yaml
on:
  push:
    branches: [main, develop]
    paths: ['src/**', 'tests/**']   # seulement si ces fichiers changent

  pull_request:
    branches: [main]

  schedule:
    - cron: '0 2 * * *'            # tous les jours à 2h

  workflow_dispatch:                # déclenchement manuel depuis l'UI GitHub

  release:
    types: [published]             # à chaque nouvelle release
```

---

## 5. Variables et secrets

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: production          # variable d'environnement du job

    steps:
      - name: Déployer
        run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.API_KEY }}           # secret GitHub
          DATABASE_URL: ${{ vars.DATABASE_URL }}    # variable (non-secrète)
```

> Les secrets se configurent dans **Settings → Secrets and variables → Actions** du dépôt. Ils ne sont jamais affichés dans les logs.

---

## 6. Exemple complet — CI/CD Node.js + Docker

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm test

  build-and-push:
    needs: test                     # s'exécute seulement si test réussit
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'   # seulement sur main

    steps:
      - uses: actions/checkout@v4

      - name: Login Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build et push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: monuser/monapp:latest,monuser/monapp:${{ github.sha }}
```

---

## 7. Jobs parallèles et matrice

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]    # exécute le job sur 3 versions Node

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci && npm test
```

---

## 8. Cache et artifacts

```yaml
steps:
  # Cache des dépendances npm
  - uses: actions/cache@v4
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

  # Sauvegarder un artifact (rapport de tests, binaire...)
  - uses: actions/upload-artifact@v4
    with:
      name: rapport-tests
      path: coverage/
      retention-days: 7

  # Télécharger un artifact dans un autre job
  - uses: actions/download-artifact@v4
    with:
      name: rapport-tests
```

---

## 9. Bonnes pratiques

- Épingler les actions par SHA (`uses: actions/checkout@11bd719` ) en production pour éviter les supply chain attacks.
- Utiliser `npm ci` plutôt que `npm install` (reproductible).
- Ne jamais afficher de secret avec `echo` — GitHub les masque mais c'est une mauvaise habitude.
- Limiter les permissions des tokens avec `permissions:` au niveau du workflow.
- Séparer les jobs `test` et `deploy` avec `needs:` — ne jamais déployer sans tests verts.

---

## 10. ✅ À retenir

- Les workflows sont des fichiers YAML dans `.github/workflows/`.
- **Trigger** déclenche → **Job** s'exécute sur un **runner** → **Steps** s'enchaînent.
- Les **secrets** se configurent dans GitHub Settings, jamais dans le code.
- `needs:` permet de chaîner les jobs (test avant deploy).
- Le Marketplace propose des milliers d'**actions** réutilisables.
