---
title: "Docker Compose"
tags:
  - docker
  - docker-compose
  - conteneurs
section: 04-Environnements
domaine: Docker
statut: actif
liens_connexes:
  - [[Docker]]
  - [[Dockerfile]]
  - [[commandes_docker]]
---

# 📘 Fiche : Docker Compose

---

## 1. Pourquoi Docker Compose ?

Lancer plusieurs conteneurs à la main avec `docker run` devient vite ingérable :
- commandes longues à retaper à chaque fois
- ordre de démarrage à gérer manuellement
- réseau entre conteneurs à configurer à la main

**Docker Compose** permet de décrire toute une stack dans un seul fichier `docker-compose.yml`, puis de la lancer en une commande.

👉 **Cas d'usage typique** : une application web = 1 conteneur app + 1 conteneur base de données + 1 conteneur reverse proxy.

---

## 2. Structure d'un `docker-compose.yml`

```yaml
services:        # liste des conteneurs
  nom_service:
    image: ...         # image à utiliser
    build: .           # ou construire depuis un Dockerfile
    ports:
      - "hôte:conteneur"
    environment:
      - CLE=valeur
    volumes:
      - nom_volume:/chemin/dans/conteneur
    depends_on:
      - autre_service
    networks:
      - nom_reseau

volumes:         # volumes nommés
  nom_volume:

networks:        # réseaux personnalisés
  nom_reseau:
```

---

## 3. Concepts clés

| Concept | Rôle |
|---|---|
| `services` | Chaque service = un conteneur |
| `image` | Image Docker à utiliser (ex: `postgres:16`) |
| `build` | Chemin vers un `Dockerfile` pour construire l'image |
| `ports` | Exposition de ports `hôte:conteneur` |
| `volumes` | Persistance des données ou partage de fichiers |
| `environment` | Variables d'environnement injectées dans le conteneur |
| `depends_on` | Ordre de démarrage (ne garantit pas que le service est *prêt*) |
| `networks` | Réseau dédié pour que les services communiquent entre eux |

> 💡 Sur un même réseau Compose, les services se joignent par leur **nom de service** (ex: `db`, `redis`).

---

## 4. Commandes essentielles

| Commande | Rôle |
|---|---|
| `docker compose up -d` | Démarre la stack en arrière-plan |
| `docker compose down` | Arrête et supprime les conteneurs + réseaux |
| `docker compose down -v` | Idem + supprime les volumes |
| `docker compose ps` | Liste les conteneurs de la stack |
| `docker compose logs -f` | Affiche les logs en temps réel |
| `docker compose logs -f nom_service` | Logs d'un seul service |
| `docker compose exec nom_service bash` | Ouvre un terminal dans un conteneur |
| `docker compose build` | Reconstruit les images |
| `docker compose pull` | Met à jour les images |
| `docker compose restart nom_service` | Redémarre un service sans tout couper |

> ⚠️ La commande moderne est `docker compose` (sans tiret). L'ancienne `docker-compose` est dépréciée.

---

## 5. Exemple complet : App Node.js + PostgreSQL + Nginx

```yaml
services:
  app:
    build: .
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
    depends_on:
      - db
    networks:
      - backend

  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - backend

volumes:
  postgres_data:

networks:
  backend:
```

---

## 6. Variables d'environnement avec `.env`

Plutôt que d'écrire les valeurs sensibles dans le YAML, utiliser un fichier `.env` à la racine :

```env
POSTGRES_USER=user
POSTGRES_PASSWORD=secret
POSTGRES_DB=mydb
```

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
```

👉 **Toujours ajouter `.env` au `.gitignore`** — il contient des secrets.

---

## 7. Bonnes pratiques

- Toujours utiliser des **volumes nommés** pour les données persistantes (pas de bind mount en production).
- Ne pas mettre de secrets en clair dans le YAML → utiliser `.env` ou Docker Secrets.
- Nommer explicitement les **réseaux** pour isoler les services (ex: séparer `frontend` et `backend`).
- Utiliser `depends_on` + **healthcheck** pour garantir qu'un service est prêt avant d'en démarrer un autre.
- En développement : utiliser `docker compose watch` pour le live reload.

---

## 8. ✅ À retenir

- `docker-compose.yml` décrit toute une stack en un fichier.
- Un service = un conteneur ; les services se parlent par leur nom.
- `docker compose up -d` démarre tout, `docker compose down` arrête tout.
- Les volumes nommés persistent les données entre redémarrages.
- Les secrets ne s'écrivent pas dans le YAML → fichier `.env`.
