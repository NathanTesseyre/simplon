---
title: "Sécurité pour développeurs"
tags:
  - sécurité
  - OWASP
  - XSS
  - SQLi
section: 03-Securite
domaine: Sécurité
statut: actif
liens_connexes:
  - [[fiche_auth_sessions]]
---

# 📘 Fiche : Sécurité pour développeurs  
*(à destination de développeurs en formation)*  

---

## 1. 🔒 Pourquoi la sécurité est essentielle ?  
Une application non sécurisée peut entraîner :  
- le vol de données (ex : mots de passe, infos personnelles),  
- la compromission d’un système (prise de contrôle par un attaquant),  
- une perte de confiance des utilisateurs.  

👉 La sécurité doit être pensée **dès le développement**, pas seulement à la fin.  

---

## 2. 🛑 Les principales menaces  

### Injection SQL  
- **Scope** : 🗄️ Backend (base de données, API).  
- **Problème** : l’application exécute directement du code SQL basé sur les entrées utilisateur.  
- **Risque** : un attaquant peut manipuler la requête pour voler ou supprimer des données.  
- **Solution** : requêtes préparées / paramétrées.  

---

### XSS (Cross-Site Scripting)  
- **Scope** : 🎨 Frontend (navigateur, rendu HTML).  
- **Problème** : du code JavaScript malveillant est injecté dans une page web.  
- **Risque** : vol de cookies, redirection vers des sites malveillants.  
- **Solution** : échapper les entrées utilisateur avant affichage.  

---

### Mauvaise gestion des mots de passe  
- **Scope** : 🔐 Backend (stockage des identifiants).  
- **Problème** : stockage en clair ou avec un hash faible.  
- **Risque** : fuite massive si la base est compromise.  
- **Solution** : hashage avec bcrypt, Argon2, salage.  

---

### Attaques réseau : HTTP non sécurisé & Man-In-The-Middle (MITM)  
- **Scope** : 🌍 Communications réseau (frontend ↔ backend, API, mobile ↔ serveur).  
- **Problème** :  
  - Si une application utilise HTTP au lieu de HTTPS, les données circulent en clair.  
  - Cela ouvre la porte aux attaques MITM où un attaquant intercepte et modifie le trafic.  
- **Exemple** : connexion à un Wi-Fi public sans HTTPS → un attaquant peut lire ou modifier les données transmises.  
- **Risque** : vol d’identifiants, injection de contenu malveillant, usurpation d’identité.  
- **Solution** :  
  - Utiliser systématiquement **HTTPS/TLS**.  
  - Vérifier les **certificats** (ne pas ignorer les alertes navigateur).  
  - Activer le **HSTS (HTTP Strict Transport Security)**.  

---

### Mauvaise gestion des erreurs  
- **Scope** : 🗄️ Backend (API, serveur), parfois frontend (messages exposés).  
- **Problème** : messages d’erreurs trop détaillés exposés aux utilisateurs.  
- **Risque** : fuite d’infos sur la base de données, les chemins du système.  
- **Solution** : messages génériques côté client, logs détaillés côté serveur.  

---

## 3. 🛡️ Bonnes pratiques de base  

- **Valider toutes les entrées utilisateur** (ne jamais faire confiance aux données reçues).  
- **Ne pas faire confiance au frontend, même quand on contrôle le code** :  
  - Un utilisateur malveillant peut contourner les validations côté client (JS désactivé, requêtes directes API).  
  - Toute donnée reçue par le backend doit être **vérifiée, validée et filtrée**.  
  - 👉 Ceinture + bretelles : validation côté frontend *et* côté backend.  
- **Principe du moindre privilège** : donner aux comptes et services uniquement les permissions nécessaires.  
- **Tenir ses dépendances à jour** (frameworks, bibliothèques).  
- **Chiffrer les données sensibles** au repos (disque) et en transit (réseau).  
- **Authentification et gestion de session sécurisées** (tokens, expiration, logout).  

---

## 4. 👨‍💻 Cas pratiques simples  

### Exemple en Python avec SQLAlchemy (sécurisé)  
```python
# Mauvais (vulnérable à l'injection)
cursor.execute("SELECT * FROM users WHERE username = '" + user_input + "'")

# Bon (requête paramétrée)
cursor.execute("SELECT * FROM users WHERE username = %s", (user_input,))
```  

### Exemple de hash sécurisé avec bcrypt (Node.js)  
```javascript
const bcrypt = require('bcrypt');
const hashed = await bcrypt.hash(password, 10); // 10 = cost factor
```  

---

## 5. ✅ À retenir  
- Les menaces principales : **Injection SQL, XSS, mots de passe faibles, attaques réseau (HTTP non sécurisé / MITM), erreurs mal gérées**.  
- Chaque menace touche un **scope différent** (backend, frontend, API, réseau).  
- Solutions : **requêtes paramétrées, échappement des entrées, hashage fort, HTTPS/TLS, gestion sécurisée des erreurs**.  
- **Règle d’or : ne jamais faire confiance aux données, pas même à celles venant de votre propre frontend.**  
- La sécurité = un **réflexe de développement**, pas un ajout optionnel.  

---

👉 Avec cette fiche, les étudiants comprennent les **attaques les plus courantes**, savent **où elles s’appliquent (backend, frontend, API, réseau)** et intègrent le principe fondamental de **“zéro confiance”** : valider et sécuriser partout.  
