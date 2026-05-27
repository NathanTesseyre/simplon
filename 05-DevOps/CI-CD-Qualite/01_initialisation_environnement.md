---
title: "CI/CD — Initialisation de l’environnement TaskBoard"
tags:
  - ci-cd
  - nodejs
  - docker
  - postgresql
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[02_setup_gitea_jenkins]]
  - [[docker_fondamentaux]]
  - [[docker_compose]]
  - [[fiche_git]]
---

# 🚀 Étape 1 — Initialisation de l’Environnement pour **TaskBoard**

## 🎯 Objectif
Mettre en place un environnement local pour **TaskBoard** (Node.js + Express) avec une base **PostgreSQL** démarrée via **Docker** et initialisée automatiquement par des scripts SQL fournis.

---

## 🧰 Prérequis multi-OS

| Outil | Windows | Debian / Ubuntu | macOS |
|:--|:--|:--|:--|
| **Node.js ≥ 18** | [Node.js LTS](https://nodejs.org) | `sudo apt install nodejs npm` (ou via nvm) | `brew install node` (ou via nvm) |
| **Git** | [Git for Windows](https://git-scm.com/download/win) | `sudo apt install git` | `brew install git` |
| **Docker** | [Docker Desktop (WSL2)](https://www.docker.com/products/docker-desktop/) | [Guide officiel Docker — Installation sur Debian](https://docs.docker.com/engine/install/debian/#install-using-the-repository) | `brew install --cask docker` |

Vérifie les versions :
```bash
node -v
npm -v
git --version
docker --version
docker compose version   # "compose" sans tiret si plugin moderne
```

> **Windows** : active **WSL2** pour de meilleures performances (Docker Desktop → Settings → WSL integration).

---

## 🪟 Notes Windows
- Utilise **Git Bash** (ou **WSL Ubuntu**) pour les commandes Unix.
- Sous **PowerShell**, remplace `cp` par `Copy-Item`.

---

## 🧩 Étape 1 — Démarrer PostgreSQL (Docker)

Le dépôt contient un `docker-compose.yml` prêt à l’emploi :

```bash
# Depuis la racine du projet (là où se trouve docker-compose.yml)
docker compose up -d
```

Ce service lance **PostgreSQL 16** et :
- expose **Postgres** sur `localhost:5000` (mappage `5000:5432`) ;
- exécute automatiquement les scripts SQL d’initialisation présents dans `db/init/*.sql` au **premier** démarrage (création de la table `tasks`, trigger `updated_at`, etc.).

Vérifie que le conteneur est prêt :
```bash
docker compose ps
docker compose logs -f db
```

---

## 🧩 Étape 2 — Préparer l’application

1. **Décompresse** l’archive `taskboard.zip` (si ce n’est pas déjà fait).  
2. Ouvre le dossier **taskboard/** dans ton éditeur.  
3. **Copie** l’exemple d’environnement :
   - Linux / macOS :
     ```bash
     cp .env.example .env
     ```
   - Windows PowerShell :
     ```powershell
     Copy-Item .env.example .env
     ```
4. **Vérifie/ajuste** `DATABASE_URL` dans `.env` :
   ```env
   PORT=3100
   DATABASE_URL=postgres://taskboard:taskboard@localhost:5000/taskboard
   ```
5. **Installe les dépendances** :
   ```bash
   npm install
   ```

---

## 🧩 Étape 3 — Lancer l’application

Démarre le serveur Node :
```bash
npm run dev
```

Sortie attendue dans la console :
```bash
> taskboard@1.0.0 dev
> nodemon src/server.js

[server] TaskBoard API running on http://localhost:3100
[server] Connected to PostgreSQL database: taskboard
```

> Si la base n’est pas encore prête, un message d’erreur de connexion peut apparaître temporairement.  
> Laisse PostgreSQL terminer son initialisation puis relance `npm run dev` si nécessaire.

L’application est servie sur :  
👉 [http://localhost:3100](http://localhost:3100)

L’interface front-end est dans `public/` (HTML + JS), et l’API sous `/api/*`.

---

## 🧪 Étape 4 — Vérifications rapides (API)

### 1️⃣ Santé de l’API
```bash
curl http://localhost:3100/api/health
```
Réponse attendue :
```json
{ "ok": true }
```

### 2️⃣ Lister les tâches
```bash
curl http://localhost:3100/api/tasks
```

### 3️⃣ Créer une tâche
```bash
curl -X POST http://localhost:3100/api/tasks   -H "Content-Type: application/json"   -d '{"title":"Ma première tâche","status":"todo"}'
```

### 4️⃣ Mettre à jour une tâche
```bash
curl -X PUT http://localhost:3100/api/tasks/1/status   -H "Content-Type: application/json"   -d '{"status":"doing"}'
```

### 5️⃣ Supprimer une tâche
```bash
curl -X DELETE http://localhost:3100/api/tasks/1
```

---

## 🧩 Organisation du projet

```
taskboard/
├── db/
│   └── init/
│       └── 0001_init.sql           # création de la table "tasks" + trigger "updated_at"
├── public/
│   ├── index.html                  # interface du tableau de tâches
│   └── app.js                      # logique front : appels API, mise à jour du DOM
├── src/
│   ├── config/
│   │   └── env.js                  # gestion des variables d’environnement
│   ├── controllers/
│   │   └── tasks.controller.js     # gère les requêtes HTTP (API REST)
│   ├── db/
│   │   ├── adapter.js              # abstraction d’accès à la base (connect/query)
│   │   └── database.js             # configuration PostgreSQL via pg.Pool
│   ├── lib/
│   │   └── asyncHandler.js         # middleware utilitaire pour les erreurs async
│   ├── repositories/
│   │   └── tasks.repo.js           # requêtes SQL CRUD vers la table "tasks"
│   ├── routes/
│   │   └── tasks.routes.js         # routes Express exposant l’API /api/tasks
│   ├── services/
│   │   └── tasks.service.js        # logique métier et validations (id, status, title)
│   ├── app.js                      # composition Express (middlewares + routes + static)
│   └── server.js                   # point d’entrée : démarre le serveur sur PORT
├── .env.example                    # modèle de configuration (.env)
├── docker-compose.yml              # service PostgreSQL + init scripts
├── package.json                    # scripts npm et dépendances (express, pg, dotenv)
└── README.md
```

---

## 🎨 Front-end (`public/`)

Le dossier `public/` contient une **interface web légère** permettant d’interagir avec l’API des tâches.

- `index.html` : structure du tableau et boutons d’action  
- `app.js` : gère les appels `fetch` vers l’API (`/api/tasks`) et met à jour le DOM  

L’application Express sert automatiquement ce dossier :
```js
app.use(express.static("public"));
```

Tu peux donc accéder à l’interface sur :  
👉 [http://localhost:3100](http://localhost:3100)

---

## 🧩 Résolution de problèmes courants

| Problème | Cause probable | Solution |
|:--|:--|:--|
| `ECONNREFUSED` ou timeout sur la DB | Postgres pas prêt / mauvais port | `docker compose ps && docker compose logs -f db` ; vérifier `DATABASE_URL` |
| `Schéma manquant` au démarrage | Scripts d’init pas encore exécutés | Laisser Postgres finir l’init ou relancer `docker compose up -d` |
| `EADDRINUSE: port 3100` | Port déjà utilisé | Modifier `PORT=3001` dans `.env` |
| `Missing required env var: DATABASE_URL` | `.env` absent/incorrect | Copier `.env.example` en `.env` et vérifier la valeur |
| Docker non détecté | WSL/Intégration désactivée | Docker Desktop → Settings → Resources → WSL integration |

**Réinitialiser la base (⚠️ supprime les données)**
```bash
docker compose down -v
docker compose up -d
```

---

## 🧹 Bonnes pratiques

- Ne versionne **jamais** le fichier `.env`.  
- Laisse Docker initialiser la base (ne crée pas les tables manuellement).  
- Utilise le même `.env` sur tous les OS pour la cohérence.  
- Utilise des scripts SQL numérotés dans `db/init/` si tu veux étendre le schéma.

---

## ✅ Résultat attendu

À la fin de cette étape :
- Le conteneur **PostgreSQL** tourne via Docker avec le schéma initial.  
- L’application Node est disponible sur **http://localhost:3100**.  
- L’API `/api/tasks` fonctionne et le front (`public/`) permet d’interagir avec elle.  
- Le projet est prêt à être versionné et intégré dans un pipeline DevOps.
