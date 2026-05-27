---
title: "Architecture des systèmes"
tags:
  - architecture
  - microservices
  - monolithique
section: 02-Architecture-Composants
domaine: Architecture
statut: actif
liens_connexes:
  - [[fiche_architecture_infra]]
  - [[docker_fondamentaux]]
---

# 📘 Fiche : Architecture des systèmes  
*(à destination de développeurs en formation)*  

---

## 1. 🏗️ Pourquoi parler d’architecture ?  
- Une application ne se limite pas à du code : elle repose sur des **couches** qui s’organisent et communiquent.  
- Comprendre cette organisation permet de :  
  - mieux développer (où placer le code),  
  - mieux déboguer (où chercher quand ça casse),  
  - mieux collaborer avec d’autres équipes (devs, ops, admins réseau).  

---

## 2. 🎨 Architecture applicative  
- Organisation de l’application du point de vue **fonctionnel** (ce que voit l’utilisateur).  
- Exemple classique → **architecture 3-tiers** :  
  - **Frontend (client)** : interface utilisateur (navigateur, app mobile).  
  - **Backend (serveur applicatif)** : logique métier, API, traitement des données.  
  - **Base de données** : stockage et accès aux informations.  

👉 Exemples :  
- Une boutique en ligne → site web (frontend), serveur de paiement et panier (backend), base produits et commandes (BDD).  
- Une app mobile → UI (frontend), API REST (backend), base Cloud (BDD).  

---

## 3. ⚙️ Architecture logicielle  
- Organisation **interne du code** : séparation des responsabilités.  
- Exemple de couches classiques :  
  - **Interface utilisateur (UI)** : boutons, formulaires, affichage.  
  - **Logique métier** : règles de calcul, processus métier.  
  - **Accès aux données (DAO, repository)** : dialogue avec la base.  

👉 Avantage :  
- Chaque couche est indépendante.  
- Facilite la maintenance, les tests, l’évolution.  

---

## 4. 🌐 Architecture infrastructure  
- Vue **matérielle et déploiement**. Où et comment l’application tourne ?  
- **Sur une seule machine** (PC local, serveur unique).  
- **Sur plusieurs serveurs** (ex : backend séparé de la BDD).  
- **Dans des VM** → chaque service tourne dans une machine virtuelle.  
- **Dans des conteneurs (Docker)** → services isolés mais plus légers que des VMs.  
- **Dans le cloud** → services hébergés et scalables automatiquement (AWS, Azure, GCP).  

👉 Exemple :  
- Une API backend déployée dans un conteneur Docker, qui communique avec une BDD hébergée dans le cloud.  

---

## 5. ✅ À retenir  
- **Architecture applicative** → qui fait quoi (frontend, backend, BDD).  
- **Architecture logicielle** → comment le code est organisé en couches.  
- **Architecture infrastructure** → où et comment ça tourne (serveurs, VM, conteneurs, cloud).  
- Chaque niveau répond à des besoins différents mais ils sont complémentaires.  

---

## 6. 🧭 Parcours recommandé  

1. **Commencez par l’architecture applicative** → comprendre les rôles du frontend, du backend et de la base de données.  
2. **Explorez ensuite l’architecture logicielle** → comment le code est organisé en couches (UI, logique métier, accès aux données).  
3. **Terminez avec l’architecture infrastructure** → voir où et comment déployer (serveurs, VM, conteneurs, cloud).  
4. **Faites le lien avec les autres fiches** :  
   - Les dossiers `Composants/` détaillent chaque brique (frontend, backend, API, BDD, serveurs web).  
   - Le dossier `Systeme/` explique ce qu’il se passe côté OS et ressources.  
   - Le dossier `Securite/` montre comment protéger chaque niveau.  

👉 Ce cheminement permet d’aller du **fonctionnel** (ce que fait l’appli) vers le **technique** (comment c’est codé et déployé).  
