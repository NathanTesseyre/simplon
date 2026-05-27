---
title: "Les API"
tags:
  - composants
  - API
  - REST
  - HTTP
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche_backend]]
  - [[fiche_reseaux]]
  - [[fiche_serveurs_web]]
---

# 🔌 Fiche API (Application Programming Interface)

## 🔎 Qu’est-ce qu’une API ?
Une **API** est une **interface** qui permet à deux logiciels de communiquer.
C’est un **contrat** (endpoints, formats, erreurs) qui définit *comment* accéder à des fonctionnalités ou des données.

**Exemples :**
- Une app mobile récupère les messages d’un utilisateur via l’API du serveur.
- Un site e-commerce déclenche un paiement via l’API d’un prestataire.
- Une appli météo lit les prévisions via l’API d’un service externe.

---

## 🧱 Rôles principaux d’une API
- **Communication** entre systèmes hétérogènes (JS, PHP, Java, Python…).
- **Abstraction** : expose le nécessaire, cache l’implémentation.
- **Interopérabilité** : formats communs (JSON/HTTP).
- **Réutilisation** : un même service pour plusieurs apps (web, mobile, partenaires).
- **Sécurité** : contrôle d’accès, quotas, journalisation.

---

## ⚙️ Types d’API (panorama)
- **REST** (*Representational State Transfer*) — le standard de facto sur HTTP.
- **GraphQL** — requêtes “sur mesure” (un seul endpoint).
- **gRPC / RPC** — binaire, très performant, souvent entre microservices.
- **SOAP** — XML, très normé, encore présent en entreprise (legacy).

---

## 🌍 Focus : API REST

### 🔎 Principes essentiels
- **HTTP** comme protocole transport.
- **Ressources** identifiées par des **URI** (`/users`, `/orders/123`).
- **Méthodes HTTP** : `GET` (lire), `POST` (créer), `PUT/PATCH` (modifier), `DELETE` (supprimer).
- **Représentations** : JSON (le plus courant).
- **Sans état (stateless)** : chaque requête est autonome (token inclus).

### 🧭 Design d’endpoints (ex. “utilisateurs”)
| Action               | Endpoint        | Méthode |
|-----------------------|-----------------|---------|
| Lire la liste         | `/users`        | GET     |
| Lire un utilisateur   | `/users/{id}`   | GET     |
| Créer                 | `/users`        | POST    |
| Mettre à jour         | `/users/{id}`   | PUT/PATCH |
| Supprimer             | `/users/{id}`   | DELETE  |

### ✅ Avantages REST
- Simple, lisible, basé sur HTTP.
- Écosystème énorme (outils, docs, middleware).
- Indépendant du langage → interopérable.
- Testable via **curl**, **Postman**, **tests auto**.

### ⚠️ Limites REST
- Risque d’**over/under-fetching** (trop ou pas assez de données).
- Plusieurs requêtes côté front pour composer un écran.
- Pas “temps réel” nativement (compléter avec **WebSockets**).

---

## 📡 Comment appeler une API ?

### Avec `curl` (ligne de commande)
```bash
# Récupérer tous les utilisateurs
curl -X GET https://api.exemple.com/users

# Créer un utilisateur (POST JSON)
curl -X POST https://api.exemple.com/users   -H "Content-Type: application/json"   -d '{"name":"Alice","email":"alice@example.com"}'

# Appel authentifié (Bearer JWT)
curl -X GET https://api.exemple.com/orders   -H "Authorization: Bearer VOTRE_JWT"
```

### Avec **JavaScript** (Frontend React/Vanilla JS)
```javascript
// Exemple d'appel GET à une API REST
fetch("https://api.exemple.com/users")
  .then(res => res.json()) // Conversion en JSON
  .then(data => console.log("Liste des utilisateurs:", data))
  .catch(err => console.error("Erreur lors de l'appel API:", err));

// Exemple d'appel POST pour créer un utilisateur
fetch("https://api.exemple.com/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice", email: "alice@example.com" })
})
  .then(res => res.json())
  .then(created => console.log("Utilisateur créé:", created))
  .catch(err => console.error("Erreur lors de la création:", err));
```

