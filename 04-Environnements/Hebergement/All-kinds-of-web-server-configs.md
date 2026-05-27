# 📘 Fiche détaillée – Rôles et configurations possibles d’un serveur web

Un serveur web moderne (Nginx, Apache, Caddy, Traefik, Envoy…) peut remplir différents **rôles** en fonction de la configuration. Voici une synthèse des principaux modes.

---

## 1. Static Web Server (serveur de fichiers statiques)
- Sert directement des **fichiers HTML, CSS, JS, images, vidéos**.
- Rapide et léger.
- Exemple : un site vitrine ou un frontend SPA (React/Angular/Vue) déjà compilé.

**Config Nginx minimale :**
```nginx
server {
  listen 80;
  server_name example.com;
  root /var/www/html;
  index index.html;
}
```

---

## 2. Reverse Proxy
- Sert d’**intermédiaire** entre les clients et les serveurs internes.
- Cache l’architecture interne, ajoute du load balancing, SSL, logs.

**Config type :**
```nginx
server {
  listen 80;
  location / {
    proxy_pass http://127.0.0.1:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

---

## 3. Forward Proxy
- Placé côté **client**.
- Relais entre clients et Internet.
- Sert à filtrer, contrôler, ou anonymiser le trafic.

**Cas d’usage :**
- Proxy d’entreprise (bloque certains sites).
- Contrôle parental.
- Cache partagé pour plusieurs utilisateurs.

---

## 4. API Gateway
- Reverse proxy spécialisé pour **API/microservices**.
- Ajoute : auth, rate limiting, CORS, versioning, agrégation de réponses.

**Config simplifiée :**
```nginx
upstream users_api { server 127.0.0.1:7001; }
upstream orders_api { server 127.0.0.1:7002; }

server {
  listen 80;
  location /users/ { proxy_pass http://users_api; }
  location /orders/ { proxy_pass http://orders_api; }
}
```

---

## 5. Load Balancer
- Répartit la charge sur plusieurs serveurs.
- Algorithmes disponibles (Nginx OSS) :
  - **round robin** (par défaut)
  - **least_conn**
  - **ip_hash** (sticky par IP)
  - **hash clé** (ex: ID utilisateur)

**Exemple round robin :**
```nginx
upstream backend_pool {
  server 10.0.0.11:8080;
  server 10.0.0.12:8080;
}
server {
  location / { proxy_pass http://backend_pool; }
}
```

---

## 6. Cache HTTP / CDN Edge
- Mise en cache des réponses côté proxy/CDN pour réduire latence et charge.

**Exemple cache local Nginx :**
```nginx
proxy_cache_path /tmp/nginx levels=1:2 keys_zone=my_cache:10m;
server {
  location /api/ {
    proxy_cache my_cache;
    proxy_pass http://api_backend;
  }
}
```

---

## 7. WAF (Web Application Firewall)
- Protection contre attaques (SQLi, XSS, bruteforce).
- Avec **ModSecurity** intégré à Nginx/Apache.

**Cas d’usage :**
- Sites sensibles (banques, e-commerce).
- Conformité sécurité (PCI-DSS).

---

## 8. WebSocket / gRPC Gateway
- Supporte des protocoles temps réel ou orientés microservices.

**WebSocket pass-through (Nginx) :**
```nginx
location /ws/ {
  proxy_pass http://backend_ws;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}
```

**gRPC :**
```nginx
location /grpc/ {
  grpc_pass grpc://127.0.0.1:50051;
}
```

---

## 9. TLS Terminator
- Déchiffre TLS (HTTPS) côté proxy, envoie du HTTP clair aux backends.
- Simplifie la gestion des certificats.

**Exemple :**
```nginx
server {
  listen 443 ssl;
  ssl_certificate     /etc/ssl/certs/site.crt;
  ssl_certificate_key /etc/ssl/private/site.key;

  location / {
    proxy_pass http://127.0.0.1:8080;
  }
}
```

---

# 📊 Tableau comparatif rapide

| Mode                     | Utilité principale | Exemple typique |
|--------------------------|-------------------|-----------------|
| Static web server        | Servir des fichiers statiques | Site vitrine, frontend SPA |
| Reverse proxy            | Masquer/relayer vers backends | Site e-commerce derrière plusieurs serveurs |
| Forward proxy            | Filtrer côté client | Proxy d’entreprise |
| API Gateway              | Gérer l’accès aux API | Microservices SaaS |
| Load balancer            | Répartir charge | Cluster d’applications |
| Cache / CDN              | Accélérer réponses | Site média, API à fort trafic |
| WAF                      | Bloquer attaques | Banque, e-commerce |
| WebSocket/gRPC Gateway   | Support temps réel | Chat, IoT, gRPC microservices |
| TLS terminator           | Centraliser SSL | SaaS avec certificats multiples |

---

# ✅ Conclusion
- **Un serveur web n’est pas qu’un "serveur de fichiers"** : il peut être **reverse proxy, gateway, cache, load balancer, firewall**…  
- Dans la pratique, on combine plusieurs rôles (ex : **API Gateway + Load Balancer + TLS terminator + WAF**).  
