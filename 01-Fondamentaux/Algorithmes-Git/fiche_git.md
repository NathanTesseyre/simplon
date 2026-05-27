---
title: "Git et gestion de versions"
tags:
  - fondamentaux
  - git
  - versionning
  - GitLab
section: 01-Fondamentaux
domaine: Git
statut: actif
liens_connexes:
  - [[trunk-based-development]]
  - [[semantic-release-and-versionning]]
---

# 📘 Fiche : Git et la gestion de versions  
*(à destination de développeurs en formation)*  

---

## 1. 🤔 Pourquoi utiliser un gestionnaire de versions ?  
Un gestionnaire de versions (VCS – *Version Control System*) permet de :  
- 📜 **Garder l’historique** → chaque modification du code est enregistrée.  
- 🔙 **Revenir en arrière** → retrouver une version précédente si nécessaire.  
- 👨‍👩‍👧‍👦 **Collaborer** → plusieurs développeurs travaillent sur le même projet sans se marcher dessus.  
- 🧪 **Expérimenter** → tester des idées dans des branches sans casser la version stable.  

---

## 2. 🐙 Qu’est-ce que Git ?  
- **Git** est le gestionnaire de versions le plus utilisé aujourd’hui.  
- Créé par **Linus Torvalds** (aussi créateur de Linux).  
- **Distribué** → chaque développeur a une copie complète du dépôt (pas besoin d’un serveur central en permanence).  

👉 Outils autour de Git : **GitHub, GitLab, Bitbucket** → plateformes pour héberger du code et collaborer.  

---

## 3. 🔑 Les concepts essentiels de Git  

### 📂 Dépôt (repository)  
Le projet versionné → contient le code + l’historique des changements.  

### 💾 Commit  
Un “instantané” du code à un moment donné.  
👉 Comme une **photo** de ton projet.  

### 🌿 Branche  
Une ligne de développement indépendante.  
👉 La branche `main` ou `master` = la version stable du projet.  

### 🔀 Merge  
Fusionner les changements d’une branche dans une autre.  

### 📌 Tag  
Un repère important dans l’historique (souvent une **version**).  

---

## 4. ⚙️ Cycle de base avec Git  

1. **Cloner** un dépôt distant  
```bash
git clone https://github.com/utilisateur/projet.git
```  

2. **Créer une branche** pour travailler  
```bash
git checkout -b nouvelle-fonction
```  

3. **Ajouter et valider des changements**  
```bash
git add fichier.py
git commit -m "Ajout de la nouvelle fonction"
```  

4. **Envoyer la branche sur le dépôt distant**  
```bash
git push origin nouvelle-fonction
```  

5. **Fusionner via une Pull Request / Merge Request** sur GitHub/GitLab.  

---

## 5. 🔧 Commandes Git essentielles  

| Commande | Rôle |
|----------|------|
| `git init` | Créer un nouveau dépôt |
| `git clone <url>` | Cloner un dépôt existant |
| `git status` | Voir les fichiers modifiés |
| `git add <fichier>` | Ajouter un fichier au prochain commit |
| `git commit -m "message"` | Créer un commit |
| `git log` | Voir l’historique |
| `git branch` | Lister les branches |
| `git checkout -b ma-branche` | Créer et basculer sur une branche |
| `git merge ma-branche` | Fusionner une branche |
| `git pull` | Récupérer les changements distants |
| `git push` | Envoyer ses changements au dépôt distant |

---

## 6. 👨‍👩‍👧‍👦 Workflow en équipe (GitHub / GitLab)  

1. Un développeur **crée une branche** depuis `main`.  
2. Il fait des **commits** réguliers.  
3. Il **pousse** sa branche sur GitHub/GitLab.  
4. Il ouvre une **Pull Request** (ou Merge Request).  
5. Les autres développeurs **relisent et commentent** le code.  
6. Une fois validé → la branche est **fusionnée** dans `main`.  

---

