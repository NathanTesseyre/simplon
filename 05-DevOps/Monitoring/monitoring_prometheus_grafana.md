---
title: "Monitoring — Prometheus & Grafana"
tags:
  - monitoring
  - prometheus
  - grafana
  - observabilité
  - devops
section: 05-DevOps
domaine: Monitoring
statut: actif
liens_connexes:
  - [[logs_supervision]]
  - [[fiche_securite_systeme]]
  - [[kubernetes_fondamentaux]]
  - [[docker_compose]]
---

# 📘 Fiche : Monitoring — Prometheus & Grafana

---

## 1. Les 3 piliers de l'observabilité

| Pilier | Quoi | Outils |
|---|---|---|
| **Logs** | Événements texte horodatés | journald, Loki, ELK |
| **Métriques** | Valeurs numériques dans le temps | **Prometheus**, InfluxDB |
| **Traces** | Parcours d'une requête entre services | Jaeger, Tempo, Zipkin |

> 💡 Pour un Junior SysAdmin DevOps, les **métriques** (Prometheus + Grafana) sont le point d'entrée le plus courant.

---

## 2. Prometheus — Collecte de métriques

### Principe de fonctionnement

```
Cibles (apps, serveurs)        Prometheus              Grafana
   exposent /metrics   →   scrape toutes les N s  →   visualise
```

Prometheus **tire** les métriques depuis les cibles (pull model) — contrairement à d'autres outils qui reçoivent les données (push).

### Types de métriques

| Type | Description | Exemple |
|---|---|---|
| **Counter** | Valeur qui ne fait qu'augmenter | nombre total de requêtes |
| **Gauge** | Valeur qui monte et descend | RAM utilisée, connexions actives |
| **Histogram** | Distribution de valeurs (latences, tailles) | durée des requêtes par bucket |
| **Summary** | Quantiles calculés côté client | p95, p99 de latence |

### Exporters — exposer des métriques

Un **exporter** est un agent qui lit des métriques système/service et les expose au format Prometheus.

| Exporter | Ce qu'il expose |
|---|---|
| `node_exporter` | CPU, RAM, disque, réseau du serveur |
| `cadvisor` | Métriques des conteneurs Docker |
| `postgres_exporter` | Métriques PostgreSQL |
| `nginx-prometheus-exporter` | Métriques Nginx |
| `blackbox_exporter` | Disponibilité HTTP, DNS, TCP (monitoring externe) |

---

## 3. Installation avec Docker Compose

```yaml
# docker-compose.yml
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

  node_exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'

volumes:
  prometheus_data:
  grafana_data:
```

### Configuration Prometheus (`prometheus.yml`)

```yaml
global:
  scrape_interval: 15s      # fréquence de collecte

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["node_exporter:9100"]

  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

---

## 4. PromQL — Requêtes Prometheus

PromQL est le langage de requête de Prometheus.

```promql
# CPU utilisé (en %)
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# RAM disponible
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100

# Espace disque utilisé
(node_filesystem_size_bytes - node_filesystem_avail_bytes) / node_filesystem_size_bytes * 100

# Taux d'erreurs HTTP (si app instruite)
rate(http_requests_total{status=~"5.."}[5m])

# Latence p95
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

---

## 5. Grafana — Visualisation

### Accès initial
- URL : `http://localhost:3000`
- Login par défaut : `admin` / `admin`

### Ajouter Prometheus comme datasource
1. **Connections → Data sources → Add new**
2. Choisir **Prometheus**
3. URL : `http://prometheus:9090`
4. **Save & test**

### Dashboards prêts à l'emploi
Grafana propose des dashboards communautaires sur [grafana.com/dashboards](https://grafana.com/grafana/dashboards/).

| Dashboard ID | Contenu |
|---|---|
| `1860` | Node Exporter Full — CPU, RAM, disque, réseau |
| `893` | Docker et conteneurs (cAdvisor) |
| `9628` | PostgreSQL |

Import : **Dashboards → Import → saisir l'ID**

---

## 6. Alertmanager — Alertes

**Alertmanager** reçoit les alertes déclenchées par Prometheus et les route (email, Slack, PagerDuty...).

### Définir une règle d'alerte (`alerts.yml`)
```yaml
groups:
  - name: infra
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Moins de 10% d'espace disque sur {{ $labels.instance }}"
```

---

## 7. Bonnes pratiques

- Commencer par les **métriques système** (node_exporter) avant d'instrumenter le code.
- Définir des **SLO** (Service Level Objectives) clairs avant de créer des alertes — alerter sur les symptômes, pas les causes.
- Ne pas alerter sur tout : chaque alerte doit nécessiter une action humaine.
- Stocker les données Prometheus dans un **volume nommé** — la rétention par défaut est 15 jours.
- En production : utiliser **Thanos** ou **VictoriaMetrics** pour le stockage long terme.
- Protéger Grafana et Prometheus derrière un **reverse proxy avec authentification**.

---

## 8. ✅ À retenir

- **Prometheus** collecte les métriques en interrogeant des cibles toutes les N secondes.
- **Exporters** exposent les métriques des systèmes (serveurs, BDD, conteneurs).
- **Grafana** visualise les métriques via des dashboards.
- **Alertmanager** envoie des notifications quand des règles sont déclenchées.
- Démarrage rapide : `docker compose up` avec Prometheus + Grafana + node_exporter.
- Dashboards communautaires : importer par ID sur grafana.com.
