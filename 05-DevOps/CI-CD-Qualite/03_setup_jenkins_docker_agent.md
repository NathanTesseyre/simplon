---
title: "CI/CD — Jenkins avec agent Docker"
tags:
  - jenkins
  - docker
  - ci-cd
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[02_setup_gitea_jenkins]]
  - [[docker_fondamentaux]]
  - [[docker_compose]]
  - [[sonarqube-jenkins-node-doc]]
---

# 🧩 Jenkins local avec agent Docker

Ce guide explique comment installer **Jenkins dans Docker** et lui connecter un **agent Docker** (pour exécuter des builds, tests, ou des commandes `docker build`, etc.).

---

## ⚙️ 1. Prérequis

- Docker installé sur la machine (Linux, macOS ou Windows WSL2)
- Port **8080** libre (interface Jenkins)
- Port **50000** libre (communication agent ↔ contrôleur)
- Aucune installation Jenkins préalable n’est nécessaire

---

## 🚀 2. Lancer le contrôleur Jenkins

Crée un fichier `docker-compose.yml` à la racine du projet :

```yaml
services:
  jenkins:
    image: jenkins/jenkins:lts-jdk17
    container_name: jenkins
    user: root # pour accéder au socket Docker
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped

volumes:
  jenkins_home:
```

Démarre Jenkins :

```bash
docker compose up -d
```

🖥️ Accède ensuite à Jenkins :  
➡️ http://localhost:8080  

Récupère le mot de passe initial :
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

---

## 🧱 3. Installer les plugins nécessaires

Depuis l’interface Jenkins :
1. Menu **Administrer Jenkins → Plugins → Plugins Disponibles**
2. Installe :
   - **Docker plugin**
   - **Docker Pipeline**

---

## 🐳 4. Créer le Cloud “Docker” dans Jenkins

Aller dans :
> **Administrer Jenkins → Clouds → Nouveau cloud → Type Docker**

Configurer Docker Cloud Details:

| Champ | Valeur |
|-------|--------|
| **Docker Host URI** | `unix:///var/run/docker.sock` |
| **Enabled** | ✅ |

Puis **Docker Agent templates → Add Docker Template**

### Template d’agent :
| Champ | Valeur |
|--------|---------|
| **Labels** | `docker-agent` |
| **Enabled** | ✅ |
| **Docker Image** | `jenkins/jnlp-agent-docker:latest` |
| **Container settings → User** | `root` |
| **Container settings → Mounts** | `type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock` |
| **Remote File System Root** | `/home/jenkins/agent` |
| **Connect method** | *Attach Docker container* |
| **Pull strategy** | *Pull once and update latest* |

> 💡 Cette image contient déjà `docker`, `git`, `curl`, etc.

Sauvegarde ensuite ta configuration.

---

## 🧩 5. Tester le fonctionnement

Crée un **nouveau pipeline** Jenkins et colle ce Jenkinsfile minimal :

```groovy
pipeline {
  agent { label 'docker-agent' }

  stages {
    stage('Test Docker') {
      steps {
        sh 'docker version'
        sh 'docker ps'
      }
    }
  }
}
```

🔧 Le build doit afficher la version Docker et la liste des containers de ton hôte (grâce au socket partagé).

---

## 🧰 6. Exemple avancé : builder une image Docker dans un job

```groovy
pipeline {
  agent { label 'docker-agent' }

  stages {
    stage('Build Docker Image') {
      steps {
        script {
          sh '''
            docker build -t myapp:test .
            docker run --rm myapp:test echo "✅ Build OK"
          '''
        }
      }
    }
  }
}
```

> 💡 Tu peux également utiliser `docker.image(...).inside { ... }` pour exécuter des étapes dans des containers éphémères.

---

## 🧱 7. Structure finale

```
.
├── docker-compose.yml
├── Jenkinsfile
└── README.md
```

---

## 🧩 8. Images officielles utiles

| Image | Description |
|--------|-------------|
| `jenkins/jenkins:lts-jdk17` | Contrôleur Jenkins |
| `jenkins/jnlp-agent-docker:latest` | Agent avec Docker CLI |
| `jenkins/inbound-agent:latest-jdk17` | Base d’agent minimal sans Docker |
| `jenkins/agent-dind` | Agent *Docker-in-Docker* (plus isolé, nécessite `privileged: true`) |

---

## 🧹 9. Nettoyage

Pour tout arrêter et supprimer :
```bash
docker compose down -v
```

---

## ✅ Résumé

| Élément | Technologie |
|----------|--------------|
| Contrôleur Jenkins | `jenkins/jenkins:lts-jdk17` |
| Agent Docker | `jenkins/jnlp-agent-docker:latest` |
| Communication | JNLP sur port 50000 |
| Build Docker | via `/var/run/docker.sock` monté |
| User | root |

---

🎉 **Ton Jenkins local est maintenant prêt** à exécuter des builds dans des containers Docker, en toute simplicité.
