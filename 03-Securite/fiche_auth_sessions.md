---
title: "Authentification et sessions"
tags:
  - sécurité
  - auth
  - JWT
  - cookies
  - MFA
section: 03-Securite
domaine: Sécurité
statut: actif
liens_connexes:
  - [[fiche_droits_autorisations]]
  - [[fiche_stockage_donnees]]
---

# 📘 Fiche : Authentification et gestion des sessions  
*(à destination de développeurs en formation)*  

---

## 1. 🔒 Pourquoi c’est important ?  
- L’authentification et la gestion des sessions sont au cœur de la sécurité applicative.  
- Si elles sont mal implémentées :  
  - un attaquant peut usurper l’identité d’un utilisateur,  
  - accéder à des données sensibles,  
  - ou prendre le contrôle d’un compte administrateur.  

👉 C’est une des **failles les plus exploitées** dans les applications web.  

---

## 2. 👤 Authentification  

### Méthodes classiques  
- **Login / mot de passe** :  
  - ⚠️ Ne jamais stocker les mots de passe en clair.  
  - Toujours les hasher avec **bcrypt, Argon2, scrypt**.  
- **Multi-Factor Authentication (MFA)** : ajouter une étape (code SMS, app d’authentification).  
- **OAuth 2.0 / OpenID Connect** : délégation via Google, GitHub, etc.  
- **SSO (Single Sign-On)** : un seul compte pour accéder à plusieurs services.  

---

## 3. 🗝️ Sessions  

### Comment ça marche ?  
- Lorsqu’un utilisateur s’authentifie (login/mot de passe, OAuth…), le **serveur crée une session**.  
- Cette session est identifiée par un **ID de session** (aléatoire et sécurisé).  
- L’ID est envoyé au client (via un cookie ou un header).  
- À chaque nouvelle requête, le client **renvoie l’ID** → le serveur retrouve l’utilisateur associé.  

---

### Cookies de session  
- Stockés côté client dans un cookie géré automatiquement par le navigateur.  
- Le serveur garde en mémoire les données de session (utilisateur connecté, rôles, préférences…).  

⚙️ **En tant que développeur** :  
- Utiliser une **lib ou framework adaptée au langage** :  
  - **JavaScript (Node.js)** → `express-session` (Express), `cookie-session`, `next-auth` (Next.js).  
  - **Java** → Spring Security (Spring Boot), `HttpSession` (servlets), Jakarta EE Security.  
  - **PHP** → gestion native avec `$_SESSION`, ou frameworks comme Symfony (`SessionInterface`) et Laravel (`Session`).  
- Toujours configurer les cookies avec :  
  - `HttpOnly` (empêche l’accès au cookie par du JS malveillant),  
  - `Secure` (uniquement via HTTPS),  
  - `SameSite` (empêche les attaques CSRF).  
- Régénérer l’ID de session après login pour limiter les attaques de fixation de session.  

---

### Tokens (JWT, OAuth)  
- Un **token signé** contient directement les infos de l’utilisateur (claims : id, rôle, expiration).  
- Pas besoin de stockage côté serveur → pratique pour les **APIs REST** et le **microservices**.  

⚙️ **En tant que développeur** :  
- Générer un JWT signé avec une clé secrète ou un certificat.  
- Définir une durée d’expiration courte.  
- Stocker le token côté client (dans `Authorization: Bearer ...`), pas dans `localStorage` si possible (risque XSS).  
- Vérifier systématiquement le token côté backend.  

---

### Cookies vs Tokens : quand utiliser quoi ?  
- **Cookies de session** :  
  - Idéal pour les applis web classiques.  
  - Facile à sécuriser avec `HttpOnly` et `Secure`.  
- **Tokens (JWT)** :  
  - Idéal pour les **APIs REST** et les applis mobiles.  
  - Plus complexe à gérer (expiration, révocation).  

---

## 4. ⚠️ Menaces courantes  

- **Vol de session (Session Hijacking)** :  
  - Interception d’un cookie (si pas en HTTPS).  
  - Solution : cookies sécurisés + HTTPS obligatoire.  

- **Fixation de session (Session Fixation)** :  
  - Attaquant force un utilisateur à utiliser un cookie connu.  
  - Solution : régénérer la session après login.  

- **Brute force sur mots de passe** :  
  - Attaques automatisées.  
  - Solution : limitation de tentatives, CAPTCHA, MFA.  

---

## 5. ✅ Bonnes pratiques  

- Toujours utiliser **HTTPS**.  
- Ne jamais stocker les mots de passe en clair.  
- Utiliser des **cookies sécurisés** (HttpOnly, Secure, SameSite).  
- Régénérer les sessions après connexion.  
- Mettre en place la **MFA**.  
- Déconnecter automatiquement les sessions inactives.  
- En cas de doute : **invalider toutes les sessions** (compte compromis).  

---

## 6. 👨‍💻 Exemple pratique : Cookie sécurisé en Express.js  

```javascript
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: true,     // seulement via HTTPS
    sameSite: 'strict',
    maxAge: 60000     // 1 minute
  }
}));
```  

---

## 7. ✅ À retenir  
- L’authentification est la **porte d’entrée** → si elle est faible, tout le reste est vulnérable.  
- Les **sessions** permettent d’identifier un utilisateur après login.  
- **Cookies de session** → simples et sécurisés pour les applis web.  
- **Tokens (JWT)** → adaptés aux APIs et aux applis distribuées.  
- Toujours sécuriser l’implémentation : HTTPS, cookies sécurisés, expiration, validation stricte.  

---

👉 Avec cette fiche, les étudiants comprennent non seulement **les risques liés aux sessions**, mais aussi **comment les utiliser concrètement** dans leurs applis avec des libs/frameworks adaptés (JS, Java, PHP).  
