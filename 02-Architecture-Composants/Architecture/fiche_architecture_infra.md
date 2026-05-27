---
title: "Architecture infrastructure"
tags:
  - architecture
  - infra
  - cloud
  - serveurs
section: 02-Architecture-Composants
domaine: Architecture
statut: actif
liens_connexes:
  - [[fiche_architecture_systemes]]
  - [[intro_virtualisation]]
  - [[hebergement]]
---

# 🖥️ Architecture systèmes / infrastructure — Cours détaillé

## Sommaire
- [1. Serveur unique](#1-serveur-unique)
- [2. Virtualisation](#2-virtualisation)
- [3. Conteneurs](#3-conteneurs)
- [4. Orchestration (Kubernetes)](#4-orchestration-kubernetes)
- [5. Réseau & distribution (Edge → Core)](#5-réseau--distribution-edge--core)
- [6. CI/CD & Infrastructure as Code](#6-cicd--infrastructure-as-code)
- [7. Observabilité & SRE](#7-observabilité--sre)
- [8. Sécurité & conformité](#8-sécurité--conformité)
- [9. Coûts & FinOps](#9-coûts--finops)
- [📊 Tableau récapitulatif](#-📊tableau-récapitulatif)
- [🧭 Arbre de décision](#🧭-arbre-de-décision)

---

## 1. Serveur unique

**Idée** : tout tourne sur **une machine** (physique ou VM). Facile à administrer, parfait pour **débuter**.

![Serveur unique](img/serveur_unique.png)

### ✅ Quand utiliser
- Sites vitrine, MVP, intranet simple, démonstrateurs.
- Contraintes budget/temps fortes.

### ⚙️ Stacks typiques
- LAMP/LEMP (Apache/Nginx + PHP + MySQL).
- Node.js (Express) + MongoDB/PostgreSQL.
- Java (Tomcat/Spring Boot) + PostgreSQL.

### 👍 Avantages
- Simplicité opérationnelle, **coût réduit**, latence minimale (tout local).

### 👎 Limites / Anti‑patterns
- Pas de **haute dispo** ; **scalabilité** limitée (verticale).
- Risque de **single point of failure**.

---

## 2. Virtualisation

**Idée** : un serveur physique héberge **plusieurs VM isolées** avec des rôles distincts (API, DB, proxy…).

![Virtualisation](img/virtualisation.png)

### ✅ Quand utiliser
- Besoin d’**isolation forte** (sécurité, compatibilité OS).
- Mutualisation d’un serveur physique.

### ⚙️ Stacks typiques
- Hyperviseurs : VMware/ESXi, Hyper‑V, KVM, Proxmox.
- Outils : Packer (images), Ansible (provisioning).

### 👍 Avantages
- Isolation, snapshots, live migration.

### 👎 Limites / Anti‑patterns
- **Overhead** supérieur aux conteneurs ; densité moindre.
- “Zoo de VMs” sans automatisation.

---

## 3. Conteneurs

**Idée** : empaqueter appli + dépendances dans une **image** ; instances légères appelées **conteneurs**.

![Conteneurs](img/containers.png)

### ✅ Quand utiliser
- Déploiements **reproductibles**, portables (dev → prod).
- Multiples services sur une même machine.

### ⚙️ Stacks typiques
- Docker/Podman, Docker Compose pour dev/staging.
- Registries : GHCR, ECR, GCR, Docker Hub.

### 👍 Avantages
- Densité élevée, démarrage rapide, **parité dev/prod**.

### 👎 Limites / Anti‑patterns
- Orchestration nécessaire dès qu’on **scale**.
- Images **lourdes** si mal construites (multi‑stage builds recommandés).

📌 **12‑Factor App (extraits)** : config via variables d’environnement, stateless, logs en flux, dépendances explicites.

---

## 4. Orchestration (Kubernetes)

**Idée** : piloter **des dizaines/centaines** de conteneurs : scheduling, **autoscaling**, **auto‑heal**, mises à jour progressives.

![Orchestration](img/orchestration.png)

### ✅ Quand utiliser
- Plusieurs services à faire évoluer **en continu**.
- Besoin de **résilience**, de rollouts avancés (blue/green, canary).

### ⚙️ Stacks typiques
- Kubernetes (EKS/AKS/GKE/K3s), Helm/Kustomize.
- Ingress Controller (NGINX/Traefik), **HPA**, **PodDisruptionBudget**.
- Secrets/ConfigMaps, CSI (stockage), Jobs/CronJobs.
- **Service Mesh** (Istio/Linkerd) si besoins réseau avancés (mTLS, retries, circuit‑breaking).

### 👍 Avantages
- **Disponibilité** & **scalabilité** natives, standard de facto du cloud‑native.

### 👎 Limites / Anti‑patterns
- **Courbe d’apprentissage** élevée, coûts d’exploitation.
- “Kubernetes pour tout” : sur‑ingénierie pour petits projets.

---

## 5. Réseau & distribution (Edge → Core)

**Idée** : optimiser l’**accès mondial** et sécuriser la **frontière** avant d’atteindre vos services internes.

![Réseau & distribution](img/reseau_distribution.png)

### Composants clés
- **DNS/CDN** : proximité, cache d’assets, TLS terminé en edge.  
- **WAF / DDoS** : filtrage OWASP Top10, rate limiting.  
- **Load Balancer** : répartition et health checks.  
- **Reverse Proxy / API Gateway** : routage, auth, quotas, observabilité.  
- **Cache & DB** : données partagées (Redis), bases managées.

### Anti‑patterns
- Terminologie TLS **multipliée** sans besoin (double terminaison non maîtrisée).
- Règles **WAF** trop agressives → faux positifs.

---

## 6. CI/CD & Infrastructure as Code

**Idée** : automatiser build/tests/déploiements et **décrire l’infra en code** pour la rendre **reproductible**.

![CI/CD & IaC](img/cicd_iac.png)

### Pratiques
- **CI** : tests unitaires/intégration, build images, scans SAST/DAST.  
- **CD** : déploiements blue/green/canary, rollback.  
- **IaC** : Terraform/CDK/Pulumi, politique de revue (PR).

### Anti‑patterns
- Déploiements manuels, secrets commités, images “latest”.

---

## 7. Observabilité & SRE

- **Logs** structurés (JSON), **traces** (OpenTelemetry), **métriques** (Prometheus/Grafana).  
- **SLI/SLO** et **error budget** pour piloter la fiabilité.  
- Dashboards **actionnables** : latence P95, taux d’erreur, saturation.

---

## 8. Sécurité & conformité

- **Principe du moindre privilège**, **RBAC**, séparation des comptes.  
- **Gestion des secrets** (Vault, Secrets Manager).  
- **mTLS** (mesh), **CSP**/CORS côté edge.  
- **Conformité** : journaux immuables, rétention, chiffrement at‑rest/in‑transit.

---

## 9. Coûts & FinOps

- **Rightsizing** (CPU/RAM), **autoscaling**, **offloading** managé (DB, queues).  
- **Politiques de rétention** (logs, métriques), **sleep schedules** des envs non‑prod.  
- Étiquetage (tags) pour l’imputation des coûts.

---

## 📊 Tableau récapitulatif

| Choix infra | Quand ? | Avantages | Limites |
|---|---|---|---|
| Serveur unique | MVP, petit trafic | Très simple, peu cher | Pas de HA/scale |
| VM | Isolation forte | Snapshots, migration | Overhead vs conteneurs |
| Conteneurs | Déploiements modernes | Parité dev/prod, densité | Orchestration requise en scale |
| Kubernetes | Multi‑services, résilience | Autoscaling, rollouts | Complexité d’exploitation |
| Edge (CDN/WAF/LB) | Audience globale | Perf, sécurité | Gouvernance à mettre en place |
| CI/CD + IaC | Besoin d’itération rapide | Traçabilité, reproductibilité | Courbe d’apprentissage outils |

---

## 🧭 Arbre de décision

1) **MVP ≤ 3 mois**, équipe ≤ 3 → **Serveur unique / VM**.  
2) Plusieurs services, besoin de **releases fréquentes** → **Conteneurs**.  
3) Haute dispo et **scale dynamique** → **Kubernetes** managé.  
4) Audience globale → **CDN + WAF + LB**.  
5) Equipe qui itère vite → **CI/CD + IaC** dès le départ.

---
