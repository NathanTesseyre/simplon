---
title: "Architecture logicielle"
tags:
  - architecture
  - MVC
  - couches
  - patterns
section: 02-Architecture-Composants
domaine: Architecture
statut: actif
liens_connexes:
  - [[fiche_architecture_applicative]]
  - [[fiche_backend]]
---

# 🧭 Architecture logicielle — Cours détaillé

## Sommaire
- [1. Architecture en couches (n‑tiers)](#1-architecture-en-couches-ntiers)
- [2. MVC (Model–View–Controller)](#2-mvc-modelviewcontroller)
- [3. Architecture hexagonale (Ports & Adapters)](#3-architecture-hexagonale-ports--adapters)
- [4. SOA interne (monolithe modulaire)](#4-soa-interne-monolithe-modulaire)
- [📊 Tableau récapitulatif](#📊-tableau-récapitulatif)
- [🧪 Tests & Qualité](#🧪-tests--qualité)
- [🔍 Observabilité & Performance](#🔍-observabilité--performance)
- [🔐 Sécurité & gestion de la config](#🔐-sécurité--gestion-de-la-config)

---

## 1. Architecture en couches (n‑tiers)

**Idée** : séparer **présentation → contrôleurs → services → accès aux données** pour réduire le couplage et clarifier les responsabilités.

![Architecture en couches](img/architecture_layered.png)

### ✅ Quand utiliser
- Projets **web classiques** CRUD, back‑offices, APIs d’entreprise.  
- Équipes junior/mixte : conventions **simples et lisibles**.

### ⚙️ Mise en œuvre (exemples)
- **Back (Express.js)** : `routes/` → `controllers/` → `services/` → `repositories/` (Sequelize/TypeORM).  
- **Back (Spring Boot)** : `@RestController` → `@Service` → `@Repository` (Spring Data JPA).  
- **Front (React)** : composants UI → **services d’accès API** (fetch/axios) → **store** (Redux/Zustand).

### 👍 Avantages
- Lecture **prévisible**, séparation nette, **tests unitaires** faciles (mocker la couche du dessous).
- Bon tremplin vers **MVC** et **hexagonale**.

### 👎 Limites / Anti‑patterns
- **Anémie du domaine** : toute la logique dans les services, modèles passifs.  
- **Couchenite** : créer des couches “pour la forme” sans valeur.  
- Risque de **contournements** (un controller qui appelle directement la DB).

📌 **Principes associés**
- **SOLID** (SRP, OCP, DIP) : une responsabilité par classe ; dépendre d’**interfaces**.  
- **KISS/DRY** : éviter la duplication de logique entre contrôleurs et services.

---

## 2. MVC (Model–View–Controller)

**Idée** : 3 rôles distincts — **View** (affichage), **Controller** (flux), **Model** (données + règles/validations).

![MVC](img/architecture_mvc.png)

### ✅ Quand utiliser
- Apps **web** avec **templates** ou **composants UI** (SSR ou SPA).  
- Projets pédagogiques pour **inculquer la séparation des rôles**.

### ⚙️ Mise en œuvre (exemples)
- **Laravel/Symfony/Rails** : routes → controllers → views ; models (Eloquent/Doctrine/ActiveRecord).  
- **Spring MVC** : `@Controller`/`@RestController`, `@Model`, `@View`.  
- **Front** : composants (View), “controllers”/gestionnaires d’événements, “models”/store.

### 👍 Avantages
- Convention **largement connue**, structuration immédiate.  
- Bon support des **validations** côté Model, **DTO** entre couches.

### 👎 Limites / Anti‑patterns
- **God Controller** : contrôleur géant, mélange des responsabilités.  
- **Logique métier** fuyante entre Controller et Model.

📌 **Principes associés**  
- **Single Responsibility (SOLID)** : chaque composant a un rôle clair.  
- **KISS** : privilégier des controllers **fins** et lisibles.

---

## 3. Architecture hexagonale (Ports & Adapters)

**Idée** : le **domaine** (règles métier) ne dépend **d’aucune techno**. Les dépendances **pointent vers l’intérieur** ; tout accès externe passe par des **ports** (interfaces) implémentés par des **adapters**.

![Hexagonale](img/architecture_hexagonale.png)

### ✅ Quand utiliser
- Domaines **riches** et susceptibles d’évoluer (paiement, pricing, conformité).  
- Besoin de **tests métier** indépendants de la DB ou du framework web.

### ⚙️ Mise en œuvre (exemples)
- **Java/Spring** : modules `domain` (use cases, entités) ; `adapters` (REST, JPA, Kafka) ; `ports` (interfaces).  
- **Node.js** : dossiers `domain/` (use cases), `ports/` (interfaces TypeScript), `adapters/` (Postgres, REST).  
- **Front** : logique métier isolée (use cases), adapters pour fetch/storage/UI.

### 👍 Avantages
- **Testabilité** élevée (tests du domaine **sans** DB/HTTP).  
- **Substituabilité** : on peut changer la DB ou le transport **sans toucher** au métier.

### 👎 Limites / Anti‑patterns
- **Sur‑ingénierie** si le domaine est simple.  
- Multiplication des **interfaces** et du “plumbing”.

📌 **Principes associés**  
- **Dependency Inversion (DIP)** : le domaine dépend d’**abstractions**.  
- **Clean Architecture** (couches concentriques), **DDD** (Ubiquitous Language, Aggregates).

---

## 4. SOA interne (monolithe modulaire)

**Idée** : structurer un **monolithe** en **services métiers** bien délimités (modules), avec des **contrats internes** clairs.

![SOA interne](img/architecture_soa.png)

### ✅ Quand utiliser
- Gros monolithes nécessitant **modularité** et **responsabilités claires**.  
- Étape **intermédiaire** avant microservices (si besoin plus tard).

### ⚙️ Mise en œuvre (exemples)
- **Java** : modules Maven/Gradle par domaine ; interfaces publiques limitées.  
- **Node** : workspaces (pnpm/yarn) + modules internes versionnés.  
- **Front** : architecture par **domaines**/features, libraries partagées.

### 👍 Avantages
- **Encastrement** des responsabilités, meilleure **lisibilité**.  
- **Déploiement unique** conservé (simplicité ops).

### 👎 Limites / Anti‑patterns
- DB **trop partagée** ⇒ couplage caché.  
- “**Pseudo‑microservices**” sans bénéfice réel (ni indépendance ni scaling).

📌 **Principes associés**  
- **Bounded Contexts (DDD)** : frontières claires, contrats explicites.  
- **Conway’s Law** : aligner les modules sur l’organisation des équipes.

---

## 📊 Tableau récapitulatif

| Architecture | Quand choisir ? | Avantages | Limites |
|---|---|---|---|
| Couches | CRUD, apps classiques | Clarifie, facilite tests | Rigidité, risque d’anémie |
| MVC | Web/SSR/SPA | Rôles clairs, convention connu | God controllers, dispersion logique |
| Hexagonale | Domaine riche, long terme | Testable, techno‑agnostique | Sur‑ingénierie possible |
| SOA interne | Monolithe large | Modularité sans multi‑déploiements | Couplage DB, discipline requise |

---

## 🧪 Tests & Qualité

- **Pyramide des tests** : beaucoup d’**unitaires** (sur le domaine), des tests **d’intégration** ciblés, quelques **end‑to‑end**.  
- **TDD** utile sur hexagonale/couches.  
- **Contrats** : OpenAPI/JSON‑Schema, Pact pour consumer/provider.  
- **Analyse statique** : ESLint/TS, SonarQube, Checkstyle/PMD.

## 🔍 Observabilité & Performance

- Logs **structurés** (JSON), **correlation IDs**.  
- **Métriques** (Prometheus), **profiling** (APM : OpenTelemetry, Jaeger/Tempo).  
- **Budgets de performance** côté front (TTI, LCP), caches et pagination côté back.

## 🔐 Sécurité & gestion de la config

- **Principes SOLID** appliqués aux services d’auth/autz (séparation responsabilités).  
- **Secrets** : vaults (HashiCorp, AWS Secrets Manager).  
- **Validation** systématique des entrées ; **sanitization** ; **rate limiting** ; **RBAC** interne.  
- **KISS/DRY** : éviter les duplications de validations et de logique d’accès.
