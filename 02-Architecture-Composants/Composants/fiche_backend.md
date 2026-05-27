---
title: "Le backend"
tags:
  - composants
  - backend
  - Node.js
  - API
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche_frontend]]
  - [[fiche_api]]
  - [[fiche_database]]
  - [[fiche-composants-backend]]
---

# ⚙️ Fiche Backend

## 🔎 Qu’est-ce que le Backend ?
Le **Backend** désigne la partie “cachée” d’une application :
- Gère la logique métier, les règles de sécurité et les données.
- Communique avec le Frontend via des API (REST, GraphQL…).
- Interagit avec bases de données et services externes.

---

## 🧱 Rôles principaux du Backend
- **Logique métier** : appliquer règles spécifiques (paiements, commandes, calculs).
- **Accès aux données** : lecture/écriture SQL/NoSQL.
- **Authentification & autorisations** : sécuriser l’accès aux ressources.
- **Communication externe** : emails, APIs tierces.
- **Performance & scalabilité**.

### 🖼️ Schéma Backend simple
![Backend simple](img/backend_simple.png)

---

## 🧩 Logique métier & règles de sécurité

### Logique métier
Ensemble des règles et processus propres au domaine.  
Exemples :
- E‑commerce : prix total, gestion stock.
- Banque : plafond virement, solde disponible.
- Réseau social : confidentialité posts, invitations.

### Règles de sécurité
- **Authentification** : identifier l’utilisateur (mot de passe hashé, OAuth2, MFA).
- **Autorisation** : définir ce que l’utilisateur peut faire (rôles, permissions).
- **Validation des données** : vérifier entrées (éviter injections, XSS).
- **Sécurité des communications** : TLS/HTTPS obligatoire.
- **Audit & traçabilité** : logs des actions sensibles.

📌 Exemple : virement bancaire  
1. Frontend demande transfert.  
2. Backend applique logique métier (solde, plafond).  
3. Applique sécurité (auth, autorisation, log).  
4. Répond.

### 🖼️ Schéma Backend logique métier
![Backend logique métier](img/backend_business.png)

### 🖼️ Schéma Backend sécurisé
![Backend sécurisé](img/backend_secure.png)

---

## ⚙️ Technologies courantes
- **Langages** : Node.js/TS (Express/Nest), PHP (Laravel, Symfony), Java (Spring Boot), Python (Django/FastAPI), C# (ASP.NET Core)
- **Bases de données** : MySQL, PostgreSQL, MongoDB, Redis
- **Outils** : ORM (TypeORM, Hibernate, Eloquent)

---

## 🏗️ Frameworks Backend
- **Express/NestJS** : APIs REST/GraphQL rapides
- **Laravel/Symfony** : web apps, back‑offices
- **Spring Boot** : standard entreprise, robuste
- **Django/FastAPI** : prototypage rapide

---

## 🔄 Communication Frontend ↔ Backend
- Frontend appelle l’API Backend
- Backend applique logique métier + sécurité
- Backend interagit avec DB et renvoie résultat

---

## 🧩 Exemples concrets
- **E‑commerce** : stock, commandes, facturation
- **Réseau social** : comptes, relations, messages
- **Application bancaire** : plafonds, transactions sécurisées

---

## 🛠️ Bonnes pratiques
- Architecture claire (MVC, DDD, Clean)
- Sécurité (validation, hashing, JWT, OAuth2)
- Tests unitaires/intégration
- Observabilité (logs, metrics)
- Scalabilité (load balancer, caching)

---

## 🧪 À vérifier après déploiement
- Disponibilité (tests curl/Postman)
- Sécurité (TLS, headers)
- Performance (stress test)
- Logs centralisés
- Connexion DB protégée

---

## 📊 Tableau récapitulatif

| Aspect | Objectif | Outils/Exemples | Points de vigilance |
|--------|----------|-----------------|---------------------|
| Logique métier | Appliquer règles du domaine | Services, contrôleurs | Tests, évolution |
| Accès aux données | Lire/écrire DB | ORM (Hibernate, TypeORM) | Performance, index |
| Communication | Exposer API | REST, GraphQL, gRPC | Auth, versioning |
| Sécurité | Protéger données | JWT, OAuth2 | Injections, CSRF |
| Scalabilité | Gérer la charge | Load balancer, Redis | Complexité infra |
