---
title: "Composants d’un backend moderne"
tags:
  - backend
  - architecture
  - api
  - base-de-données
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche_backend]]
  - [[fiche_api]]
  - [[fiche_database]]
  - [[fiche_architecture_systemes]]
  - [[fiche_auth_sessions]]
  - [[fiche_securite_dev]]
  - [[fiche_stockage_donnees]]
  - [[fiche_droits_autorisations]]
  - [[kubernetes_fondamentaux]]
  - [[docker_fondamentaux]]
  - [[tests_unitaires]]
  - [[tests_integrations]]
---

# ⚙️ Fiche détaillée — Composants d’un backend moderne

Cette fiche recense les **composants** qu’on retrouve le plus souvent dans un backend, avec leurs rôles, options technologiques, bonnes pratiques et pièges courants.

---

## 1) Serveur applicatif (logiciel métier)
**Rôle** : héberge la logique métier, expose des endpoints, orchestre les dépendances.

**Technos fréquentes**  
- **Node.js** : Express, NestJS, Fastify, Hapi  
- **Python** : Django, FastAPI, Flask  
- **Java/JVM** : Spring Boot, Quarkus, Micronaut, Ktor (Kotlin)  
- **Go** : net/http, Gin, Echo, Fiber  
- **Ruby** : Rails, Hanami

**Bonnes pratiques**  
- Séparer **couches** : contrôleurs ⇢ services ⇢ dépôts (repositories).  
- Validation d’input systématique (DTO, pydantic, class-validator…).  
- Gestion d’erreurs centralisée (middleware).  
- Gestion du **graceful shutdown** (SIGTERM) pour containers et rolling updates.

**Pièges**  
- Couplage fort à la base de données.  
- Gestion concurrente/async mal maîtrisée (fuites, deadlocks).

---

## 2) Interface réseau & API → [[fiche_api]]
**Rôle** : protocole et contrat d’échange avec les clients.

**Styles**  
- **REST** (HTTP/JSON) : le plus répandu, simple, cache HTTP.  
- **GraphQL** : flexibilité des requêtes, un endpoint unique, nécessite une gouvernance de schéma.  
- **gRPC** : binaire (Protobuf), très performant, idéal inter-services.  
- **WebSocket / SSE** : temps réel, push serveur → client.

**Transverses**  
- **Versioning** : `/v1`, `/v2` ou version dans l’entête.  
- **CORS** : autoriser les origins nécessaires uniquement.  
- **Pagination / tri / filtrage** : conventions claires (`?page=…&limit=…`).  
- **Idempotence** : clés idempotentes pour POST sensibles (paiement).

**Pièges**  
- Schémas non documentés (solution : OpenAPI/Swagger, GraphQL SDL).  
- Mélange des responsabilités (auth, transformation) : externaliser dans Gateway si possible.

---

## 3) Stockage persistant (bases de données) → [[fiche_database]]
**Rôle** : conserver l’état de l’application.

**Types**  
- **SQL** : PostgreSQL, MySQL/MariaDB, SQL Server, CockroachDB (scaling distribué).  
- **NoSQL** : MongoDB (documents), DynamoDB (clé-valeur), Cassandra/Scylla (large scale), Redis (clé-valeur volatile persistant possible).  
- **Time-series** : TimescaleDB, InfluxDB.  
- **Graph** : Neo4j, JanusGraph.

**Bonnes pratiques**  
- Migrations versionnées (Flyway, Liquibase, Alembic, Prisma).  
- Indexation ciblée, audit des requêtes lentes.  
- Séparation **lecture/écriture** (réplicas read-only) si besoin.  
- Sauvegardes + tests de **restauration** (DR).

**Pièges**  
- N+1 queries (utiliser includes/joins, DataLoader, repos).  
- Transactions longues, verrous pessimistes.

---

## 4) Cache (mémoire & distribué)
**Rôle** : réduire la latence et la charge sur la base.

**Outils** : **Redis**, Memcached; caches applicatifs (Caffeine Java), HTTP cache (Varnish, Nginx `proxy_cache`).

