---
title: "Guide Nginx"
tags:
  - devops
  - nginx
  - reverse-proxy
  - config
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[fiche_serveurs_web]]
  - [[hebergement]]
---

# Déploiement local : Nginx + front statique + API Express (`/api`)

## Pourquoi utiliser Nginx ?

Nginx est utilisé ici comme **serveur frontal** (reverse proxy) pour deux raisons principales :

1. **Servir efficacement le front statique**  
   Nginx est très rapide pour distribuer des fichiers statiques (`index.html`, CSS, JS, images).
   C’est sa spécialité, et il le fait mieux qu’un serveur Node.

2. **Rediriger les requêtes API vers Node.js**  
   En gardant le front et le back sous un même « domaine local », on simplifie la communication
   (ex : éviter certains soucis CORS) tout en gardant une séparation claire des rôles.

En résumé :
**Nginx sert le front**, **proxifie les appels `/api` vers l’API Node**, **et masque l’architecture interne**.
Les utilisateurs ne voient qu’une seule "entrée" même si plusieurs services tournent derrière.  
Node n’a alors qu’à gérer la logique applicative, pas le service de fichiers.

---

## 1) Arborescence

```
taskboard/
  public/        # front statique (index.html, assets/*)
  src/           # back Node.js / Express (API déjà prête)
```

> L’API écoute en local (ex : `127.0.0.1:3000`).

---

## 2) Configuration Nginx minimale

Créer : `/etc/nginx/sites-available/taskboard`

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name localhost;  # remplacé par un domaine local si utilisé (voir section suivante)

    root /taskboard/public;
    index index.html;

    # Front (SPA)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API → Node
    location /api/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
    }
}
```

---

## 3) (Optionnel) Utiliser un nom de domaine local

1. Modifier `/etc/hosts` :
   ```
   127.0.0.1   taskboard.local
   ```

2. Dans Nginx :
   ```
   server_name taskboard.local;
   ```

3. Accéder à l’application via :
   ```
   http://taskboard.local/
   ```

---

## 4) Amélioration — Mettre en place HTTPS en local avec mkcert

### A) Installation (Debian 12)

```bash
sudo apt update
sudo apt install -y mkcert libnss3-tools
mkcert -install
```

- `mkcert` : génère des certificats locaux valides.
- `libnss3-tools` : permet à Chrome/Chromium de reconnaître automatiquement ces certificats.

### B) Générer le certificat local

```bash
mkdir -p /taskboard/certs/
cd /taskboard/certs/
mkcert taskboard.local
```

Cela génère :
```
taskboard.local.pem
taskboard.local-key.pem
```

### C) Ajouter le bloc HTTPS dans Nginx

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name taskboard.local;

    ssl_certificate     /taskboard/certs/taskboard.local.pem;
    ssl_certificate_key /taskboard/certs/taskboard.local-key.pem;

    root /taskboard/public;
    index index.html;

    location / { try_files $uri $uri/ /index.html; }

    location /api/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
    }
}
```

### D) (Optionnel) Rediriger HTTP → HTTPS

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name taskboard.local;
    return 301 https://$host$request_uri;
}
```

---

## Résultat attendu

| URL                         | Rôle                         | Notes |
|----------------------------|------------------------------|-------|
| `http://localhost/`        | Front statique               | configuration minimale |
| `http://localhost/api/...` | API Express                  | proxifiée vers Node.js |
| `https://taskboard.local/` | Front en HTTPS               | certificat reconnu localement grâce à mkcert |
