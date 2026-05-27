---
title: "Loki — Centralisation des logs"
tags:
  - loki
  - logs
  - observabilité
  - grafana
  - devops
section: 05-DevOps
domaine: Monitoring
statut: actif
liens_connexes:
  - [[monitoring_prometheus_grafana]]
  - [[logs_supervision]]
  - [[docker_compose]]
  - [[kubernetes_fondamentaux]]
---

# 📘 Fiche : Loki — Centralisation des logs

---

## 1. Pourquoi centraliser les logs ?

Avec plusieurs services ou conteneurs, les logs sont éparpillés sur des dizaines de fichiers et machines. Les consulter un par un est impossible en production.

| Sans centralisation | Avec Loki |
|---|---|
| `ssh serveur1` puis `cat /var/log/app.log` | Interface Grafana unique |
| Impossible de corréler les logs de 5 services | Requêtes sur tous les services simultanément |
| Logs perdus si le conteneur est supprimé | Logs persistants dans Loki |

### La stack PLG (Promtail + Loki + Grafana)

```
Services / conteneurs
       ↓ (logs)
   Promtail          → collecte et envoie les logs
       ↓
     Loki            → stocke et indexe les logs
       ↓
   Grafana           → visualise les logs (même UI que les métriques)
```

> 💡 Loki s'intègre dans le même Grafana que Prometheus → une seule interface pour métriques ET logs.

---

## 2. Concepts clés

| Concept | Définition |
|---|---|
| **Loki** | Moteur de stockage de logs, indexe uniquement les labels (pas le contenu) |
| **Promtail** | Agent installé sur chaque machine/conteneur, collecte et pousse les logs vers Loki |
| **Alloy** | Successeur de Promtail (plus polyvalent), recommandé pour les nouveaux setups |
| **Label** | Métadonnée attachée à un flux de logs (`job`, `instance`, `container`...) |
| **LogQL** | Langage de requête Loki (inspiré de PromQL) |
| **Stream** | Flux de logs identifié par un ensemble de labels unique |

---

## 3. Installation avec Docker Compose

```yaml
# docker-compose.yml
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yml:/etc/loki/local-config.yaml
      - loki_data:/loki
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:latest
    volumes:
      - ./promtail-config.yml:/etc/promtail/config.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    command: -config.file=/etc/promtail/config.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  loki_data:
  grafana_data:
```

---

## 4. Configuration Loki (`loki-config.yml`)

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1

schema_config:
  configs:
    - from: 2024-01-01
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache
  filesystem:
    directory: /loki/chunks

limits_config:
  retention_period: 720h   # 30 jours
```

---

## 5. Configuration Promtail (`promtail-config.yml`)

```yaml
server:
  http_listen_port: 9080

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  # Logs système Linux
  - job_name: system
    static_configs:
      - targets: [localhost]
        labels:
          job: varlogs
          __path__: /var/log/*.log

  # Logs des conteneurs Docker
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
    relabel_configs:
      - source_labels: [__meta_docker_container_name]
        target_label: container
      - source_labels: [__meta_docker_compose_service]
        target_label: service
```

---

## 6. Ajouter Loki comme datasource dans Grafana

1. **Connections → Data sources → Add new**
2. Choisir **Loki**
3. URL : `http://loki:3100`
4. **Save & test**

---

## 7. LogQL — Requêtes Loki

LogQL est le langage de requête de Loki. Syntaxe de base :

```
{labels} | filtres
```

### Sélecteurs de stream (labels)
```logql
{job="varlogs"}                        # tous les logs du job "varlogs"
{container="nginx"}                     # logs du conteneur nginx
{service="api", env="production"}       # combinaison de labels
```

### Filtres sur le contenu
```logql
{container="api"} |= "error"           # contient "error"
{container="api"} != "debug"           # ne contient pas "debug"
{container="api"} |~ "error|warn"      # regex
{container="api"} !~ "healthcheck"     # exclure regex
```

### Métriques depuis les logs
```logql
# Taux d'erreurs par minute
rate({container="api"} |= "error" [1m])

# Nombre de lignes par service sur 5 min
sum by (service) (rate({job="docker"}[5m]))
```

---

## 8. Loki avec Kubernetes

Avec Helm, déploiement simplifié :

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Stack complète Loki + Grafana
helm install loki grafana/loki-stack \
  --set grafana.enabled=true \
  --set prometheus.enabled=true \
  -n monitoring --create-namespace
```

---

## 9. Bonnes pratiques

- Utiliser des **labels peu cardinals** (service, env, job) — pas de labels à valeur unique comme un user ID. Loki indexe les labels, pas le contenu.
- Structurer les logs en **JSON** côté application — LogQL peut parser les champs JSON.
- Définir une **rétention** adaptée (`retention_period`) pour maîtriser l'espace disque.
- Corréler logs et métriques dans Grafana via les **Explore** panels côte à côte.
- En production : utiliser le mode distribué de Loki (microservices) plutôt que single-binary.

---

## 10. ✅ À retenir

- **Loki** stocke les logs, **Promtail** les collecte, **Grafana** les visualise.
- Loki n'indexe que les **labels** (pas le contenu) → stockage efficace, requêtes sur les labels rapides.
- **LogQL** : `{labels} |= "filtre"` — proche de PromQL.
- S'intègre dans le même Grafana que Prometheus → stack observabilité unifiée.
- Démarrage rapide : `docker compose up` avec la stack PLG.
