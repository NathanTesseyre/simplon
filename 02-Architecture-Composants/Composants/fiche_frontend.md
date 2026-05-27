---
title: "Le frontend"
tags:
  - composants
  - frontend
  - HTML
  - CSS
  - JS
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche_backend]]
  - [[fiche_api]]
  - [[fiche_composants_frontend]]
---

# 🎨 Fiche Frontend

## 🔎 Qu’est-ce que le Frontend ?
Le **Frontend** désigne la partie visible et interactive d’une application, ce que l’utilisateur manipule directement.

Il repose sur trois piliers :
- **HTML** (structure)
- **CSS** (style)
- **JavaScript/TypeScript** (interactions)

---

## 🧱 Rôles principaux
- Affichage, ergonomie, design
- Interaction utilisateur
- Communication avec le backend (API)
- Gestion d’état
- Performance

---

## ⚙️ Technologies de base
- **Langages** : HTML5, CSS3, JS/TS
- **Outils build** : Vite, Webpack, Babel
- **UI kits** : Tailwind, Bootstrap, Material UI
- **Tests** : Jest, Cypress, Playwright

---

## 🏗️ Frameworks Frontend modernes

### ⚡ Différence avec HTML/CSS/JS “classique”
En pur HTML/CSS/JS, on peut créer un site simple. Mais dès que la complexité augmente (SPA, état, routing, API), un framework devient nécessaire.

### ⚙️ Les trois principaux
- **React** : flexible, écosystème large
- **Angular** : framework complet et strict
- **Vue.js** : compromis simplicité/structure

### 📚 Librairie vs Framework

Il est important de distinguer les deux notions :

- **Librairie** : un ensemble de fonctions/utilitaires que l’on appelle quand on en a besoin.  
  👉 C’est **le développeur qui contrôle le flux**.  
  - Exemples JS : **Lodash** (manipulation de tableaux/objets), **Axios** (requêtes HTTP), **D3.js** (visualisations), **React** (gestion de la vue uniquement).

- **Framework** : un cadre structurant qui définit comment organiser son application.  
  👉 C’est **le framework qui appelle ton code** (principe d’“inversion de contrôle”).  
  - Exemples JS : **Angular** (complet et strict), **Vue.js** (progressif), **Next.js** (basé sur React, avec routing/SSR).

⚖️ **Différence clé** :  
- Avec une librairie → tu es le chef d’orchestre, tu décides quand et comment l’utiliser.  
- Avec un framework → tu suis ses conventions, il impose un cycle de vie et appelle ton code au bon moment.  

💡 Exemple concret :  
- Avec **Axios** (librairie), tu décides quand lancer une requête HTTP.  
- Avec **Angular** (framework), c’est lui qui gère le cycle de vie des composants et déclenche certaines actions automatiquement.

#### 🤔 Pourquoi **React** est considéré comme une *librairie* (et pas un framework)
- **Portée ciblée** : React gère essentiellement **la vue** (components, rendu, état local).  
- **Pas d’architecture imposée** : tu choisis comment structurer ton projet.  
- **Tu appelles React** : tu importes et utilises ses hooks (`useState`, `useEffect`…), plutôt que l’inverse.  
- **Écosystème à la carte** : ajoute **React Router** (routing), **Redux/Zustand** (état global), **SWR/React Query** (données), **Next.js** (SSR/routing).  
➡️ Pris seul, React n’est pas un “tout-en-un”, donc on le classe côté **librairie**.

### 🖼️ Schéma
![Frontend avec framework](img/frontend_framework.png)

---

## 🔄 Alternatives côté serveur (PHP & Java)

Avant les frameworks JS modernes, beaucoup d’applications généraient le frontend côté serveur.

### PHP
- **Sans framework** : simples fichiers `.php` qui génèrent dynamiquement du HTML en mélangeant logique et présentation.  
- **CMS** : WordPress, Drupal, Joomla → solutions clés en main pour créer rapidement des sites dynamiques (blog, e-commerce).  
- **Frameworks modernes** : Laravel (Blade), Symfony (Twig), CodeIgniter → plus structurés, séparation MVC, templates, sécurité renforcée.  
👉 Très adapté aux sites éditoriaux, blogs, boutiques en ligne.

### Java
- **Servlets/JSP** : approche plus bas niveau, manipulation manuelle du HTML et des requêtes.  
- **Spring MVC + Thymeleaf** : génération côté serveur avec un moteur de template moderne, utilisé en entreprise.  
- **JSF (Java Server Faces)** : composants UI côté serveur, adoption surtout dans les SI d’entreprise.  
👉 Souvent choisi pour les portails, intranets et applications lourdes avec forte intégration backend.

## 📊 Comment choisir ?

