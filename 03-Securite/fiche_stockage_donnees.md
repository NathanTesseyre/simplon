---
title: "Stockage sécurisé des données"
tags:
  - sécurité
  - hashage
  - chiffrement
  - credentials
section: 03-Securite
domaine: Sécurité
statut: actif
liens_connexes:
  - [[fiche_auth_sessions]]
  - [[fiche_database]]
---

# 📘 Fiche : Bonnes pratiques de stockage des données sensibles  
*(à destination de développeurs en formation)*  

---

## 1. 🔒 Pourquoi c’est important ?  
- Les applications manipulent souvent des **données sensibles** : mots de passe, clés API, numéros de carte bancaire, données personnelles (RGPD).  
- Un mauvais stockage peut entraîner :  
  - Fuite de données utilisateurs (ex. mots de passe en clair).  
  - Compromission de l’application (ex. clé API exposée).  
  - Sanctions légales (RGPD, PCI-DSS).  

👉 La **sécurité des données sensibles** est une obligation pour tout développeur.  

---

## 2. 🧩 Quelles données sont sensibles ?  
- **Mots de passe** des utilisateurs.  
- **Clés API** et **tokens d’accès**.  
- **Informations personnelles** (emails, adresses, numéros de téléphone).  
- **Informations financières** (numéros de carte bancaire).  
- **Secrets d’infrastructure** (certificats, variables d’environnement).  

---

## 3. 🔑 Stockage des mots de passe  

- ❌ **Jamais en clair** dans la base.  
- ✅ Toujours stocker des **hashs sécurisés** avec :  
  - **bcrypt**, **Argon2** ou **scrypt**.  
  - ⚠️ Éviter SHA-256 seul → trop rapide → vulnérable au brute force.  
- ✅ Ajouter un **sel aléatoire** pour chaque mot de passe → rend les rainbow tables inutiles.  

Exemple avec bcrypt en Node.js :  
```javascript
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12); // 12 = facteur de coût
```  

---

## 4. 🗝️ Stockage des clés et secrets  

### ⚙️ En tant que développeur : comment gérer les credentials ?  
- **Ne jamais hardcoder** un mot de passe, une clé API ou un token directement dans le code.  
  - Mauvaise pratique ❌ :  
    ```javascript
    const dbPassword = "12345";
    ```  
- **Utiliser les variables d’environnement** pour injecter les credentials au runtime.  
  - Bonne pratique ✅ :  
    ```javascript
    const dbPassword = process.env.DB_PASSWORD;
    ```  
- **Ne jamais versionner son fichier `.env`** → l’ajouter au `.gitignore`.  
- **Séparer les environnements** (développement, test, production) avec des credentials différents.  
- **Renouveler régulièrement** les credentials (tokens expirables, rotation de clés).  
- **Limiter les droits** : une clé API ne doit donner accès qu’à ce qui est strictement nécessaire.  

### Gestionnaires de secrets recommandés  
- **Vault (HashiCorp)**, **AWS Secrets Manager**, **GCP Secret Manager**, **Azure Key Vault**.  
👉 Ces outils permettent de stocker les secrets de manière centralisée, chiffrée et avec un contrôle fin des accès.  

---

## 5. 🔐 Chiffrement des données sensibles  

- Les données sensibles stockées en base doivent être **chiffrées**.  
- Deux niveaux de chiffrement :  
  - **Au repos (at rest)** → chiffrement du disque ou de la base.  
  - **En transit** → toujours utiliser **HTTPS/TLS**.  
- Exemple : chiffrer un numéro de carte bancaire avec AES (clé secrète stockée dans un gestionnaire de secrets).  

---

## 6. ⚠️ Erreurs courantes à éviter  
- Stocker les mots de passe ou clés en clair dans une base ou un fichier.  
- Laisser des **credentials dans le code source** (GitHub, GitLab).  
- Réutiliser le même mot de passe ou token pour plusieurs services.  
- Utiliser des clés API permanentes au lieu de tokens temporaires.  
- Ne pas appliquer de rotation régulière sur les clés et certificats.  

---

## 7. ✅ Bonnes pratiques  
- Toujours hasher les mots de passe avec bcrypt/Argon2/scrypt.  
- Stocker les credentials dans des **variables d’environnement** ou des **gestionnaires de secrets**.  
- Ne jamais partager un `.env` ou un mot de passe en clair (préférer des vaults ou gestionnaires d’équipe).  
- Mettre en place une **rotation régulière** des clés API et tokens.  
- Utiliser des outils de scan (ex : **GitLeaks**) pour détecter des secrets exposés dans le code.  
- Documenter les bonnes pratiques dans l’équipe (ex : “aucun secret en clair dans le code”).  

---

## 8. ✅ À retenir  
- **Mots de passe** → hash + sel (bcrypt, Argon2, scrypt).  
- **Credentials (clés API, tokens, BDD)** → toujours via variables d’environnement ou gestionnaires de secrets.  
- **Jamais de secrets dans le code source** → même en dev.  
- **Rotation régulière** et **moindre privilège** pour limiter les impacts en cas de fuite.  
- **Scanner et auditer** régulièrement le dépôt pour éviter les fuites accidentelles.  

---

👉 Avec cette fiche, les étudiants comprennent comment **gérer concrètement leurs credentials** en tant que développeurs, éviter les erreurs classiques (hardcoding, secrets versionnés) et mettre en place des pratiques professionnelles (variables d’environnement, vaults, rotation).  
