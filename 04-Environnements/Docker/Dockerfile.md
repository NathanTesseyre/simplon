---
title: "Dockerfile"
tags:
  - docker
  - dockerfile
  - image
section: 04-Environnements
domaine: Docker
statut: actif
liens_connexes:
  - [[Docker]]
  - [[commandes_docker]]
---

# Déroulé de formation : Création et utilisation des Dockerfiles
**Public cible** : Débutants/intermédiaires en développement et administration système
**Durée** : 1 journée (3h théorie + 3h pratique)

---

## Matinée : Cours théorique (3h)

### 1. Introduction aux Dockerfiles (30 min)
- **Objectif** : Comprendre ce qu'est un Dockerfile et son utilité.
- **Contenu** :
  - Définition : Un Dockerfile est un fichier texte contenant des instructions pour construire une **image Docker**.
  - À quoi ça sert ?
    - Automatiser la création d'un environnement de développement ou de production.
    - Garantir que tous les membres d'une équipe utilisent la même configuration.
    - Isoler les dépendances (PHP, Apache, MySQL, etc.) pour éviter les conflits.

---

### 2. Structure de base d'un Dockerfile (45 min)

| Instruction | Description | Exemple |
| --- | --- | --- |
| **FROM** | Définit l'image de base (ex: PHP, Apache, Nginx). | `FROM php:8.2-apache` |
| **WORKDIR** | Définit le répertoire de travail dans le conteneur. | `WORKDIR /var/www/html` |
| **COPY** | Copie des fichiers du système hôte vers le conteneur. | `COPY . .` |
| **RUN** | Exécute une commande pendant la construction de l'image. | `RUN docker-php-ext-install pdo pdo_mysql` |
| **CMD** | Définit la commande par défaut à exécuter quand le conteneur démarre. | `CMD ["apache2-foreground"]` |
| **EXPOSE** | Indique le port sur lequel le conteneur écoute. | `EXPOSE 80` |
| **ENV** | Définit une variable d'environnement. | `ENV APACHE_DOCUMENT_ROOT=/var/www/html` |

---

### 3. Exemple simple : Application PHP avec Apache (30 min)
- **Objectif** : Découvrir un exemple concret de Dockerfile pour une application PHP.
- **Contenu** :
  ```dockerfile
  # Utilise une image PHP officielle avec Apache
  FROM php:8.2-apache

  # Définit le répertoire de travail
  WORKDIR /var/www/html

  # Installe les extensions PHP nécessaires
  RUN docker-php-ext-install pdo pdo_mysql

  # Copie les fichiers locaux dans le conteneur
  COPY . .

  # Expose le port 80 (port par défaut pour Apache)
  EXPOSE 80

  # Définit la commande par défaut pour lancer Apache
  CMD ["apache2-foreground"]
  ```

---

### 4. Bonnes pratiques (45 min)
- **Objectif** : Apprendre à écrire des Dockerfiles optimisés et sécurisés.
- **Contenu** :
  - **Minimiser les couches** : Regrouper les commandes `RUN` pour réduire la taille de l'image.
    ```dockerfile
    RUN apt-get update && apt-get install -y \
        git \
        curl
    ```
  - **Utiliser `.dockerignore`** : Exclure les fichiers inutiles (comme `.git`, `vendor/`, `.env`) pour accélérer la construction.
  - **Éviter de lancer Apache en tant que root** : Utiliser `USER` pour des raisons de sécurité.
  - **Taguer ses images** : Utiliser des tags clairs (ex: `monapp-php:v1.0`).

---

### 5. Construire et lancer une image (30 min)
- **Objectif** : Apprendre à construire une image et à lancer un conteneur.
- **Contenu** :
  1. Construire l'image :
     ```bash
     docker build -t monapp-php\:latest .
     ```
  2. Lancer un conteneur :
     ```bash
     docker run -p 8080:80 monapp-php
     ```
  3. Vérifier que l'application est accessible sur `http://localhost:8080`.

---

