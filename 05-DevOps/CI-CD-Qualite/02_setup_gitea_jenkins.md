# 🚀 Intégration locale Gitea ↔ Jenkins (via Docker)
*(Multibranch Pipeline déclenché automatiquement à chaque `git push`)*

---

## 🧩 1. Lancer Gitea et Jenkins

### `docker-compose.yml`
```yaml
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    ports:
      - "3000:3000"   # Web Gitea
    volumes:
      - gitea:/data
    environment:
      # Autorise les webhooks vers n'importe quel hôte (OK pour dev local)
      - GITEA__webhook__ALLOWED_HOST_LIST=*

  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    user: root
    ports:
      - "8080:8080"   # Web Jenkins
    volumes:
      - jenkins:/var/jenkins_home

volumes:
  gitea:
  jenkins:
```

### Démarrage
```bash
docker compose up -d
```

**Accès :**
- Gitea → [http://localhost:3000](http://localhost:3000)
- Jenkins → [http://localhost:8080](http://localhost:8080)

---

## 🔐 2. Récupérer le mot de passe admin Jenkins
```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

➡️ Copie la clé affichée et colle-la sur [http://localhost:8080](http://localhost:8080)  
➡️ Installe les plugins suggérés puis crée ton utilisateur admin.

---

## 👤 3. Créer un utilisateur admin Gitea
1. Ouvre [http://localhost:3000](http://localhost:3000)  
2. Suis l’assistant d’installation (SQLite par défaut).  
3. Crée un compte admin (ex. `admin / adminpass`).  
4. Connecte-toi avec ce compte.

---

## 🔑 4. Créer un token personnel Gitea

**Gitea → Configuration → Applications → Générer un nouveau jeton**

- **Nom du jeton** : `jenkins`
- **Permissions à cocher** :
  - `user → Lecture`
  - `organization → Lecture`
  - `repository → Lecture et écriture`
  - `admin → Lecture et écriture`
- Clique **Générer un jeton**
- Copie le token (il ne sera plus affiché ensuite)

---

## 🔌 5. Installer le plugin Jenkins

**Manage Jenkins → Plugins → Available**

Installe uniquement :
- `Gitea`

Puis redémarre Jenkins.

---

## 🧷 6. Déclarer le Gitea Server dans Jenkins

1. Va dans **Manage Jenkins → Configure System → Gitea Servers → Add Gitea Server**  
2. **Server URL** : `http://gitea:3000`  
3. **Credentials** :
   - **Add → Jenkins**
   - **Kind** : `Gitea Personal Access Token`
   - **Personal Access Token** : colle le token Gitea généré
   - **ID** : `gitea-admin-token`
   - **Description** : “Token admin Gitea local” (optionnel)
   - **Save**
4. Reviens sur la configuration du serveur :
   - Sélectionne le credential `gitea-admin-token`
   - Coche **Manage hooks**
5. **Save**

---

## 🧱 7. Créer un dépôt Gitea
1. Connecte-toi à ton compte **admin** sur [http://localhost:3000](http://localhost:3000).  
2. Clique sur **+ → Nouveau Dépôt**.  
3. Renseigne :
   - **Nom du Dépôt** : `taskboard`  
   - **Visibilité** : Public  
4. Clique sur **Créer le Dépôt**.  

---

## 💻 8. Cloner le dépôt localement

Depuis ton poste de travail, dans le dossier **taskboard** :  
```bash
git init
git add .
git commit -m "Initial commit - TaskBoard"
git branch -M main
git remote add origin http://localhost:3000/admin/taskboard.git
git push -u origin main
```

Vérifie ensuite dans Gitea :  
👉 [http://localhost:3000/admin/taskboard](http://localhost:3000/admin/taskboard)

Le dépôt doit contenir tous les fichiers de l’application et la branche `main`.

---

## 🧱 9. Créer un Pipeline multibranches dans Jenkins
1. **New Item → Multibranch Pipeline**  
2. **Name** : `taskboard-pipeline`  
3. **Branch Sources → Add source → Gitea**
   - **Server** : ton serveur Gitea
   - **Credentials** : `gitea-admin-token`
   - **Owner** : `admin`
   - **Repository** : `taskboard`
   - Coche *Discover branches* et *Discover pull requests*
4. **Build Configuration** : `by Jenkinsfile`
5. **Save** → **Scan Multibranch Pipeline Now**

---

## 🔔 10. Créer le webhook Gitea → Jenkins
Dans Gitea → **Configuration → Webhooks → Ajout un Webhook → Gitea**
- **URL cible** : `http://jenkins:8080/gitea-webhook/post`
- **Events** : coche **Tous les événements**
- **Active** : Oui → **Ajouter Webhook**

---

## 🧩 11. Ajouter un Jenkinsfile

Crée un fichier nommé **`Jenkinsfile`** à la racine du dépôt cloné, avec le contenu suivant :

```groovy
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo "Building branch ${env.BRANCH_NAME}"
      }
    }
    stage('Test') {
      steps {
        echo "Running tests on ${env.BRANCH_NAME}"
      }
    }
  }
}
```

### 🧠 Explication rapide
Ce fichier définit un **pipeline Jenkins** simple, composé de deux étapes :
- **Build** : affiche le nom de la branche actuellement construite.  
- **Test** : simule une étape de tests.  

Il sert à vérifier que la connexion **Gitea ↔ Jenkins** et le déclenchement du **webhook** fonctionnent correctement.

---

## 🚀 12. Test du webhook
```bash
git add Jenkinsfile
git commit -m "Add Jenkinsfile"
git push
```

➡️ Ce push enverra le **webhook Gitea** vers Jenkins.  
Cependant, le pipeline multibranches **ne se lancera pas automatiquement** tant qu’un premier **scan manuel** n’a pas été effectué.

### 🔄 Lancer le scan manuellement
1. Dans Jenkins, ouvre ton projet multibranches.  
2. Clique sur **“Scan Multibranch Pipeline Now”** dans le menu gauche.  
3. Jenkins détecte alors les branches contenant un `Jenkinsfile` et lance le premier build.  

> 🧠 Une fois le premier scan effectué, les prochains **push** déclencheront automatiquement les builds via le webhook.

---

## ✅ 13. Résultat final
- Tu pousses ton code dans Gitea  
- Gitea envoie un webhook à Jenkins  
- Jenkins détecte la branche et exécute automatiquement le pipeline correspondant 🎯  
- Le build s’affiche dans l’historique avec le message de ton dernier commit (`Add Jenkinsfile`).