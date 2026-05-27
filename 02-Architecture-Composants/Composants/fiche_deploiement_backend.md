---
title: "Déploiement backend selon la technologie"
tags:
  - backend
  - déploiement
  - docker
  - nginx
  - CI/CD
  - systemd
  - Node.js
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche_backend]]
  - [[fiche_deploiement_frontend]]
  - [[nginx_guide]]
  - [[docker_fondamentaux]]
  - [[services_demarrage]]
---

# 📘 Déploiement backend selon la technologie

## Vue d'ensemble des approches

```
Code source backend
        │
        ▼ Build (si nécessaire)
  Artefact déployable
  (binaire, .jar, bundle JS...)
        │
        ├── Processus sur VM/VPS (+ systemd + Nginx reverse proxy)
        ├── Conteneur Docker (local ou registre)
        ├── Plateforme managée (Heroku, Railway, Render)
        └── Serverless (AWS Lambda, Google Cloud Functions)
```

Le choix dépend de la taille du projet, des contraintes de coût, d'équipe et de scalabilité.

## 1. Node.js / Express sur VPS (déploiement classique)

Le cas le plus courant pour commencer : une API Node.js sur un VPS Ubuntu, derrière Nginx.

**Structure cible sur le serveur :**
```
/opt/mon-api/
├── src/
├── package.json
├── .env              ← jamais dans git
└── node_modules/
```

**Déploiement manuel :**
```bash
# Sur le serveur
git clone https://github.com/monorg/mon-api /opt/mon-api
cd /opt/mon-api
npm ci --production   # install sans devDependencies
cp .env.example .env
nano .env             # remplir les variables
```

**Service systemd** pour garder l'API en vie :
```ini
# /etc/systemd/system/mon-api.service
[Unit]
Description=Mon API Node.js
After=network.target postgresql.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/mon-api
ExecStart=/usr/bin/node src/server.js
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
EnvironmentFile=/opt/mon-api/.env

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mon-api
sudo systemctl status mon-api
journalctl -u mon-api -f
```

**Nginx comme reverse proxy :**
```nginx
# /etc/nginx/sites-available/mon-api
server {
    listen 80;
    server_name api.mondomaine.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/mon-api /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

**HTTPS avec Let's Encrypt (Certbot) :**
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d api.mondomaine.com
# Certbot modifie nginx.conf automatiquement et renouvelle le certificat
```

## 2. Node.js conteneurisé (Docker)

**Dockerfile production-ready :**
```dockerfile
# Étape 1 : dépendances
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Étape 2 : application finale
FROM node:20-alpine
WORKDIR /app
# Créer un utilisateur non-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=deps /app/node_modules ./node_modules
COPY src ./src
COPY package.json .
# Changer le propriétaire
RUN chown -R appuser:appgroup /app
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "src/server.js"]
```

**Variables d'environnement (ne jamais les mettre dans l'image) :**
```bash
# Passer les variables au run
docker run -d \
  --name mon-api \
  --env-file .env \
  -p 3000:3000 \
  mon-api:latest

# Ou avec un fichier compose
```

**docker-compose.yml complet (API + BDD + cache) :**
```yaml
version: "3.9"

services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://user:${DB_PASSWORD}@db:5432/app
      REDIS_URL: redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./db/init:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  cache:
    image: redis:7-alpine
    restart: unless-stopped

volumes:
  pgdata:
```

```bash
docker-compose up -d
docker-compose logs -f api
docker-compose ps
docker-compose down   # arrêter (les volumes persistent)
docker-compose down -v  # arrêter et supprimer les volumes (perd les données)
```

## 3. Pipeline CI/CD complet (GitHub Actions + Docker)

```yaml
# .github/workflows/deploy.yml
name: CI/CD Backend

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: monorg/mon-api

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports: ["5432:5432"]
        options: --health-cmd pg_isready
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm test
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/test

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ env.IMAGE_NAME }}:${{ github.sha }},ghcr.io/${{ env.IMAGE_NAME }}:latest

      - name: Deploy on server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker-compose pull api
            docker-compose up -d api
            docker image prune -f
```

## 4. Migrations de base de données

Problème classique : comment synchroniser le schéma de la base avec le code déployé ?

**Règle d'or :** les migrations doivent être compatibles avec la version N-1 du code (déploiement sans downtime).

**Avec Node.js et un outil de migration (ex: node-pg-migrate, Knex, Prisma) :**
```bash
# Dans le pipeline CI, avant de démarrer la nouvelle version
npx prisma migrate deploy     # ou
npx knex migrate:latest       # ou
node-pg-migrate up

# Dans docker-compose, via un service one-shot
services:
  migrate:
    image: mon-api:latest
    command: node migrate.js
    depends_on:
      db:
        condition: service_healthy
    restart: on-failure

  api:
    image: mon-api:latest
    depends_on:
      - migrate
```

## 5. Gestion des secrets

**Ne jamais mettre de secrets dans le code ou dans les images Docker.**

Bonnes pratiques :
```bash
# .env (local uniquement, dans .gitignore)
DATABASE_URL=postgres://user:secret@localhost/mydb
JWT_SECRET=un-secret-tres-long-et-aleatoire
SMTP_PASSWORD=motdepasse

# En production : utiliser les secrets du CI/CD
# GitHub Actions → Settings → Secrets → Actions
# Accès dans le workflow : ${{ secrets.JWT_SECRET }}

# Sur le serveur : fichier .env avec permissions restrictives
chmod 600 /opt/mon-api/.env
chown www-data:www-data /opt/mon-api/.env

# Alternativement : outils dédiés
# AWS Secrets Manager, HashiCorp Vault, Doppler
```

## 6. Déploiement sans interruption (zero-downtime)

Sur un seul serveur, avec systemd et Nginx :
```bash
# Tirer le nouveau code
git pull origin main
npm ci --production

# Recharger le processus sans couper les connexions existantes
sudo systemctl reload mon-api   # si l'appli supporte SIGHUP pour reload
# ou
sudo systemctl restart mon-api  # redémarrage rapide (< 1s avec PM2 ou systemd)
```

Avec Docker (rolling update) :
```bash
# Démarrer le nouveau conteneur avant d'arrêter l'ancien
docker run -d --name mon-api-new -p 3001:3000 mon-api:new
# Vérifier qu'il est sain
curl http://localhost:3001/health
# Basculer Nginx
sed -i "s/3000/3001/" /etc/nginx/sites-available/mon-api
nginx -s reload
# Arrêter l'ancien
docker stop mon-api-old && docker rm mon-api-old
```

---

### ✅ À retenir

- Toujours tourner en utilisateur non-root dans Docker
- systemd + Nginx = pattern fiable pour un VPS simple
- Variables d'environnement = jamais dans le code ni dans les images
- Migrations = avant le déploiement du code, pas après
- HTTPS = obligatoire en prod, Certbot/Let's Encrypt est gratuit et automatique
- CI/CD = tests → build image → push registre → deploy sur serveur

**Voir aussi →** [[nginx_guide]] pour la config Nginx avancée, [[services_demarrage]] pour systemd, [[docker_fondamentaux]] pour les bases Docker.