**Patterns**  
- **Cache-aside** (lazy): lire cache → si miss, DB puis populate.  
- **Write-through / write-behind** selon cohérence désirée.  
- **TTL** et **invalidation** explicites.

**Pièges**  
- Stale data (données périmées) : bien définir TTL et invalidations par clé.  
- Sur-caching de réponses personnalisées (oublier `Vary`).

---

## 5) Messagerie / Streaming (asynchrone)
**Rôle** : découpler, lisser la charge, tolérer l’intermittence.

**Outils**  
- **Queues** : RabbitMQ, ActiveMQ, SQS.  
- **Streaming** : Kafka, Redpanda, Pulsar (events ordonnés et relecture).

**Patterns**  
- **Pub/Sub** : broadcast d’événements (ex. “user.created”).  
- **Work queues** : tâches asynchrones (emails, thumbnails).  
- **Outbox** : fiabilité entre DB et bus d’événements.

**Pièges**  
- Perte d’événements (mauvais acks).  
- Consommateurs non idempotents.

---

## 6) Authentification, Autorisation & Sécurité → [[fiche_auth_sessions]] · [[fiche_droits_autorisations]] · [[fiche_securite_dev]]
**Rôle** : prouver l’identité et contrôler l’accès.

**Mécanismes**  
- **Auth** : sessions (cookies + SameSite), **JWT** (exp court + rotation), OAuth2/OIDC (PKCE), API keys.  
- **Authorization** : RBAC/ABAC, policies (OPA/Styra, Casbin).  
- **Sécurité** : TLS obligatoire, hashing des mots de passe (bcrypt/argon2) → [[fiche_stockage_donnees]], rate limiting, validation d’input, entêtes sécurité (CSP, HSTS).

**Pièges**  
- JWT trop longs/éternels (pas de rotation / revoke).  
- Mauvaise gestion des cookies (SameSite=None + Secure pour cross-site).

---

## 7) Intégrations externes (paiement, email, stockage…)
**Rôle** : étendre les capacités via des services tiers.

**Domaines**  
- **Paiement** : Stripe, Adyen.  
- **Email/SMS** : Postmark, Sendgrid, SES, Twilio.  
- **Stockage objet** : S3/MinIO, GCS, Azure Blob.  
- **Recherche** : Elasticsearch, OpenSearch, Meilisearch.  
- **Notifications push** : FCM/APNS, Web Push.

**Bonnes pratiques**  
- Timeout + retry + **circuit breaker** (resilience).  
- Webhooks sécurisés (secrets, signatures, replays).  
- Idempotence des callbacks.

---

## 8) Observabilité (logs, métriques, traces) → [[logs_supervision]]
**Rôle** : diagnostiquer, surveiller, alerter.

**Piliers**  
- **Logs** structurés (JSON) → EFK/ELK, Loki.  
- **Métriques** (Prometheus) → Dashboards Grafana, alertes (Alertmanager).  
- **Traces distribuées** : OpenTelemetry + Jaeger/Tempo/Zipkin.

**Bonnes pratiques**  
- **Correlation IDs** (X-Request-Id) propagés.  
- SLO/SLI définis (latence P95, taux d’erreurs).  
- Alerter sur symptômes, pas sur implémentations.

---

## 9) Orchestration, déploiement & mise à l’échelle → [[docker_fondamentaux]] · [[kubernetes_fondamentaux]]
**Rôle** : faire tourner et scaler les composants en prod.

**Composants**  
- **Reverse proxy / LB** : Nginx → [[fiche_serveurs_web]], HAProxy, Envoy.  
- **Orchestration** : [[docker_fondamentaux]], [[kubernetes_fondamentaux]] (Deployments, Services, Ingress).  
- **CI/CD** : pipelines build/test/deploy → [[02_setup_gitea_jenkins]], blue/green, canary.  
- **Secrets & config** : Vault, KMS, K8s Secrets/ConfigMaps.

**Bonnes pratiques**  
- Health checks (liveness/readiness).  
- **Autoscaling** sur CPU/RAM/latence/queue depth.  
- Rollbacks rapides, images immuables, SBOM/signatures.

