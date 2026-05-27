---
title: "Droits et autorisations"
tags:
  - sécurité
  - RBAC
  - ABAC
  - IAM
section: 03-Securite
domaine: Sécurité
statut: actif
liens_connexes:
  - [[fiche_auth_sessions]]
  - [[utilisateurs_permissions]]
---

# 📘 Fiche : Gestion des droits et autorisations  
*(à destination de développeurs en formation)*  

---

## 1. 🔒 Pourquoi c’est important ?  
- Un utilisateur ne doit jamais pouvoir accéder à des données ou fonctions qui ne le concernent pas.  
- Une mauvaise gestion des droits peut permettre :  
  - à un utilisateur lambda d’accéder à des données sensibles,  
  - à un attaquant de devenir administrateur,  
  - à une API publique d’être utilisée de manière abusive.  

👉 Bien gérer les droits = protéger les utilisateurs et l’application.  

---

## 2. 🧩 Concepts clés  

### Authentification vs Autorisation  
- **Authentification** → vérifier *qui* est l’utilisateur (login, mot de passe, token).  
- **Autorisation** → vérifier *ce que* l’utilisateur a le droit de faire.  

---

### Principes de base  
- **Principe du moindre privilège** → donner uniquement les permissions nécessaires.  
- **Séparation des rôles** → distinguer les droits entre utilisateur, modérateur, admin, etc.  
- **Contrôle systématique côté serveur** → ne jamais se baser uniquement sur le frontend.  

---

## 3. 🎭 Modèles de gestion des droits  

### 1. Contrôle basé sur les rôles (RBAC – Role-Based Access Control)  
- Chaque utilisateur est associé à un ou plusieurs rôles.  
- Chaque rôle définit des permissions.  
- Exemple :  
  - Rôle `user` → lire ses données.  
  - Rôle `admin` → gérer les utilisateurs, accéder aux logs.  

---

### 2. Contrôle basé sur les attributs (ABAC – Attribute-Based Access Control)  
- Les autorisations dépendent d’attributs (utilisateur, ressource, contexte).  
- Exemple :  
  - Utilisateur appartient à `service=X`.  
  - Peut accéder uniquement aux données dont `service=X`.  

---

### 3. Listes de contrôle d’accès (ACL – Access Control Lists)  
- Chaque ressource a une liste définissant quels utilisateurs/roles y ont accès.  
- Plus granulaire, mais peut devenir complexe à gérer.  

---

## 4. 👨‍💻 Mise en œuvre côté développeur  

### JavaScript (Node.js – Express)  
```javascript
// Middleware simple de contrôle de rôle
function checkRole(role) {
  return (req, res, next) => {
    if (req.user && req.user.role === role) {
      next();
    } else {
      res.status(403).send("Accès interdit");
    }
  }
}

// Exemple : route réservée aux admins
app.get("/admin", checkRole("admin"), (req, res) => {
  res.send("Bienvenue admin !");
});
```  

---

### Java (Spring Security)  
```java
// Annotation de rôle sur une méthode
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long userId) {
    // code pour supprimer un utilisateur
}
```  

---

### PHP (Laravel)  
```php
// Vérification via les policies
if ($user->can('delete', $post)) {
    // L’utilisateur peut supprimer le post
}
```  

---

## 5. 🤖 Comptes de service  

- Un **compte de service** est un compte non humain, utilisé par un processus ou une application pour exécuter des tâches automatisées.  
- Exemple :  
  - Sur une VM Linux, on crée un utilisateur `backup` qui lance un **cron** pour sauvegarder la base de données.  
  - Dans le cloud (GCP, AWS, Azure), un service peut avoir un **compte de service** avec des droits limités pour accéder à une API.  
- Ces comptes doivent respecter le **principe du moindre privilège** :  
  - Le compte `backup` n’a accès qu’aux fichiers de sauvegarde, pas au reste du système.  
  - Un compte de service cloud a uniquement les permissions nécessaires (ex. lecture d’un bucket S3).  

👉 En tant que développeur ou admin :  
- Créez toujours des **comptes séparés** pour les services automatisés.  
- Ne réutilisez pas des comptes humains (ex : admin) pour exécuter des tâches planifiées.  
- Limitez strictement leurs permissions.  
- Surveillez et loggez leurs actions pour détecter un comportement anormal.  

---

## 6. ⚠️ Erreurs courantes à éviter  
- Vérifier uniquement côté frontend (les droits doivent être vérifiés côté backend).  
- Donner trop de permissions par défaut.  
- Oublier de révoquer les droits d’un utilisateur qui change de rôle ou quitte l’organisation.  
- Laisser des endpoints d’API accessibles sans contrôle d’autorisation.  

---

## 7. ✅ Bonnes pratiques  
- Toujours appliquer le **principe du moindre privilège**.  
- Centraliser la gestion des droits (middleware, annotations, policies).  
- Logger les accès sensibles.  
- Tester régulièrement les autorisations avec des cas limites (ex. utilisateur normal essayant d’accéder à une route admin).  

---

## 8. 🛠️ Outils et solutions  

- **Keycloak** : solution open-source pour la gestion centralisée de l’authentification et des autorisations (supporte OAuth2, OpenID Connect, SAML, RBAC).  
- **Auth0** : service SaaS qui fournit authentification, gestion des rôles et règles personnalisées.  
- **Open Policy Agent (OPA)** : moteur de règles permettant de définir des autorisations selon le modèle ABAC (politiques en Rego).  
- **AWS IAM / GCP IAM / Azure AD** : solutions cloud natives pour gérer les identités, rôles et permissions des services et utilisateurs.  

👉 Ces outils permettent de **ne pas réinventer la roue** et de déléguer la complexité de la gestion des identités et des droits à des solutions robustes et éprouvées.  

---

## 9. ✅ À retenir  
- **Authentification ≠ Autorisation**.  
- Trois grands modèles : **RBAC**, **ABAC**, **ACL**.  
- Toujours vérifier côté serveur, jamais uniquement côté client.  
- Les **comptes de service** doivent être créés séparément et limités dans leurs droits.  
- Les **outils spécialisés** (Keycloak, Auth0, OPA, IAM cloud) facilitent la mise en œuvre dans des environnements complexes.  
- Une gestion stricte des droits protège contre des compromissions majeures.  

---

👉 Avec cette fiche, les étudiants comprennent la différence entre **authentification** et **autorisation**, découvrent les modèles de gestion des droits, apprennent à gérer les **comptes de service** et découvrent les **outils existants** pour gérer ces problématiques dans des systèmes réels.  
