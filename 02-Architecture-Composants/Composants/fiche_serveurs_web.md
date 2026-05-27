---
title: "Les serveurs web"
tags:
  - composants
  - Nginx
  - Apache
  - proxy
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[nginx_guide]]
  - [[hebergement]]
  - [[fiche_reseaux]]
---

# 🌍 Le serveur web — Fiche complète

## Sommaire
- [1. Définition & rôles](#1-définition--rôles)
- [2. Cycle de vie d’une requête HTTP](#2-cycle-de-vie-dune-requête-http)
- [3. Hôtes virtuels & routage](#3-hôtes-virtuels--routage)
- [4. HTTPS/TLS, HTTP/2 & HTTP/3](#4-httpstls-http2--http3)
- [5. Reverse proxy vs Load balancer vs App server](#5-reverse-proxy-vs-load-balancer-vs-app-server)
- [6. Caching, compression & static files](#6-caching-compression--static-files)
- [7. Connexion aux backends (PHP‑FPM, Node, Java…)](#7-connexion-aux-backends-phpfpm-node-java)
- [8. Sécurité (en-têtes, CORS, rate‑limiting, WAF)](#8-sécurité-en-têtes-cors-rate-limiting-waf)
- [9. Logs, observabilité & performance](#9-logs-observabilité--performance)
- [10. Déploiement & exemples de conf](#10-déploiement--exemples-de-conf)
- [📊 Tableau récapitulatif](#-tableau-récapitulatif)
- [🧭 Arbre de décision](#-arbre-de-décision)

---

## 1. Définition & rôles
Il peut :
- Servir des **fichiers statiques** (HTML, CSS, JS, images),  
- **Proxifier** des requêtes vers des **backends** (Node/Express, PHP‑FPM, Java/Tomcat),  
- Jouer le rôle de **reverse proxy** / **load balancer**,  
- Gérer **TLS/HTTPS**, **compression**, **caching**, **rewrites** et redirections.

---

## 2. Cycle de vie d’une requête HTTP
![HTTP lifecycle](img/http_lifecycle.png)

1) Résolution DNS → IP, connexion TCP, **négociation TLS** si HTTPS.  
2) Le serveur web lit **Host** (vhost), **Path** (routage), **méthode** (GET/POST…), **en‑têtes** et **corps**.  
3) Selon les règles : fichier **statique**, **proxy** vers backend, ou **erreur** (404/403/500).  
4) Réponse : **status**, **headers** (cache, CORS, CSP…), **payload** (HTML/JSON, fichiers), éventuellement **compression** (gzip/br).

---

## 3. Hôtes virtuels & routage
- **Virtual hosts** : un même serveur écoute plusieurs **noms de domaine**.  
- **Routage par chemin** : `/assets/` → statique ; `/api/` → backend.  
- **Priorités** : l’ordre des règles et **emplacements** (locations) compte.

**Nginx (extrait)** :
```nginx
server {
  listen 443 ssl http2;
  server_name www.exemple.com;
  root /var/www/site/public;

  location /assets/ { try_files $uri =404; }
  location /api/    { proxy_pass http://localhost:3000; }
}
```

**Apache (extrait)** :
```apache
<VirtualHost *:443>
  ServerName www.exemple.com
  DocumentRoot "/var/www/site/public"
  ProxyPass "/api/" "http://localhost:3000/"
  ProxyPassReverse "/api/" "http://localhost:3000/"
</VirtualHost>
```


---

## 4. HTTPS/TLS, HTTP/2 & HTTP/3
![TLS split](img/tls_static_dynamic.png)

- **TLS** : chiffrement des échanges + authentification du serveur (certificats).  
- **HTTP/2** : multiplexage sur une connexion → **latence réduite**.  
- **HTTP/3** (QUIC/UDP) : plus robuste aux pertes, latences plus faibles.  
- **ALPN** choisit le protocole (h2, h3). **SNI** sélectionne le bon certificat selon le nom d’hôte.

---

## 5. Reverse proxy vs Load balancer vs App server
![Reverse proxy + LB](img/reverse_proxy_lb.png)

- **Reverse proxy** : termine TLS, applique des règles (rewrites, headers, CORS), **forward** vers les backends.  
- **Load balancer** : répartit la charge (round‑robin, least‑conn, IP‑hash), **health checks**.  
- **App server** : exécute la **logique métier** (Express, PHP‑FPM, Spring).  
👉 En pratique, un même logiciel peut faire RP **et** LB (Nginx, Traefik, HAProxy).

---

## 6. Caching, compression & static files
![Caching layers](img/caching_layers.png)

- **Caching HTTP** : `Cache-Control`, `ETag`, `Last-Modified`, `Vary`.  
- Caches : **CDN (edge)**, **reverse proxy** (Nginx/Varnish), **navigateur**.  
- **Compression** : `gzip`, `br` (Brotli), **minification** côté build.  
- **Static files** : headers de longue durée sur `/assets/` avec **fingerprinting** (hash dans le nom de fichier).

---

## 7. Connexion aux backends (PHP‑FPM, Node, Java…)
![Backends](img/backends_matrix.png)

- **PHP‑FPM** : FastCGI (`fastcgi_pass unix:/run/php-fpm.sock`).  
- **Node.js/Express** : `proxy_pass http://localhost:3000`.  
- **Java/Spring/Tomcat** : `proxy_pass http://localhost:8080`.  
- **.NET/Kestrel** : `proxy_pass http://localhost:5000`.

**Exemple Nginx (PHP‑FPM)** :
```nginx
location ~ \.php$ {
  include fastcgi_params;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  fastcgi_pass unix:/run/php/php-fpm.sock;
}
```

---

## 8. Sécurité (en-têtes, CORS, rate‑limiting, WAF)
- **En‑têtes** : `Content-Security-Policy`, `X-Frame-Options`, `Referrer-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`.  
- **CORS** : autoriser des origines spécifiques, éviter `*` sur les credentials.  
- **Rate‑limiting / WAF** : freiner les abus, bloquer OWASP Top 10, **bot management**.  
- **mTLS** (mutual TLS) possible entre reverse proxy et backends.

---

## 9. Logs, observabilité & performance
- **Access logs** (format JSON), **error logs**, **correlation‑id**.  
- **Metrics** (requests/sec, latence p50/p95/p99, taux d’erreur) → Prometheus/Grafana.  
- **Tracing** : exporter headers `traceparent` / OpenTelemetry.  
- **Tuning** : workers/processes, keep‑alive, buffers, limites taille requête, TLS session tickets.

---

## 10. Déploiement & exemples de conf

### Nginx (site statique + API Node)
```nginx
server {
  listen 443 ssl http2;
  server_name www.exemple.com;
  root /var/www/site/public;
  location /assets/ { expires 30d; add_header Cache-Control "public, max-age=2592000"; }
  location /api/    { proxy_pass http://127.0.0.1:3000; proxy_set_header Host $host; }
}
```

### Apache (proxy + compression)
```apache
<VirtualHost *:443>
  ServerName api.exemple.com
  ProxyPass        / http://127.0.0.1:8080/
  ProxyPassReverse / http://127.0.0.1:8080/
  AddOutputFilterByType DEFLATE text/html text/css application/javascript
</VirtualHost>
```


---



## 🔧 Exemple complet de configuration Nginx

```nginx
# Upstreams (pool API)
upstream api_upstream {
    server 127.0.0.1:3001 max_fails=3 fail_timeout=10s;
    server 127.0.0.1:3002 max_fails=3 fail_timeout=10s;
    keepalive 32;
}

# HTTP -> HTTPS (site)
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

# HTTP -> HTTPS (API)
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}

# HTTPS - site www.example.com
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate     /etc/ssl/certs/example.com/fullchain.pem;
    ssl_certificate_key /etc/ssl/private/example.com/privkey.pem;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options nosniff always;

    gzip on;
    gzip_types text/plain text/css application/javascript application/json image/svg+xml;

    root /var/www/site/public;
    index index.html;

    location /assets/ {
        try_files $uri =404;
        expires 30d;
        add_header Cache-Control "public, max-age=2592000, immutable";
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}

# HTTPS - API api.example.com
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate     /etc/ssl/certs/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/ssl/private/api.example.com/privkey.pem;

    location / {
        proxy_pass http://api_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_http_version 1.1;
        proxy_read_timeout 60s;
        proxy_buffering off;
    }
}
```



## 🧪 À vérifier rapidement après déploiement (Nginx)

### 1. Valider la configuration
```bash
sudo nginx -t
```
👉 Vérifie la syntaxe et la présence des certificats.

### 2. Recharger sans couper les connexions
```bash
sudo systemctl reload nginx
```

### 3. Vérifier la redirection HTTP→HTTPS
```bash
curl -I http://example.com
```

### 4. Vérifier TLS & HTTP/2
```bash
openssl s_client -connect example.com:443 -servername example.com
```

### 5. Vérifier les headers de sécurité
```bash
curl -I https://example.com | grep -E "Strict|X-|Referrer"
```

### 6. Vérifier le cache des assets
```bash
curl -I https://example.com/assets/app.js
```

### 7. Vérifier l’API reverse proxy
```bash
curl -I https://api.example.com/health
```

### 8. Vérifier WebSocket
```bash
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket"      -H "Host: api.example.com" -H "Origin: https://example.com"      https://api.example.com/ws/
```

### 9. Suivre les logs
```bash
tail -f /var/log/nginx/www_access.log
tail -f /var/log/nginx/api_error.log
```

## 📊 Tableau récapitulatif

| Sujet | À retenir | Avantages | Limites |
|---|---|---|---|
| Reverse proxy | Terminer TLS, router vers backends | Sécurité, flexibilité | Complexité des règles |
| Load balancer | Répartir la charge + health checks | HA, scalabilité | Sessions collantes à gérer |
| Static files | Servir via CDN/proxy | Très performant | Invalidations de cache |
| Caching | HTTP, CDN, proxy | Diminue la charge | Cohérence à gérer |
| Compression | gzip/br | Gain bande passante | CPU côté serveur |
| PHP‑FPM/Node/Java | Connecteurs adaptés | Découplage web/app | Supervision multi‑processe |
| Sécurité | Headers, CORS, WAF, mTLS | Réduit surface d’attaque | Faux positifs WAF |
| Observabilité | Logs, métriques, traces | Diagnostic rapide | Coûts de stockage |
| HTTP/2/3 | Multiplexage, QUIC | Latence réduite | Compat. réseau à vérifier |

---

## 🧭 Arbre de décision

1) **Site statique** mondial → **CDN** (+ compression) ; serveur web minimal.  
2) **App Node/PHP/Java** → **reverse proxy** (TLS, headers, cache statique) → **backend**.  
3) **Trafic élevé** → ajouter **load balancer** + **health checks** ; sessions **stateless**.  
4) **Performance** → **HTTP/2**, **gzip/br**, **cache** HTTP + CDN.  
5) **Sécurité** → **HSTS**, **CSP**, **WAF**, **rate‑limit**, **CORS** strict.  
6) **Observabilité** → logs JSON, métriques, traces (OpenTelemetry).

---

### Conseils pratiques
- Tenir la **config sous Git** et la **tester** (CI) avant prod.  
- Préférer des **règles simples** et explicites ; documenter le routage.  
- Mesurer (latence, erreurs) avant d’optimiser.  
