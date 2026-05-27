---
title: "Haute disponibilité — concepts et patterns"
tags:
  - fondamentaux
  - infrastructure
  - haute-disponibilité
  - DevOps
section: 01-Fondamentaux
domaine: Infrastructure
statut: actif
liens_connexes:
  - [[fiche_adressage_ip]]
  - [[fiche_architecture_infra]]
  - [[fiche_stockage]]
  - [[cloud_aws_fondamentaux]]
---

# 🔁 Fiche : Haute disponibilité — concepts et patterns
*(à destination d'administrateurs système en formation)*

---

## 1. Pourquoi la haute disponibilité ?

Un service **non disponible coûte** : perte de revenus, dégradation de la réputation, impact sur les utilisateurs.  
La **haute disponibilité (HA)** vise à maintenir un service opérationnel même en cas de panne d'un composant.

### SLI / SLO / SLA

| Terme | Définition | Exemple |
|---|---|---|
| **SLI** (Service Level Indicator) | Mesure concrète de la disponibilité | Taux de requêtes HTTP 2xx |
| **SLO** (Service Level Objective) | Objectif interne | 99,9% de disponibilité sur 30 jours |
| **SLA** (Service Level Agreement) | Engagement contractuel avec pénalités | 99,5% garanti, remboursement si dépassé |

### Tableau de disponibilité (uptime)

| SLO | Indisponibilité / an | Indisponibilité / mois |
|---|---|---|
| 99% | 3 j 15 h | 7 h 18 min |
| 99,9% (three nines) | 8 h 45 min | 43 min |
| 99,95% | 4 h 22 min | 21 min |
| 99,99% (four nines) | 52 min | 4 min |
| 99,999% (five nines) | 5 min | 26 sec |

> **Five nines** (99,999%) est l'objectif des opérateurs télécom critiques. Pour la plupart des services web, **99,9%** est un objectif raisonnable.

---

## 2. Concepts fondamentaux

### SPOF (Single Point of Failure)
Un **SPOF** est un composant dont la panne entraîne l'indisponibilité totale du service.  
L'objectif HA est d'**éliminer tous les SPOF** par la redondance.

```
❌ Architecture avec SPOF :
[Users] → [Load Balancer unique] → [Serveur unique] → [BDD unique]
              SPOF !                   SPOF !              SPOF !

✅ Architecture HA :
[Users] → [LB1]   [LB2] → [Web1] [Web2] [Web3] → [BDD Primary]
           └─── VIP ───┘                              └─ [BDD Replica]
```

### Redondance et réplication
| Terme | Signification |
|---|---|
| **Actif/Passif** | Un seul nœud actif, le second prend le relais en cas de panne (failover) |
| **Actif/Actif** | Plusieurs nœuds actifs simultanément, la charge est répartie |
| **Réplication** | Copie des données en temps réel vers un ou plusieurs nœuds |
| **Failover** | Basculement automatique vers un nœud de secours |
| **Failback** | Retour sur le nœud principal après sa remise en service |

### RTO et RPO
| Terme | Définition | Question |
|---|---|---|
| **RTO** (Recovery Time Objective) | Durée max d'indisponibilité acceptable | "En combien de temps doit-on être de retour ?" |
| **RPO** (Recovery Point Objective) | Perte de données maximale acceptable | "Jusqu'où peut-on remonter dans le temps ?" |

```
Incident        Reprise
   │               │
───┼───────────────┼───────→ temps
   │←─── RTO ────→│
   │←RPO→│
   Dernier backup
```

---

## 3. Load Balancing

Le **load balancer** répartit le trafic entre plusieurs serveurs backend.

### Algorithmes de répartition

| Algorithme | Description | Usage |
|---|---|---|
| **Round Robin** | Rotation séquentielle entre les serveurs | Serveurs identiques, requêtes courtes |
| **Weighted Round Robin** | Round Robin avec des poids | Serveurs de capacités différentes |
| **Least Connections** | Envoie vers le serveur le moins chargé | Sessions longues (WebSocket, BDD) |
| **IP Hash** | Même IP client → même serveur | Sessions sans état partagé |
| **Random** | Aléatoire | Simple, efficace à grande échelle |

### Couches OSI

| Type | Couche | Exemples |
|---|---|---|
| **L4 (Transport)** | TCP/UDP | HAProxy, AWS NLB |
| **L7 (Application)** | HTTP/HTTPS | Nginx, HAProxy, AWS ALB, Traefik |

> L7 est plus intelligent (routing par URL, headers, cookies) mais plus coûteux en CPU.

### Health checks
Le LB doit savoir si un backend est opérationnel :
```
LB → GET /health → Backend
        ↓
   200 OK = sain → reçoit du trafic
   5xx / timeout = défaillant → retiré de la rotation
```

```bash
# Exemple de health check Nginx (upstream)
upstream backend {
    server web1:80;
    server web2:80;
    keepalive 32;
}

# Exemple HAProxy
backend web_servers
    balance roundrobin
    option httpchk GET /health
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
```

---

## 4. Clustering

Un **cluster** est un groupe de serveurs qui se comportent comme une unité unique.

### Cluster actif/passif (heartbeat)
```
[Node1 — ACTIF]  ←──heartbeat──→  [Node2 — PASSIF]
      │
  [VIP : 192.168.1.100]   ← IP virtuelle flottante

Si Node1 tombe → Node2 prend la VIP → le service continue
```

**Outils Linux** : Pacemaker + Corosync, Keepalived (VRRP).

```bash
# Keepalived — exemple de config VRRP
# /etc/keepalived/keepalived.conf sur le master
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    virtual_ipaddress {
        192.168.1.100/24
    }
}
```

### Cluster actif/actif
Les deux nœuds servent du trafic simultanément. Nécessite un **stockage partagé** ou une **réplication des données**.

---

## 5. Réplication de base de données

### Master/Replica (lecture/écriture séparées)
```
Écritures → [Primary / Master]
                    │ réplication binlog
                    ▼
           [Replica 1] [Replica 2]  ← Lectures
```

| Mode | Description | Risque |
|---|---|---|
| **Asynchrone** | Replica peut être en retard | Perte de données en cas de bascule |
| **Synchrone** | Écriture validée sur tous les nœuds | Latence accrue |
| **Semi-synchrone** | Validé sur au moins un replica | Compromis |

### Multi-Primary (actif/actif)
Les deux nœuds acceptent des écritures. Complexe à gérer (conflits). Exemples : Galera Cluster (MySQL), Patroni (PostgreSQL).

---

## 6. Zones de disponibilité et régions

Les cloud providers (AWS, Azure, GCP) structurent leur infrastructure en :

```
Région (ex: eu-west-1 Paris)
   ├── Zone A (eu-west-1a)  — datacenter A
   ├── Zone B (eu-west-1b)  — datacenter B
   └── Zone C (eu-west-1c)  — datacenter C
```

| Déploiement | Protection contre | Coût |
|---|---|---|
| **Single AZ** | Pannes serveur uniquement | Faible |
| **Multi-AZ** | Panne d'un datacenter entier | Moyen |
| **Multi-Region** | Catastrophe régionale, latence | Élevé |

---

## 7. Patterns de déploiement HA

### Blue/Green
```
Actuellement actif     En préparation
  [Blue v1.0]   →   [Green v1.1]  (tests)
       │
   [Load Balancer]
       │
   Basculement instantané : LB redirige vers Green
       │
  [Green v1.1]  ←  Blue conservé pour rollback rapide
```

### Canary Release
```
[LB] → 95% du trafic → [v1.0 stable]
     →  5% du trafic → [v1.1 canary]  (surveillance des erreurs)
```
Si v1.1 est stable → augmenter progressivement jusqu'à 100%.

### Rolling Update
```
[Web1 v1.0] [Web2 v1.0] [Web3 v1.0]
     ↓
[Web1 v1.1] [Web2 v1.0] [Web3 v1.0]   ← Web1 mis à jour
[Web1 v1.1] [Web2 v1.1] [Web3 v1.0]   ← Web2 mis à jour
[Web1 v1.1] [Web2 v1.1] [Web3 v1.1]   ← Web3 mis à jour
```

---

## 8. À retenir pour un admin sys

- **Identifier les SPOF** est la première étape de tout audit HA.
- **RTO** = durée max de panne acceptable, **RPO** = perte de données max acceptable.
- Le load balancer doit faire des **health checks** pour retirer automatiquement les nœuds défaillants.
- RAID protège contre la **panne disque**, HA protège contre la **panne serveur**, Multi-AZ protège contre la **panne datacenter**.
- **Blue/Green** permet un rollback instantané ; **canary** permet de tester en production avec un risque limité.