| Option | Cas d’usage typiques | Points forts | Limites | À surveiller |
|---|---|---|---|---|
| **HTML/CSS/JS (sans framework)** | Sites vitrines simples, formulaires basiques, pages statiques | Simplicité, peu de dépendances, chargements très rapides | Peu adapté aux apps interactives/complexes | Accessibilité, organisation du code |
| **React (librairie UI)** | Apps modulaires, besoin de liberté d’architecture | Écosystème riche, composables, adoption progressive | À compléter (routing, état, data) | Taille de bundle, conventions d’équipe |
| **Angular (framework complet)** | Grands projets, équipes nombreuses | Structure, tooling intégré, DX cohérente | Courbe d’apprentissage, verbosité | Performance initiale, discipline modules |
| **Vue.js** | Projets petits à moyens, montée progressive | Simplicité, approche progressive, écosystème officiel | Moins “opinionated” qu’Angular | Choix d’outils (Router, Pinia) |
| **SSR/SSG (Next.js, Angular Universal)** | SEO, temps de premier rendu | Meilleur SEO, rendu initial rapide | Plus complexe côté ops | Cache, hydratation, bords serveur |
| **PHP (WordPress, Symfony, Laravel)** | Sites éditoriaux, intranets, back-offices | Génération HTML côté serveur, maturité | UX moins SPA par défaut | Sécurité plugins, perfs, cache |
| **Java (Spring MVC + Thymeleaf/JSF)** | SI d’entreprise, portails | Robustesse, intégrations | Lourd pour du web public | Temps de build, déploiements |
| **Node.js/Express (backend API)** | API REST/GraphQL pour le front | Même langage front/back, temps réel | Nécessite un frontend séparé | Observabilité, montée en charge |

## 🌐 Dynamique vs Statique

- **Site statique** :  
  - Le serveur envoie des fichiers **HTML/CSS/JS déjà générés**.  
  - Chaque utilisateur reçoit exactement la même page.  
  - Rapide et léger, mais peu flexible.  
  - Exemples : **landing pages marketing**, **documentation technique (ex. MDN, docs d’API)**, **sites hébergés en statique**.

- **Site dynamique** :  
  - Le serveur génère du contenu **à la volée** en fonction de l’utilisateur, de la session ou d’une base de données.  
  - Permet des fonctionnalités interactives (panier, profil, dashboard).  
  - Exemples : **PHP avec WordPress, Laravel**, **Java avec Spring MVC**, **Node.js/Express avec bases de données**.

⚖️ **En résumé** :  
- Statique = plus rapide, adapté au contenu fixe.  
- Dynamique = plus flexible, adapté aux applications interactives.

## 🖼️ Schéma
![Frontend généré côté serveur](img/frontend_serverside.png)

## 🖥️ La notion de serveur (où intervient-il ?)

Un **serveur** est une machine (physique ou cloud) qui exécute du code et répond aux requêtes des utilisateurs.

- **Serveur web classique (ex. PHP)**  
  - L’utilisateur demande une page (ex. `/profil.php`). La requête arrive au serveur.  
  - Le serveur **exécute le PHP**, interroge éventuellement la base de données, **génère un HTML complet**, puis le renvoie au navigateur.  
  - Le serveur est **indispensable** : sans lui, le code PHP ne s’exécute pas.

- **Frameworks frontend modernes (React, Angular, Vue) en mode SPA**  
  - En production, le projet est **compilé/buildé** et livré sous forme d’un dossier **`dist/` ou `build/`** contenant : `index.html`, fichiers `.js`, `.css`, assets.  
  - Lors du premier accès, le navigateur **télécharge `index.html` et le JavaScript** nécessaire.  
  - La navigation intérieure se fait **côté navigateur** : le JavaScript met à jour l’interface sans recharger de nouvelles pages HTML.

- **Rendu côté serveur (SSR : Next.js, Angular Universal)**  
  - Le serveur **exécute le code** (React/Angular) pour **rendre du HTML** avant de l’envoyer.  
  - Avantages : **premier affichage plus rapide**, meilleur **SEO**.  
  - Le navigateur prend ensuite le relais (hydratation) pour l’interactivité.

## 🛠️ Bonnes pratiques
- Accessibilité (ARIA, contrastes, navigation clavier)
- Performance (Lighthouse, lazy load)
- Sécurité (XSS, CSRF, CSP)
- SEO (balises meta, SSR/SSG)
- Organisation (composants réutilisables)

---

## 🧪 À vérifier après déploiement
- Performance : Lighthouse
- Accessibilité : tests clavier, lecteurs d’écran
- Compatibilité : Chrome, Firefox, Safari, Edge
- API calls : DevTools Network
- Sécurité : headers CSP/HSTS

---

## 🖼️ Autres schémas

### Frontend simple
![Frontend simple](img/frontend_simple.png)

### Comparatif global
![Comparatif](img/frontend_comparatif.png)