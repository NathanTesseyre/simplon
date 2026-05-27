---
title: "Introduction à Docker"
tags:
  - docker
  - conteneurs
  - formation
section: 04-Environnements
domaine: Docker
statut: actif
liens_connexes:
  - [[dockerfile]]
  - [[commandes_docker]]
  - [[vm_vs_conteneurs]]
---

# Déroulé de formation : Introduction à Docker
**Public cible** : Débutants/intermédiaires en développement et administration système
**Durée** : 1 journée (3h théorie + 3h pratique)

---

## Matinée : Cours théorique (3h)

### 1. Introduction à Docker (30 min)
- **Objectif** : Comprendre ce qu'est Docker et ses avantages.
- **Contenu** :
  - Définition : Docker est une plateforme pour créer, déployer et exécuter des applications dans des **conteneurs**.
  - Avantages :
    - **Portabilité** : Une application conteneurisée s'exécute de la même manière sur n'importe quel système.
    - **Isolation** : Chaque conteneur est indépendant, évitant les conflits entre applications.
    - **Efficacité** : Les conteneurs partagent le noyau du système hôte, ce qui les rend plus légers que les machines virtuelles.
    - **Scalabilité** : Facile à déployer et à dupliquer.

---

### 2. Concepts clés de Docker (45 min)

| Concept | Description | Exemple/Commande |
| --- | --- | --- |
| **Image** | Modèle immuable utilisé pour créer des conteneurs. | `docker pull php:8.2-apache` |
| **Conteneur** | Instance exécutable d'une image. | `docker run -d -p 8080:80 monapp-php` |
| **Docker Hub** | Registre public d'images Docker. | [hub.docker.com](https://hub.docker.com) |
| **Volume** | Permet de persister des données ou de partager des fichiers avec le conteneur. | `docker volume create mon_volume` |
| **Réseau** | Permet aux conteneurs de communiquer entre eux ou avec l'extérieur. | `docker network create mon_reseau` |

---

### 3. Commandes Docker essentielles (1h)

| Commande | Description |
| --- | --- |
| `docker pull <image>` | Télécharge une image depuis Docker Hub. |
| `docker build -t <nom>` | Construit une image à partir d'un Dockerfile. |
| `docker run <image>` | Lance un conteneur à partir d'une image. |
| `docker ps` | Liste les conteneurs en cours d'exécution. |
| `docker ps -a` | Liste tous les conteneurs (y compris ceux arrêtés). |
| `docker stop <conteneur>` | Arrête un conteneur. |
| `docker rm <conteneur>` | Supprime un conteneur. |
| `docker images` | Liste les images locales. |
| `docker rmi <image>` | Supprime une image. |
| `docker exec -it <conteneur> bash` | Ouvre un terminal interactif dans un conteneur en cours d'exécution. |

---

### 4. Cas d'usage courants (30 min)
- **Développement local** : Isoler les dépendances d'un projet (ex: PHP, MySQL, Node.js).
- **Déploiement** : Déployer une application et ses dépendances de manière cohérente.
- **Tests** : Créer des environnements de test identiques à la production.
- **Microservices** : Exécuter chaque service d'une application dans un conteneur dédié.

---

### 5. Bonnes pratiques (30 min)
- **Nettoyer régulièrement** : Supprimer les conteneurs et images inutilisés pour libérer de l'espace.
  ```bash
  docker system prune
  ```
- **Utiliser des volumes** pour les données persistantes (ex: bases de données).
- **Documenter** les configurations et commandes utilisées dans un `README.md`.

---

### 6. Exemple : Lancer un conteneur PHP + MySQL (30 min)
1. Lancer un conteneur MySQL :
   ```bash
   docker run --name mon-mysql -e MYSQL_ROOT_PASSWORD=monmotdepasse -d mysql:8.0
   ```