## 7. 🌟 Bonnes pratiques Git  
- Faire des commits **petits et clairs** (un commit = une idée).  
- Écrire des **messages explicites** → “Fix bug X” est mieux que “changement”.  
- Toujours **travailler dans une branche**, jamais directement sur `main`.  
- Mettre à jour sa branche régulièrement (`git pull`) pour éviter les conflits.  
- Utiliser des **tags** pour marquer les versions stables.  

---

## 8. ⚠️ Messages d’erreurs fréquents et solutions  

### 🔴 `fatal: not a git repository`  
➡️ Git ne trouve pas de dépôt.  
👉 Vérifier que vous êtes bien dans un dossier cloné ou initialisé avec `git init`.  

---

### 🔴 `error: failed to push some refs`  
➡️ Quelqu’un a modifié la branche à distance.  
👉 Solution :  
```bash
git pull origin main
git push origin main
```  
(souvent avec un merge ou un rebase à gérer).  

---

### 🔴 `merge conflict`  
➡️ Git ne peut pas fusionner automatiquement car deux personnes ont modifié la même partie du fichier.  
👉 Solution :  
1. Ouvrir le fichier et choisir quelles lignes garder.  
2. Supprimer les marqueurs `<<<<<<<`, `=======`, `>>>>>>>`.  
3. Valider avec un commit :  
```bash
git add fichier
git commit
```  

---

### 🔴 `detached HEAD`  
➡️ Vous êtes sur un commit “isolé” et non sur une branche.  
👉 Solution : créer une branche depuis là :  
```bash
git checkout -b ma-nouvelle-branche
```  

---

### 🔴 `permission denied (publickey)`  
➡️ Problème d’accès SSH à GitHub/GitLab.  
👉 Vérifier que votre clé SSH est générée et ajoutée à votre compte.  

---

## 9. 🔐 Utiliser un **Personal Access Token** (PAT) avec GitLab  

Lorsque votre compte GitLab est lié à un **SSO (Google, GitHub, etc.)**, vous n’avez pas de mot de passe classique.  
👉 Dans ce cas, pour utiliser `git clone`, `git push` ou `git pull` en HTTPS, il faut utiliser un **Personal Access Token** comme mot de passe.  

### Étapes pour générer un PAT  
1. Aller dans **GitLab → Paramètres → Access Tokens**.  
2. Donner un nom au token (ex : *git-formation*).  
3. Choisir une date d’expiration (éviter “jamais” pour la sécurité).  
4. Sélectionner les permissions nécessaires (souvent : `read_repository` et `write_repository`).  
5. Copier le token généré (il ne sera plus visible après).  

---

### Utiliser le PAT avec Git  
Lorsque Git demande un **username/password** :  
- **Username** → votre identifiant GitLab (ou votre email).  
- **Password** → le **Personal Access Token**.  

Exemple :  
```bash
git clone https://gitlab.com/mon-groupe/mon-projet.git
Username: mon.email@exemple.com
Password: <votre-token-ici>
```  

👉 Vous pouvez aussi enregistrer le token pour éviter de le ressaisir :  
```bash
git config --global credential.helper store
```

---

### Alternative : SSH  
Pour éviter de gérer des PAT, vous pouvez configurer une **clé SSH** et l’associer à votre compte GitLab.  
- Générer une clé :  
```bash
ssh-keygen -t ed25519 -C "mon.email@exemple.com"
```  
- Ajouter la clé publique (`~/.ssh/id_ed25519.pub`) dans **GitLab → SSH Keys**.  
- Cloner le dépôt avec l’URL SSH :  
```bash
git clone git@gitlab.com:mon-groupe/mon-projet.git
```  

---

## 10. ✅ À retenir  
- Git est un **outil incontournable** pour tout développeur moderne.  
- Concepts clés : **dépôt, commit, branche, merge**.  
- Les erreurs Git sont fréquentes au début, mais ont **toujours une solution simple**.  
- Avec GitLab + SSO → utiliser un **PAT** comme mot de passe HTTPS, ou configurer une **clé SSH**.  
- En équipe → on travaille par **branches** + **pull requests**.  

---

👉 Avec cette fiche, les étudiants savent **utiliser Git, comprendre ses concepts, résoudre les erreurs fréquentes, et gérer l’authentification moderne avec GitLab**.  