### Avec **Postman** (outil graphique)
- Configurer **URL** + méthode (GET/POST/PUT/DELETE).
- Ajouter les **headers** (`Content-Type`, `Authorization`).
- Fournir le **body** (JSON pour POST/PUT).
- Sauvegarder dans une **Collection**, partager avec l’équipe.

---

### 📦 Comment envoyer des données à une API ?
1. **Dans le corps (body)** — pour `POST` ou `PUT`  
   - Format courant : **JSON**  
   - Header requis : `Content-Type: application/json`

```bash
curl -X POST https://api.exemple.com/users   -H "Content-Type: application/json"   -d '{"name": "Alice", "email": "alice@example.com"}'
```

2. **Dans l’URL (query string)** — pour filtres, recherches, pagination
```bash
curl -X GET "https://api.exemple.com/users?page=2&limit=20&sort=-createdAt"
```

3. **Dans les headers** — pour authentification ou métadonnées
```bash
curl -X GET https://api.exemple.com/orders   -H "Authorization: Bearer VOTRE_JWT"
```

👉 **Résumé** : Body JSON → création/modification ; Query → filtres ; Headers → sécurité/format.

---

## 📨 Les headers HTTP dans une API REST
### 🔑 Côté requête (client → serveur)
- `Content-Type` : format envoyé (`application/json`)
- `Accept` : format attendu (`application/json`)
- `Authorization` : `Bearer <JWT>`

```bash
curl -X POST https://api.exemple.com/users   -H "Content-Type: application/json"   -H "Accept: application/json"   -H "Authorization: Bearer VOTRE_JWT"   -d '{"name":"Alice"}'
```

### 📤 Côté réponse (serveur → client)
- `Content-Type` : format renvoyé (JSON, HTML…)
- `Cache-Control` : directives de cache
- `Access-Control-Allow-Origin` : CORS
- `Retry-After` : délai avant nouvel essai (`429`)

---

## 🔐 Sécuriser une API (REST)
1) **AuthN/Z** : HTTPS, tokens **JWT**, rôles/scopes.  
2) **Validation** : contrôler entrées, prévenir les injections, limiter la taille.  
3) **Protection** : rate limiting, quotas, surveiller `401/403/429`.  
4) **HTTP** : codes cohérents, CORS restreints, erreurs sans fuite d’info.  
5) **Checks** : endpoints protégés, TLS actif, pas de tokens en logs.

---

## 🛠️ Bonnes pratiques (REST)
- **Nommage clair** : `/users`, `/orders/{id}`
- **Versioning** : `/api/v1/...`
- **Filtres, pagination, tri** : `?page=2&limit=20`
- **Erreurs uniformes** : `{ "error": "Not Found" }`
- **Documentation vivante** : OpenAPI/Swagger
- **Observabilité** : logs, monitoring

---

## 🖼️ Schéma — Frontend → API REST → Backend/DB
![Flux API REST](img/api_rest_flow.png)

---

## 🧪 À vérifier après déploiement
- Endpoints testés (`curl`, Postman)
- TLS actif, permissions respectées
- Latence & débit corrects
- Pas de données sensibles dans les logs
- Documentation Swagger à jour

---

## 📊 Tableau récapitulatif
| Aspect         | Objectif               | Outils/Exemples             | Points de vigilance |
|----------------|------------------------|-----------------------------|---------------------|
| Ressources     | Modéliser le domaine   | `/users`, `/orders/{id}`    | Nommage clair       |
| Méthodes HTTP  | Standardiser actions   | GET/POST/PUT/DELETE         | Idempotence         |
| Données        | Format d’échange       | JSON                        | Validation stricte  |
| Sécurité       | Protéger l’API         | JWT, OAuth2, HTTPS          | Rate limiting, CORS |
| Erreurs        | Réponses cohérentes    | Codes HTTP + JSON           | Pas de fuite info   |
| Documentation  | Adoption               | Swagger / OpenAPI           | À jour              |
| Versioning     | Gérer évolutions       | `/api/v1/...`               | Compat ascendante   |
