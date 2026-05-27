---
title: "Déploiement frontend selon la technologie"
tags:
  - frontend
  - déploiement
  - nginx
  - docker
  - CI/CD
  - SPA
  - SSR
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche-composants-frontend]]
  - [[fiche-deploiement-backend]]
  - [[nginx-guide]]
  - [[Docker]]
---

# 📘 Déploiement frontend selon la technologie

## Comprendre le pipeline frontend

Avant de déployer, il faut comprendre ce que produit le build :

```
Code source (React/Vue/Svelte...)
        │
        ▼ npm run build
  Fichiers statiques
  (HTML, CSS, JS, assets)
        │
        ├── Serveur web (Nginx, Caddy)
        ├── CDN / hébergeur statique (Netlify, Vercel, S3)
        └── Image Docker (Nginx)
```

Pour le SSR (Next.js, Nuxt en mode serveur), le build produit aussi du code Node.js qui tourne côté serveur.

## 1. Sites statiques purs (HTML/CSS/JS)

Aucun build nécessaire, on sert directement les fichiers.

**Nginx :**
```nginx
server {
    listen 80;
    server_name monsite.com;
    root /var/www/monsite;
    index index.html;

    # Cache les assets statiques
    location ~* \.(css|js|png|jpg|ico|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Compression gzip
    gzip on;
    gzip_types text/css application/javascript image/svg+xml;
}
```

**GitHub Pages (gratuit, simple) :**
```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: .   # ou ./public si les fichiers sont dans un sous-dossier
      - uses: actions/deploy-pages@v4
```

## 2. SPA (React, Vue, Angular)

Le build produit un dossier `dist/` ou `build/` avec un `index.html` et des assets.

**Point critique : le fallback vers index.html**

Dans une SPA, le routage est géré par JavaScript. Si l'utilisateur accède directement à `/dashboard` ou `/users/42`, le serveur doit renvoyer `index.html` (qui chargera le JS qui gérera ensuite la route).

**Nginx avec fallback :**
```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
        # Essaie le fichier exact, puis le dossier, puis renvoie index.html
    }

    # Cache long pour les assets avec hash dans le nom (ex: main.a1b2c3.js)
    location ~* \.[0-9a-f]{8}\.(css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Pas de cache pour index.html (il change à chaque déploiement)
    location = /index.html {
        add_header Cache-Control "no-cache";
    }
}
```

**Build + déploiement manuel :**
```bash
npm run build              # génère dist/
rsync -av dist/ user@serveur:/var/www/monapp/
sudo nginx -s reload
```

**Déploiement via Netlify (drag and drop ou CLI) :**
```bash
npm install -g netlify-cli
netlify login
npm run build
netlify deploy --prod --dir=dist
```

## 3. SPA conteneurisée (Docker)

Pattern multi-stage : build dans un conteneur Node.js, puis copie dans un conteneur Nginx minimaliste.

```dockerfile
# Stage 1 : build
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2 : serveur de production
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

`nginx.conf` pour la SPA :
```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;
    location / { try_files $uri /index.html; }
    gzip on;
    gzip_types text/css application/javascript;
}
```

```bash
docker build -t mon-frontend .
docker run -p 8080:80 mon-frontend
```

**Avec docker-compose (frontend + backend + base de données) :**
```yaml
version: "3.9"
services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - api

  api:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/app

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

## 4. Next.js (React SSR/SSG)

Next.js peut être déployé en mode statique, Node.js, ou sur Vercel.

**Mode static export (pages sans données dynamiques) :**
```bash
# next.config.js
module.exports = { output: "export" }

npm run build   # génère out/
# Déployer out/ comme un site statique (Nginx, S3, Netlify)
```

**Mode Node.js (SSR + API routes) :**
```bash
npm run build   # génère .next/
node .next/standalone/server.js   # lancer le serveur

# Avec PM2 (gestionnaire de processus Node.js)
pm2 start .next/standalone/server.js --name mon-app
pm2 save
pm2 startup   # démarrer au boot
```

**Dockerfile pour Next.js en mode standalone :**
```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/.next/standalone ./
COPY --from=build /app/.next/static ./.next/static
COPY --from=build /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

**Sur Vercel (le plus simple pour Next.js) :**
```bash
npx vercel --prod
# ou connecter le repo GitHub dans l'interface Vercel
# → déploiement automatique à chaque push sur main
```

## 5. Pipeline CI/CD complet (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Build Docker image
        run: docker build -t ghcr.io/monorg/mon-frontend:${{ github.sha }} .

      - name: Push to registry
        run: |
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker push ghcr.io/monorg/mon-frontend:${{ github.sha }}

      - name: Deploy on server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull ghcr.io/monorg/mon-frontend:${{ github.sha }}
            docker stop frontend || true
            docker rm frontend || true
            docker run -d --name frontend -p 80:80 \
              ghcr.io/monorg/mon-frontend:${{ github.sha }}
```

## Stratégie de cache

Le cache bien configuré réduit drastiquement le temps de chargement pour les visiteurs qui reviennent.

```
index.html         → Cache-Control: no-cache (doit être rechargé à chaque déploiement)
main.[hash].js     → Cache-Control: max-age=31536000 (1 an, le hash change à chaque build)
vendor.[hash].js   → idem
logo.png           → Cache-Control: max-age=86400 (1 jour, pas de hash)
```

Le hash dans les noms de fichiers (généré par Vite/webpack) garantit que le navigateur recharge le fichier quand son contenu change.

---

### ✅ À retenir

- SPA → fallback `try_files $uri /index.html` dans Nginx (indispensable)
- Multi-stage Dockerfile → build en Node, serve en Nginx alpine (image légère)
- Next.js → Vercel pour la simplicité, ou mode standalone pour Docker
- `index.html` sans cache, assets avec hash = cache 1 an
- CI/CD → build → tests → image Docker → push registry → deploy

**Voir aussi →** [[fiche-composants-frontend]] pour les concepts, [[nginx-guide]] pour configurer Nginx en détail.