---

## 10) Stockage de fichiers & médias
**Rôle** : recevoir, transformer, servir des assets (images, PDF, vidéo).

**Options**  
- Stockage objet (S3-compatible) + **presigned URLs**.  
- CDN pour diffusion (caching, redimensionnement à la volée).  
- Pipelines : redimensionnement, transcodage, antivirus.

**Pièges**  
- Servir directement depuis l’app (bloquant).  
- Manque de quotas/taille max, pas de scan AV.

---

## 11) Tâches planifiées (scheduler)
**Rôle** : exécuter des jobs à intervalles réguliers.

**Outils**  
- Cron/cron-like (systemd timers, Kubernetes CronJob).  
- Librairies : BullMQ/Agenda (Node), Celery/APScheduler (Python), Quartz (Java).

**Bonnes pratiques**  
- Idempotence des jobs, verrou distribué (ex: Redis) pour éviter les doublons.  
- Observabilité : logs + métriques de réussite/échec.

---

## 12) Gouvernance des schémas & contrats
**Rôle** : maîtriser l’évolution des contrats API et des schémas.

**Outils & pratiques**  
- **OpenAPI/Swagger**, GraphQL SDL + codegen.  
- Contrat-first & tests de **compatibilité** (consumer-driven contracts : Pact).  
- Versioning clair, dépréciations annoncées.

---

# 🧱 Exemples d’architectures « type »

### A. API REST simple avec cache
```
Client ─► Nginx (TLS, CORS) ─► App (Express/FastAPI) ─► PostgreSQL
                               └────────► Redis (cache)
```

### B. Microservices événementiels
```
Client ─► API Gateway ─► Services (users, orders, billing)
                           │        │        │
                           └─► Kafka/Redpanda (events) ─► Workers
                                   │
                                   └─► Data lake / OLAP (analytics)
```

### C. Temps réel & fichiers
```
Client (WebSocket) ─► Gateway ─► Service RT (WS) ─► Redis (pub/sub)
Client (upload)    ─► Gateway ─► App ─► S3/MinIO (+ presigned URLs) ─► CDN
```

---

# ✅ Checklist rapide (mise en prod)
- [ ] TLS strict (HSTS), entêtes sécurité (CSP, X-Frame-Options…).  
- [ ] Authn/Authz robustes (JWT rotatifs, scopes/roles).  
- [ ] Logs JSON + métriques + traces (OTel) + dashboards.  
- [ ] Backups & restore testés, migrations atomiques.  
- [ ] Health checks, readiness, autoscaling, budgets d’erreurs (SLO).  
- [ ] Rate limiting, circuit breakers, timeouts/retries.  
- [ ] Secrets gérés hors code, rotation planifiée.  
- [ ] Plans d’incident, runbooks, alertes pertinentes.

---

# 📎 Annexes — snippets utiles

**Nginx : CORS + proxy de base**
```nginx
location /api/ {
  add_header Access-Control-Allow-Origin https://app.example.com always;
  add_header Vary Origin always;
  if ($request_method = OPTIONS) {
    add_header Access-Control-Allow-Methods "GET,POST,PUT,DELETE,OPTIONS" always;
    add_header Access-Control-Allow-Headers "Authorization,Content-Type" always;
    return 204;
  }
  proxy_pass http://api_backend;
}
```

**Express.js : gestion d’erreurs centralisée**
```js
app.use((err, req, res, next) => {
  console.error({ err });
  res.status(err.status || 500).json({ error: 'internal_error' });
});
```

**OpenTelemetry (pseudo)**
```txt
Tracer.startSpan('db.query').setAttribute('sql.table','orders')...
```

---

**TL;DR** : Un backend solide s’appuie sur 12 briques : **app**, **API**, **DB**, **cache**, **messagerie**, **sécurité**, **intégrations**, **observabilité**, **orchestration**, **stockage fichiers**, **scheduler**, **gouvernance**. Choisis ce qui est nécessaire **maintenant**, garde le reste **évolutif**.
