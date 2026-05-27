---
title: "Docker pas à pas"
tags:
  - docker
  - pratique
  - exercice
section: 04-Environnements
domaine: Docker
statut: actif
liens_connexes:
  - [[commandes_docker]]
  - [[exercices_docker]]
---

# 🐳 Fiche pas-à-pas – Expériences Docker (commentée)

## 🎯 Objectif  
Découvrir Docker par l’expérimentation : chaque concept est testé par une **manip concrète**, avec **avant/après**, explications et observations.

---

## 1️⃣ Images vs Conteneurs

### Expérience
1) Voir les images locales :  
```bash
docker images
```
→ Liste des images disponibles en local.

2) Lancer une image non présente :  
```bash
docker run hello-world
```
→ Télécharge l’image si absente, crée un conteneur, exécute, puis s’arrête.

3) Lister les conteneurs (même stoppés) :  
```bash
docker ps -a
```

4) Vérifier que l’image reste :  
```bash
docker images
```
→ L’image `hello-world` est toujours là, même si le conteneur est supprimé.

✅ **Conclusion**  
- **Image** = modèle figé (reste tant qu’on ne l’efface pas)  
- **Conteneur** = instance jetable  

---

## 2️⃣ Conteneur éphémère et logs

### Expérience
1) Lancer un conteneur qui logge :  
```bash
docker run -d --name logger alpine sh -c "i=0; while true; do echo log $i; i=$((i+1)); sleep 1; done"
```

2) Suivre les logs :  
```bash
docker logs -f logger
```

3) Arrêter et supprimer :  
```bash
docker stop logger
docker rm logger
```

4) Vérifier :  
```bash
docker ps -a
docker images
```
→ Le conteneur a disparu, mais **l’image alpine est toujours là**.

✅ **Conclusion**  
Un conteneur est éphémère, mais l’image qui l’a créé reste.

---

## 3️⃣ Persistance avec Postgres

### Étape A — **sans volume**
```bash
docker run -d --name db1 -e POSTGRES_PASSWORD=postgres postgres:16
```
→ Base créée, mais stockée **dans le conteneur**.

Dans `psql` :
```sql
CREATE TABLE demo(id serial PRIMARY KEY, msg text);
\dt        -- liste les tables
\q         -- quitter psql
```

→ Supprimer et relancer un autre conteneur = **table disparue**.

---

### Étape B — **avec volume**
```bash
docker volume create pgdata
docker run -d --name db3 \\
  -e POSTGRES_PASSWORD=postgres \\
  -v pgdata:/var/lib/postgresql/data \\
  postgres:16
```

- Ici, `pgdata` est un **volume géré par Docker**.  
- On **ne choisit pas** vraiment où il se trouve : Docker le gère et l’isole.  
- On peut inspecter son emplacement exact (souvent `/var/lib/docker/volumes/...`), mais ce n’est pas censé être manipulé directement.

Dans `psql` :
```sql
CREATE TABLE demo(id serial PRIMARY KEY, msg text);
\dt        -- la table existe
INSERT INTO demo(msg) VALUES ('persisted!');
SELECT * FROM demo;
\q
```

→ Supprimer et relancer un conteneur avec le même volume :  
```bash
docker rm -f db3
docker run -d --name db4 \\
  -e POSTGRES_PASSWORD=postgres \\
  -v pgdata:/var/lib/postgresql/data \\
  postgres:16
docker exec -it db4 psql -U postgres -d postgres -c "SELECT * FROM demo;"
```

→ La table et les données sont toujours là.

```bash
docker volume inspect pgdata
```
→ Le champ `Mountpoint` montre **où Docker stocke physiquement les données**, mais rappelez que c’est géré par Docker, pas par l’utilisateur.

✅ **Conclusion**  
- Sans volume : données perdues à la suppression.  
- Avec volume : données persistées, **même si le conteneur disparaît**.  
- Le volume est un objet **géré par Docker**, pas un simple dossier.

---

## 4️⃣ Réseaux Docker

### Expérience
1) Créer un réseau :  
```bash
docker network create testnet
```

2) Lancer Postgres sur ce réseau :  
```bash
docker run -d --name db --network testnet -e POSTGRES_PASSWORD=postgres postgres:16
```

3) Tester la connexion **dans le même réseau** :  
```bash
docker run -it --rm --network testnet postgres:16 \\
  psql -h db -U postgres -c "SELECT 'Connexion réussie';"
```

4) Tester **hors réseau** :  
```bash
docker run -it --rm postgres:16 \\
  psql -h db -U postgres -c "SELECT 'Test';"
```
→ Échec : le client ne connaît pas `db`.

✅ **Conclusion**  
Les réseaux Docker fournissent un **DNS interne** entre les conteneurs d’un même réseau.

---

## 5️⃣ Orchestration avec Docker Compose

> 🎯 **Pourquoi Compose ?**  
Taper et retaper ces longues commandes avec `--network`, `-v`, `-e`, `-p` est **fastidieux et source d’erreurs**.  
Avec Compose : **un fichier → une commande**. La configuration est **partageable, lisible, reproductible**.

### Fichier `docker-compose.yml` minimal (commenté)
```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: postgres        # mot de passe superuser
      POSTGRES_DB: trainingdb            # base créée à l'init
    volumes:
      - pgdata:/var/lib/postgresql/data  # volume persistant géré par Docker

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: trainer@example.com
      PGADMIN_DEFAULT_PASSWORD: secret123
    ports:
      - "5050:80"    # interface pgAdmin dispo sur http://localhost:5050
    # Pas besoin de réseau déclaré : Compose crée un réseau par défaut,
    # et les services peuvent se joindre par leur nom (ici : "db").

volumes:
  pgdata:   # volume persistant pour Postgres (géré par Docker, emplacement interne)
```

### Commandes Compose
```bash
docker compose up -d      # démarrer en arrière-plan
docker compose ps         # état des services
docker compose logs db    # logs Postgres
docker compose down       # stoppe les services (conserve les volumes)
docker compose down -v    # stoppe + supprime les volumes (⚠️ données perdues)
```

---

### Équivalent en lignes de commande (sans Compose)
```bash
# Réseau
docker network create training_net

# Volume
docker volume create pgdata

# Postgres
docker run -d --name db \\
  --network training_net \\
  -e POSTGRES_PASSWORD=postgres \\
  -e POSTGRES_DB=trainingdb \\
  -v pgdata:/var/lib/postgresql/data \\
  postgres:16

# pgAdmin
docker run -d --name pgadmin \\
  --network training_net \\
  -p 5050:80 \\
  -e PGADMIN_DEFAULT_EMAIL=trainer@example.com \\
  -e PGADMIN_DEFAULT_PASSWORD=secret123 \\
  dpage/pgadmin4
```

→ **C’est exactement la même chose que Compose… mais beaucoup plus long à taper !**

---

## 🧩 Récap apprentissages

- **Images vs conteneurs** : l’image reste, le conteneur est jetable.  
- **Logs** : `docker logs -f` permet d’observer l’exécution.  
- **Volumes** :  
  - Sans volume → perte de données  
  - Avec volume → données persistées dans un espace **géré par Docker**  
  - On ne gère pas directement les fichiers, Docker s’en occupe.  
- **Réseaux** : fournissent un DNS interne entre services.  
- **Compose** : évite de retaper de longues commandes, tout est dans un seul fichier.
