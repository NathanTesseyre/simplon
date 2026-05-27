---
title: "Gestion de la mémoire"
tags:
  - fondamentaux
  - OS
  - mémoire
section: 01-Fondamentaux
domaine: Machine & OS
statut: actif
liens_connexes:
  - [[fiche_processus_threads]]
  - [[fiche_fonctionnement_ordinateur]]
---

# 📘 Fiche : Gestion de la mémoire  
*(à destination de développeurs en formation)*  

---

## 1. 🤔 Pourquoi parler de mémoire ?  
Lorsqu’un programme tourne, il manipule deux grandes familles de stockage :  

- **RAM (mémoire vive)** → mémoire rapide, utilisée pendant l’exécution.  
- **Disque dur (HDD/SSD)** → mémoire persistante, utilisée pour stocker fichiers et données à long terme.  

👉 Confondre les deux est une erreur fréquente :  
- La RAM est **volatile** (son contenu disparaît quand on éteint la machine).  
- Le disque dur est **persistant** (les données restent stockées).  

Une mauvaise gestion de la mémoire vive peut provoquer :  
- des programmes lents,  
- des fuites mémoire (la RAM se remplit inutilement),  
- des crashs (*segmentation fault*, *out of memory*).  

---

## 2. 📦 Les zones de la RAM principales  

### 🔹 La pile (*stack*)  
- Stockée en **RAM**.  
- Contient :  
  - les variables locales,  
  - les paramètres des fonctions,  
  - l’adresse de retour après un appel de fonction.  
- Organisation **LIFO (Last In, First Out)** → chaque appel de fonction crée une “pile d’exécution”.  
- Allocation **automatique** et très rapide.  
- ⚠️ Taille limitée → trop d’appels récursifs = *stack overflow*.  

---

### 🔹 Le tas (*heap*)  
- Stocké aussi en **RAM**.  
- Contient les objets, tableaux, structures créés dynamiquement.  
- Allocation contrôlée par le programme (`malloc`, `new`).  
- Plus flexible que la pile, mais plus lente car nécessite une gestion explicite.  
- ⚠️ Risques :  
  - **fuite mémoire** → oublier de libérer la RAM,  
  - **dangling pointer** → accéder à une mémoire libérée.  

---

## 3. 🛠️ Gestion de la mémoire selon les langages  

### 🔹 Langages avec gestion manuelle  
- C, C++ → le développeur doit **allouer** et **libérer** explicitement dans la **RAM**.  
- Exemple :  
```c
int* ptr = malloc(sizeof(int) * 10); // allocation dans le tas (RAM)
// utilisation...
free(ptr); // libération
```  
- Avantages : contrôle total.  
- Inconvénients : erreurs fréquentes (fuites, corruption mémoire).  

---

### 🔹 Langages avec gestion automatique (Garbage Collector)  
- Java, C#, Python, Go → un **GC** libère automatiquement la mémoire RAM non utilisée.  
- Exemple en Java :  
```java
String s = new String("Bonjour"); // allocation dans la RAM
// pas besoin de free(), le GC nettoie
```  
- Avantages : moins d’erreurs humaines.  
- Inconvénients : pauses imprévisibles quand le GC s’exécute, consommation RAM plus importante.  

---

## 4. 🧮 RAM vs disque dur dans la gestion mémoire  

### 🔹 RAM (mémoire vive)  
- Très rapide mais limitée en taille.  
- Contient : pile, tas, variables, instructions en cours.  
- Accès en nanosecondes.  
- ⚠️ Volatile : effacée quand on éteint l’ordinateur.  

### 🔹 Disque dur (HDD/SSD)  
- Plus lent mais grande capacité.  
- Sert au **stockage permanent** : fichiers, bases de données, système d’exploitation.  
- Accès en microsecondes à millisecondes (beaucoup plus lent que la RAM).  

### 🔹 Swap (quand la RAM déborde)  
- Si la RAM est saturée, l’OS déplace temporairement des données de la RAM vers le disque dur (*fichier de swap*).  
- Avantage : évite un crash immédiat.  
- Inconvénient : énorme perte de performance (le disque est ~1000x plus lent que la RAM).  

---

## 5. ⚠️ Problèmes fréquents liés à la mémoire  

- **Fuite mémoire (RAM)** : mémoire jamais libérée → le programme consomme de plus en plus de RAM.  
- **Segmentation fault (RAM)** : accès à une zone mémoire interdite (ex : C/C++).  
- **Stack overflow (RAM)** : pile saturée (ex : récursion infinie).  
- **Dangling pointer (RAM)** : pointeur qui référence une mémoire déjà libérée.  
- **Swap excessif (RAM → disque)** : ralentissements énormes si la RAM est pleine et que l’OS doit écrire sur le disque.  

---

## 6. ✅ À retenir  
- La **RAM** est la mémoire de travail du programme → rapide mais limitée et volatile.  
- Le **disque dur** est le stockage permanent → lent mais persistant.  
- La **pile (stack)** et le **tas (heap)** résident en RAM.  
- Une bonne gestion mémoire est essentielle pour éviter fuites, crashs et ralentissements.  
- L’OS peut utiliser le disque pour **swapper** quand la RAM est pleine → solution de secours mais très lente.  

---

## 7. 🐞 Déboguer un usage excessif de mémoire  

### 🔹 Étape 1 : Observer la consommation mémoire  
- Sous **Linux/Mac** :  
  ```bash
  top
  htop   # plus lisible
  free -h   # état de la RAM et du swap
  ps aux --sort=-%mem | head -10   # top 10 processus les plus gourmands
  ```  
- Sous **Windows** :  
  - Gestionnaire des tâches → onglet *Mémoire*.  
  - Outil avancé : *Resource Monitor*.  

---

### 🔹 Étape 2 : Déterminer la cause  
- **Fuite mémoire** : la consommation RAM d’un processus augmente en continu sans jamais redescendre.  
- **Données trop volumineuses** : chargement d’un fichier énorme en RAM.  
- **Boucle infinie / récursion** : empilement excessif dans la pile (*stack overflow*).  
- **Swap excessif** : signe que la RAM est saturée → le disque prend le relais.  

---

### 🔹 Étape 3 : Outils spécifiques par langage  
- **C / C++** : `valgrind`, `asan` (AddressSanitizer) pour détecter fuites et accès mémoire illégaux.  
- **Java** : VisualVM, JConsole → analyser la heap et les objets retenus.  
- **Python** : `memory_profiler`, `objgraph`, `gc` module → inspecter les objets et cycles non libérés.  
- **JavaScript (Node.js)** : `--inspect` avec Chrome DevTools → profiler l’usage mémoire.  

---

### 🔹 Étape 4 : Bonnes pratiques pour éviter les soucis  
- Libérer explicitement les ressources (fichiers, connexions).  
- Éviter de tout charger en RAM d’un coup (privilégier le streaming, pagination).  
- Surveiller la taille des structures de données (listes, dictionnaires, caches).  
- Utiliser des outils de monitoring (Prometheus, Grafana, Datadog) en production.  

---

👉 Avec cette fiche, les étudiants comprennent **où vont réellement les données (RAM vs disque)**, pourquoi la gestion mémoire est cruciale, et comment **diagnostiquer un programme qui consomme trop de mémoire**.  