2. Lancer un conteneur PHP connecté à MySQL :
   ```bash
   docker run --name monapp-php -p 8080:80 --link mon-mysql\:mysql -d monappphp
   ```

---

## Après-midi : Travaux pratiques (3h)

### 1. Installation et configuration de Docker (30 min)
- **Objectif** : Installer Docker et vérifier son fonctionnement.
- **Étapes** :
  1. Installer Docker sur sa machine (Linux, macOS, ou Windows avec WSL2).
     ```bash
     sudo apt update && sudo apt install docker.io
     ```
  2. Vérifier l'installation :
     ```bash
     docker --version
     ```
  3. Lancer un conteneur de test :
     ```bash
     docker run hello-world
     ```

---

### 2. Manipulation des images et conteneurs (45 min)
- **Objectif** : Apprendre à télécharger, lancer, et gérer des images et conteneurs.
- **Étapes** :
  1. Télécharger une image (ex: Nginx) :
     ```bash
     docker pull nginx
     ```
  2. Lancer un conteneur Nginx :
     ```bash
     docker run -d -p 8080:80 --name mon-nginx nginx
     ```
  3. Vérifier que le conteneur est en cours d'exécution :
     ```bash
     docker ps
     ```
  4. Accéder à Nginx depuis un navigateur (`http://localhost:8080`).
  5. Arrêter et supprimer le conteneur :
     ```bash
     docker stop mon-nginx
     docker rm mon-nginx
     ```

---

### 3. Utilisation des volumes (45 min)
- **Objectif** : Persister des données avec des volumes Docker.
- **Étapes** :
  1. Créer un volume :
     ```bash
     docker volume create mon_volume
     ```
  2. Lancer un conteneur MySQL avec un volume :
     ```bash
     docker run --name mon-mysql -e MYSQL_ROOT_PASSWORD=monmotdepasse -v mon_volume:/var/lib/mysql -d mysql:8.0
     ```
  3. Vérifier que les données persistent après un redémarrage du conteneur.

---

### 4. Création d'un Dockerfile (45 min)
- **Objectif** : Créer une image personnalisée à partir d'un Dockerfile.
- **Étapes** :
  1. Créer un fichier `Dockerfile` :
     ```dockerfile
     FROM ubuntu:22.04
     RUN apt update && apt install -y python3
     COPY app.py /app/
     WORKDIR /app
     CMD ["python3", "app.py"]
     ```
  2. Construire l'image :
     ```bash
     docker build -t mon-app-python .
     ```
  3. Lancer un conteneur à partir de l'image :
     ```bash
     docker run -d --name mon-app mon-app-python
     ```

---

### 5. Utilisation de Docker Compose (45 min)
- **Objectif** : Gérer plusieurs conteneurs avec Docker Compose.
- **Étapes** :
  1. Créer un fichier `docker-compose.yml` :
     ```yaml
     version: '3'
     services:
       web:
         image: nginx
         ports:
           - "8080:80"
       db:
         image: mysql:8.0
         environment:
           MYSQL_ROOT_PASSWORD: monmotdepasse
         volumes:
           - mon_volume:/var/lib/mysql
     volumes:
       mon_volume:
     ```
  2. Lancer les conteneurs :
     ```bash
     docker-compose up -d
     ```
  3. Vérifier que les conteneurs sont en cours d'exécution :
     ```bash
     docker-compose ps
     ```

---

### 6. Exercice final : Déploiement d'une application complète (30 min)
- **Objectif** : Déployer une application web avec une base de données.
- **Étapes** :
  1. Créer un `Dockerfile` pour une application Node.js ou Python.
  2. Créer un fichier `docker-compose.yml` pour l'application et une base de données (ex: MySQL ou PostgreSQL).
  3. Lancer l'application avec `docker-compose up -d`.
  4. Vérifier que l'application est accessible depuis un navigateur.
  5. Nettoyer les ressources après le test :
     ```bash
     docker-compose down
     docker system prune
     ```
