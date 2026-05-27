---
title: "Architecture applicative"
tags:
  - architecture
  - frontend
  - backend
  - API
section: 02-Architecture-Composants
domaine: Architecture
statut: actif
liens_connexes:
  - [[fiche_architecture_logicielle]]
  - [[fiche_architecture_infra]]
  - [[fiche_backend]]
  - [[fiche_frontend]]
---

# 🌐 Architectures applicatives — Frontend/Backend intégré

## Sommaire
- [1. Monolithique](#1-monolithique)
- [2. Client–Serveur](#2-client-serveur)
- [3. Microservices](#3-microservices)
- [4. Serverless](#4-serverless)
- [5. Orientée événements](#5-orientée-événements)
- [📊 Tableau récapitulatif](#📊-tableau-récapitulatif)

## 1. Monolithique

### 💡 Idée générale
Tout est regroupé dans un même déploiement : **Frontend (UI)** et **Backend (logique + DB)**.  
Le serveur web (Apache/Nginx ou embarqué comme Tomcat) sert statics et exécute/proxifie le code.

![Monolithique](img/monolithique_ws_fb.png)

### ✅ Quand utiliser
- MVP, intranets, petites applis.  
- Time‑to‑market prioritaire.

### ⚙️ Stacks typiques
- PHP + MySQL/PostgreSQL via Apache/Nginx.  
- Java/Spring Boot (Tomcat).  
- Node/Express.

### 📡 Rôle du serveur web
- HTTPS, statics, proxy/exécution.  
- Sécurité headers, logs, cache simple.

### 🧩 Exemples concrets
- WordPress/Drupal sur Apache/Nginx.  
- ERP interne monolithique.  
- Back‑office Laravel.

---

## 2. Client–Serveur

### 💡 Idée générale
Le **Frontend** (SPA React/Angular/Vue, mobile) consomme une **API Backend**.  
Le serveur web sert le Frontend et proxifie `/api/*` vers le Backend.

![Client–Serveur](img/client_serveur_ws_fb.png)

### ✅ Quand utiliser
- Apps web classiques, séparation UI/API.  
- Contrats d’API stables.

### ⚙️ Stacks typiques
- Front React/Angular/Vue.  
- Back Express.js, Laravel, Spring.

### 📡 Rôle du serveur web
- Sert UI + proxy API.  
- TLS, CORS, cache, rate‑limit.

### 🧩 Exemples concrets
- Appli bancaire mobile + API.  
- E‑commerce React/Next + Laravel.  
- SaaS Angular + Node.js.

---

## 3. Microservices

### 💡 Idée générale
Le **Frontend** consomme une **API Gateway** qui route vers des **services Backend** spécialisés avec DB séparée.

![Microservices](img/microservices_ws_fb.png)

### ✅ Quand utiliser
- Domaines clairs, équipes multiples.  
- Scalabilité fine, résilience.

### ⚙️ Stacks typiques
- Services Node/Spring/Go.  
- Gateway Nginx/Kong/Traefik.

### 📡 Rôle du serveur web
- TLS, routage endpoints.  
- Sécurité (auth, quotas, CORS).  
- Observabilité (logs/metrics).

### 🧩 Exemples concrets
- Marketplace : catalogue, panier, commande, livraison.  
- Plateforme B2B : auth, billing, notifications.  
- Scaling ciblé service recherche.

---

## 4. Serverless

### 💡 Idée générale
Le **Frontend** appelle une **API Gateway** qui déclenche des **fonctions Backend**.

![Serverless](img/serverless_ws_fb.png)

### ✅ Quand utiliser
- Workloads irréguliers, automatisations.  
- APIs légères, coût à l’usage.

### ⚙️ Stacks typiques
- Fonctions Lambda/Cloud Functions.  
- DB managées Dynamo/Firestore.

### 📡 Rôle du serveur web
- TLS managé, routage, quotas, WAF.  
- Transformation requêtes, cache edge.

### 🧩 Exemples concrets
- Chatbot Slack/Teams.  
- Webhooks Git déclenchant CI/CD.  
- Traitement images/PDF.

---

## 5. Orientée événements

### 💡 Idée générale
Les **Frontend/producteurs** publient via HTTP → Gateway → Bus.  
Des **consommateurs Backend** s’abonnent et réagissent.

![Event‑driven](img/event_driven_ws_fb.png)

### ✅ Quand utiliser
- Temps réel, intégrations découplées.

### ⚙️ Stacks typiques
- Producteurs/consommateurs Node/Java/Go.  
- Kafka/RabbitMQ.

### 📡 Rôle du serveur web
- HTTPS, validation, routage → bus.  
- Gère CORS, auth, rate‑limit.

### 🧩 Exemples concrets
- Paiement en ligne (Stripe events).  
- IoT : capteurs → gateway → bus.  
- Tracking clics/logs.

---

## 📊 Tableau récapitulatif

| Architecture | Quand choisir ? | Avantages | Limites |
|---|---|---|---|
| Monolithique | MVP, petites applis | Simple, peu coûteux | Peu scalable, couplage |
| Client–Serveur | UI/API séparés | Maintenabilité, réutilisation API | CORS, besoin coordination |
| Microservices | Domaines clairs, équipes multiples | Scalabilité fine, résilience | Complexité, observabilité |
| Serverless | Workloads irréguliers | Coût à l’usage, ops minimes | Cold starts, lock‑in |
| Événements | Temps réel, intégrations | Découplage, extensible | Cohérence/traçage difficiles |
