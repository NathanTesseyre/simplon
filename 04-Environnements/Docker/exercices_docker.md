# 🐳 Fiche Pratique – Exercices Docker

## 🎯 Objectif général
Découvrir Docker par étapes : exécuter des conteneurs simples, créer et persister des données, connecter des services, et orchestrer avec Docker Compose.

---

## 1️⃣ Découverte des conteneurs (guidé)
- Exécuter : `docker run hello-world`
- Lancer un conteneur qui affiche des logs :
  ```bash
  docker run --rm alpine sh -c "yes 'Hello Docker logs!' | head -n 5"
  ```
- Lancer un serveur web Nginx : `docker run -d -p 8080:80 nginx`
- Jouer avec : `docker ps`, `docker stop`, `docker rm`, `docker images`, `docker rmi`

---

## 2️⃣ Manipulation d’images (semi-guidé)
- Écrire un Dockerfile simple (Node.js ou Python).
- Builder : `docker build -t hello-custom .`
- Exécuter : `docker run --rm hello-custom`

---

## 3️⃣ Persistance des données avec Postgres
- Créer un volume : `docker volume create pgdata`
- Lancer Postgres avec le volume :  
  ```bash
  docker run -d --name db -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=trainingdb -v pgdata:/var/lib/postgresql/data postgres:16
  ```
- Se connecter : `docker exec -it db psql -U postgres -d trainingdb`
- Créer une table :
  ```sql
  CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    message TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now()
  );
  ```
- Vérifier la persistance après suppression/recréation du conteneur.

---

## 4️⃣ Réseaux et multi-services avec pgAdmin
- Créer un réseau : `docker network create training_net`
- Supprimer le container Postgres :
  ```bash
  docker container rm --force db
  ```
- Lancer Postgres avec le réseau :  
  ```bash
  docker run -d --name db -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=trainingdb -v pgdata:/var/lib/postgresql/data --network training_net postgres:16
  ```
- Lancer pgAdmin :  
  ```bash
  docker run -d --name pgadmin --network training_net -p 5050:80 -e PGADMIN_DEFAULT_EMAIL=trainer@example.com -e PGADMIN_DEFAULT_PASSWORD=secret123 dpage/pgadmin4
  ```
- Se connecter à http://localhost:5050 et ajouter le serveur `db`.

---

## 5️⃣ Docker Compose
- Créer un fichier `docker-compose.yml` :
  ```yaml
  services:
    db:
      image: postgres:16
      environment:
        POSTGRES_PASSWORD: postgres
        POSTGRES_DB: trainingdb
      volumes:
        - pgdata:/var/lib/postgresql/data

    pgadmin:
      image: dpage/pgadmin4
      environment:
        PGADMIN_DEFAULT_EMAIL: trainer@example.com
        PGADMIN_DEFAULT_PASSWORD: secret123
      ports:
        - "5050:80"
      depends_on:
        - db

  volumes:
    pgdata:
  ```
- Lancer avec `docker compose up -d`
- Vérifier : `docker compose ps`, logs et connexion pgAdmin

---

## 6️⃣ Challenge final
Créer une stack avec :
- Postgres et la table `events` :
  ```sql
  CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    message TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now()
  );
  ```
- Une API web (Node/Python) qui écrit/lit dans `events`
- pgAdmin pour le monitoring