### 6. Ressources utiles (30 min)
- **Documentation officielle Docker** : [docs.docker.com](https://docs.docker.com)
- **Images PHP officielles sur Docker Hub** : [hub.docker.com/_/php](https://hub.docker.com/_/php)
- **Tutoriels Docker pour PHP** : [Docker et PHP](https://www.php.net/manual/fr/install.unix.docker.php)

---

## Après-midi : Travaux pratiques (3h)

### 1. Création d'un Dockerfile simple (45 min)
- **Objectif** : Créer un Dockerfile pour une application basique.
- **Étapes** :
  1. Créer un répertoire pour le projet (`mkdir mon-projet-docker`).
  2. Créer un fichier `index.php` simple :
     ```php
     <?php
     echo "Bonjour depuis Docker !";
     ?>
     ```
  3. Créer un Dockerfile :
     ```dockerfile
     FROM php:8.2-apache
     WORKDIR /var/www/html
     COPY . .
     EXPOSE 80
     CMD ["apache2-foreground"]
     ```
  4. Construire l'image :
     ```bash
     docker build -t monapp-php .
     ```

---

### 2. Construction et exécution d'une image (45 min)
- **Objectif** : Construire une image et exécuter un conteneur.
- **Étapes** :
  1. Lancer le conteneur :
     ```bash
     docker run -d -p 8080:80 --name mon-conteneur monapp-php
     ```
  2. Vérifier que le conteneur est en cours d'exécution :
     ```bash
     docker ps
     ```
  3. Accéder à l'application depuis un navigateur (`http://localhost:8080`).
  4. Arrêter et supprimer le conteneur :
     ```bash
     docker stop mon-conteneur
     docker rm mon-conteneur
     ```

---

### 3. Utilisation de `.dockerignore` (30 min)
- **Objectif** : Optimiser la construction d'une image en excluant des fichiers inutiles.
- **Étapes** :
  1. Créer un fichier `.dockerignore` :
     ```
     .git
     vendor/
     .env
     *.log
     ```
  2. Reconstruire l'image et observer la différence de taille :
     ```bash
     docker build -t monapp-php-optimise .
     ```

---

### 4. Création d'un Dockerfile pour une application Node.js (45 min)
- **Objectif** : Adapter un Dockerfile pour une application Node.js.
- **Étapes** :
  1. Créer un projet Node.js simple avec un fichier `app.js` :
     ```javascript
     const http = require('http');
     const server = http.createServer((req, res) => {
       res.end('Bonjour depuis Node.js dans Docker !');
     });
     server.listen(3000);
     ```
  2. Créer un Dockerfile :
     ```dockerfile
     FROM node:18
     WORKDIR /app
     COPY package.json .
     RUN npm install
     COPY . .
     EXPOSE 3000
     CMD ["node", "app.js"]
     ```
  3. Construire et exécuter l'image :
     ```bash
     docker build -t monapp-node .
     docker run -d -p 3000:3000 monapp-node
     ```
  4. Vérifier que l'application est accessible sur `http://localhost:3000`.

---

### 5. Utilisation de variables d'environnement (30 min)
- **Objectif** : Utiliser des variables d'environnement dans un Dockerfile.
- **Étapes** :
  1. Modifier le Dockerfile pour inclure des variables d'environnement :
     ```dockerfile
     FROM php:8.2-apache
     ENV APACHE_DOCUMENT_ROOT=/var/www/html
     WORKDIR \${APACHE_DOCUMENT_ROOT}
     COPY . .
     EXPOSE 80
     CMD ["apache2-foreground"]
     ```
  2. Reconstruire l'image et vérifier que les variables sont correctement interprétées.

---

### 6. Exercice final : Dockerfile pour une application complète (30 min)
- **Objectif** : Créer un Dockerfile pour une application complète avec une base de données.
- **Étapes** :
  1. Créer une application PHP simple qui se connecte à une base de données MySQL.
  2. Créer un Dockerfile pour l'application PHP.
  3. Créer un fichier `docker-compose.yml` pour gérer l'application et la base de données :
     ```yaml
     version: '3'
     services:
       web:
         build: .
         ports:
           - "8080:80"
         depends_on:
           - db
       db:
         image: mysql:8.0
         environment:
           MYSQL_ROOT_PASSWORD: monmotdepasse
           MYSQL_DATABASE: monapp
         volumes:
           - db_data:/var/lib/mysql
     volumes:
       db_data:
     ```
  4. Lancer l'application avec Docker Compose :
     ```bash
     docker-compose up -d
     ```
  5. Vérifier que l'application et la base de données fonctionnent correctement.
