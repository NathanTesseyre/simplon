---
title: "Exécution des applications"
tags:
  - fondamentaux
  - OS
  - compilation
section: 01-Fondamentaux
domaine: Machine & OS
statut: actif
liens_connexes:
  - [[fiche_fonctionnement_ordinateur]]
  - [[fiche_processus_threads]]
---

# 📘 Fiche : Les différentes manières d’exécuter une application  
*(Compilation, interprétation, machines virtuelles, conteneurs, cloud, navigateur)*  

---

## 1. 🚀 Vue générale : du code source à l’application qui tourne  

Quand un développeur écrit du code, l’ordinateur **ne comprend pas directement** ce qui est écrit dans un langage comme Python, Java ou C.  
👉 Le CPU (processeur) n’exécute que du **langage machine** : une suite de 0 et 1.  

Il faut donc un **intermédiaire** pour transformer le code source en instructions compréhensibles par la machine.  

### Étapes simplifiées :  
1. **Code source** → écrit dans un langage humainement lisible (C, Python, JavaScript…).  
2. **Traduction** → par un **compilateur** (traduction avant exécution) ou un **interpréteur** (traduction à la volée).  
3. **Code machine** → instructions binaires exécutées par le processeur.  
4. **Résultat** → affichage, calcul, communication réseau, etc.  

---

### Métaphore 🥗  
- Le **code source** = la recette écrite.  
- Le **compilateur / interpréteur** = le traducteur qui transforme la recette en instructions pour le robot de cuisine.  
- Le **CPU** = le robot qui suit ces instructions à la lettre.  

➡️ Selon le traducteur choisi (compilateur ou interpréteur), la préparation du plat sera **plus rapide** (compilation) ou **plus flexible** (interprétation).  

---

Cette distinction (comment le code est traduit) est la **clé** pour comprendre les différences de performance, de portabilité et de cycle de développement entre les langages.  

---

## 2. 🏗️ Traduction du code : Compilation vs Interprétation  

### Compilation  
- Transforme **tout le code source** en un **programme exécutable** avant son exécution.  
- Exemples : C, C++, Go, Rust.  
- Avantages : ⚡ exécution rapide, optimisations possibles.  
- Inconvénients : ⏳ compilation longue, dépend du système d’exploitation.  
- Métaphore : traduire **tout un livre** avant de pouvoir le lire.  

---

### Interprétation  
- Le code est **lu et exécuté ligne par ligne** par un interpréteur.  
- Exemples : Python, JavaScript, PHP, Ruby.  
- Avantages : 🔁 flexibilité, exécution immédiate (pas besoin de compilation).  
- Inconvénients : 🐢 plus lent, moins optimisé.  
- Métaphore : traduire **une phrase à la fois** en lisant un livre.  

---

### Bytecode et machines virtuelles  
- Certains langages compilent d’abord en **bytecode** (code intermédiaire).  
- Ce bytecode est exécuté par une **machine virtuelle** (ex. JVM pour Java, CLR pour C#).  
- Avantages : portabilité (un même bytecode tourne partout où la VM est installée).  
- Exemple : Java, Kotlin, C#.  

---

### Compilation JIT (Just-In-Time)  
- Compilation **à la volée** pendant l’exécution.  
- Combine flexibilité et performance.  
- Exemples : Java (JVM HotSpot), C# (.NET CLR), JavaScript (moteur V8 de Chrome/Node.js).  

---

### Comparaison rapide  
| Aspect | Compilation | Interprétation | Bytecode/JIT |
|--------|-------------|----------------|--------------|
| Traduction | Avant exécution | Pendant exécution | Hybride |
| Vitesse exécution | ⚡ Rapide | 🐢 Plus lent | ⚡ Souvent rapide |
| Flexibilité | ❌ Moins | ✅ Plus | ✅ Bonne |
| Portabilité | ❌ Limitée | ✅ Interpréteur partout | ✅ VM partout |

---

## 3. 🖥️ Autres modes d’exécution (vue développeur)  

### 🔹 Machines virtuelles (VM complètes)  
- Simulent **un ordinateur entier** (CPU, mémoire, disque).  
- Permettent d’exécuter un OS complet isolé.  
- Exemple : VirtualBox, VMware, Hyper-V.  
- Cas d’usage : tester sur différents environnements, serveurs isolés.  

---

### 🔹 Conteneurs (Docker, Kubernetes)  
- Partagent le **même noyau système** mais isolent les applications.  
- Plus légers et rapides que des VM.  
- Cas d’usage : déploiement d’applications modernes, microservices.  

---

### 🔹 Exécution dans le cloud  
- L’application tourne sur des serveurs distants.  
- Le développeur ne gère pas forcément l’infrastructure (PaaS, Serverless).  
- Exemples : AWS Lambda (serverless), Heroku, Google Cloud Run.  

---

### 🔹 Exécution dans le navigateur  
- Applications directement accessibles par l’utilisateur via un navigateur.  
- Langage dominant → JavaScript (et ses dérivés comme TypeScript).  
- Possibilités élargies avec **WebAssembly (WASM)** → exécuter du code compilé (C, Rust) dans un navigateur.  

---

## 4. 🎯 Pourquoi c’est important pour un développeur ?  
- Comprendre les différences entre **compilation et interprétation** → savoir anticiper performances et portabilité.  
- Savoir ce qu’il y a derrière les **moteurs d’exécution modernes** (JIT, VM) → mieux comprendre pourquoi “ça rame” ou pourquoi “ça marche partout”.  
- Maîtriser les **environnements d’exécution (VM, conteneurs, cloud, navigateur)** → indispensable pour développer et déployer aujourd’hui.  

---

## 5. ✅ À retenir  
- **Compilation** = rapide, optimisé mais rigide.  
- **Interprétation** = flexible, portable mais plus lent.  
- **Hybride (bytecode + JIT)** = équilibre entre les deux.  
- Les applications peuvent tourner dans différents environnements : **VM, conteneurs, cloud, navigateur**.  
- Le choix dépend du **contexte** (performance, portabilité, déploiement, architecture).  

---

👉 Avec cette fiche unique, les étudiants comprennent **à la fois la traduction du code (compilation/interprétation)** et **les différents environnements modernes d’exécution**.  
