---
title: "Semantic release et versioning"
tags:
  - devops
  - release
  - semver
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[trunk-based-development]]
  - [[fiche_git]]
  - [[02_setup_gitea_jenkins]]
---

# 🏷️ Versionning, SemVer, Conventional Commits & Release Automatique

Cette fiche explique **de manière progressive et pédagogique** :
1. Pourquoi on versionne un logiciel
2. Comment fonctionnent les versions avec **SemVer**
3. Comment **Conventional Commits** permet de décrire clairement les changements
4. Comment **automatiser le versionning** avec **semantic-release**
5. Comment l'intégrer dans **Jenkins**

---

## 1) 🎯 Pourquoi versionner ?

Versionner une application signifie attribuer un **numéro unique** à chaque état livré du logiciel.

Cela permet de :
- **Communiquer clairement** l’impact d’un changement
- **Identifier précisément** la version utilisée par l’utilisateur
- **Faciliter les déploiements** et les retours en arrière (rollback)
- **Tracer l’historique du projet**
- **Structurer le support** (bugs, tickets, demandes)

> La version est un **langage** entre développeurs, produit, QA, support et clients.

---

## 2) 🔢 SemVer — Versionnement Sémantique

Nous utilisons la notation :

```
MAJOR.MINOR.PATCH
```

| Partie | Exemple | Quand la modifier | Interprétation |
|-------|---------|------------------|----------------|
| **MAJOR** | `1 → 2.0.0` | Lorsque l’on casse une compatibilité | ⚠️ Potentiellement impactant |
| **MINOR** | `1.4 → 1.5.0` | Lorsqu’on ajoute une fonctionnalité compatible | Nouveauté sans danger |
| **PATCH** | `1.4.2 → 1.4.3` | Lors d’une correction ou amélioration interne | Changement sûr |

Exemples :
- Correction d’un message d’erreur → **PATCH**
- Ajout d’un bouton dans l’interface → **MINOR**
- Changement d’un endpoint API → **MAJOR**

➡️ SemVer permet de **déduire l’impact d’une version sans lire le code**.

---

## 3) ✍️ Conventional Commits — Décrire clairement les changements

Pour que la version puisse être **calculée automatiquement**, les commits doivent être **structurés**.

Format :

```
<type>[scope]: description courte
```

Exemples :

```
feat(cart): ajout de la gestion des codes promo
fix(api): correction du code HTTP sur /login
docs(readme): ajout du guide d’installation
refactor(user): simplification de la logique de création
```

| Type | Signification | Impact SemVer |
|------|--------------|---------------|
| `feat` | Nouvelle fonctionnalité | **MINOR** |
| `fix` | Correction | **PATCH** |
| `feat!` ou Ajout du footer `BREAKING CHANGE:` | Rupture de compatibilité | **MAJOR** |
| `docs`, `test`, `chore`, `refactor` | Pas de modification fonctionnelle | Aucun impact |

➡️ Les commits deviennent **informatifs**, **cohérents**, et **exploitables par la CI**.

---

## 4) 🤖 Automatiser le versionning avec semantic-release

**Objectif :**
Ne plus jamais écrire soi-même :
- le numéro de version
- le changelog
- le tag git
- la release GitHub / GitLab

### Fonctionnement
1. semantic-release lit les commits
2. Détermine MAJOR / MINOR / PATCH
3. Met à jour `CHANGELOG.md`
4. Met à jour `package.json`
5. Crée un tag Git
6. (Optionnel) Publie la release (npm, GitHub, GitLab, Artifactory…)

---

## 5) 🛠️ Installation dans un projet Node.js

```
npm install -D semantic-release   @semantic-release/commit-analyzer   @semantic-release/release-notes-generator   @semantic-release/git
```

---

## 6) 📝 Configuration minimale `.releaserc`

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    ["@semantic-release/git", {
      "assets": ["CHANGELOG.md", "package.json"]
    }]
  ]
}
```

> Cette configuration :
> - Calcule la version automatiquement
> - Met à jour `package.json` et `CHANGELOG.md`
> - Commit et push ces fichiers sur la branche

---

## 7) 🧱 Exemple Jenkins Pipeline

```groovy
pipeline {
  agent any

  stages {
    stage('Install') {
      steps { sh 'npm ci' }
    }
    stage('Test') {
      steps { sh 'npm test' }
    }
    stage('Release') {
      when { branch 'main' }
      steps { sh 'npx semantic-release' }
    }
  }
}
```

---

## ✅ Résumé visuel

```
Conventional Commits
        ↓
semantic-release → calcule SemVer + génère changelog + crée le tag + commit auto
        ↓
CI (Jenkins) → lance la release automatiquement sur main
```

➡️ **Livraison plus fluide**  
➡️ **Historique clair**  
➡️ **Moins d’erreurs humaines**

---

## 📚 Références

- SemVer → https://semver.org/lang/fr/
- Conventional Commits → https://www.conventionalcommits.org/
- semantic-release → https://semantic-release.gitbook.io/
