---
title: "Processus et threads"
tags:
  - fondamentaux
  - OS
  - concurrence
section: 01-Fondamentaux
domaine: Machine & OS
statut: actif
liens_connexes:
  - [[fiche_gestion_memoire]]
  - [[fiche_execution_applications]]
---

# 📘 Fiche : Processus et Threads  
*(à destination de développeurs en formation)*  

---

## 1. 🤔 Pourquoi parler de processus et threads ?  
Quand on lance un programme (navigateur, éditeur de texte, serveur web), l’OS crée une **unité d’exécution** appelée **processus**.  
- Un ordinateur exécute souvent **des dizaines de processus en parallèle**.  
- À l’intérieur d’un processus, on peut avoir **plusieurs threads** → plusieurs flux d’instructions s’exécutant en même temps.  

👉 Comprendre ça permet de mieux appréhender :  
- le **multitâche** (l’illusion que tout tourne en même temps),  
- la **concurrence** et le **parallélisme**,  
- les problèmes classiques comme les **verrous, deadlocks, synchronisation**.  

---

## 2. 📦 Processus  

### Définition  
Un processus est une **instance d’un programme en cours d’exécution**, gérée par le système d’exploitation.  

### Caractéristiques  
- Possède sa **mémoire propre** (pile, tas, variables globales).  
- Isolé des autres processus (sécurité et stabilité).  
- Contrôlé par l’OS (création, suspension, terminaison).  

### Exemple  
- Quand on ouvre **Chrome**, chaque onglet peut être un processus distinct.  
- Quand on lance `python script.py`, l’OS crée un processus Python.  

---

## 3. 🔀 Threads  

### Définition  
Un thread est une **sous-unité d’exécution à l’intérieur d’un processus**.  
- Tous les threads d’un processus **partagent la même mémoire**.  
- Chaque thread a sa propre pile d’exécution.  

### Avantages  
- ⚡ Plus légers que des processus (création et commutation rapides).  
- Permettent de faire plusieurs choses en parallèle dans une même application.  

### Exemple (système)  
- Un navigateur :  
  - Un thread pour l’affichage.  
  - Un thread pour charger une page.  
  - Un thread pour exécuter du JavaScript.  

---

### 👨‍💻 Threads dans le développement applicatif  
Dans la pratique, les développeurs utilisent les threads pour rendre une application **plus réactive**.  

#### Exemple en **Java**  
En Java, on peut créer un thread en héritant de `Thread` ou en utilisant l’interface `Runnable` :  

```java
class MonThread extends Thread {
    public void run() {
        System.out.println("Thread en cours d'exécution !");
    }
}

public class Exemple {
    public static void main(String[] args) {
        MonThread t = new MonThread();
        t.start(); // lance le thread
    }
}
```

- Ici, `t.start()` crée un nouveau thread qui s’exécute **en parallèle** du thread principal (`main`).  
- Utile pour des tâches de fond : télécharger un fichier, écouter une requête réseau, traiter des données…  

👉 Dans les applis modernes, les threads sont souvent gérés via des **pools de threads** ou des **APIs de concurrence** (`ExecutorService` en Java, `async/await` en Python/JavaScript).  

---

> ⚠️ **À ne pas confondre : threads logiciels vs threads matériels**  
> - **Thread logiciel** : flux d’exécution créé par un programme, géré par l’OS.  
> - **Thread matériel** : capacité d’un cœur de processeur à exécuter plusieurs flux (Hyper-Threading / SMT).  
> 👉 L’OS planifie les threads logiciels sur les threads matériels disponibles.  

---

## 4. 🧮 Multitâche, Concurrence et Parallélisme  

### Multitâche  
- Capacité de l’OS à **alterner rapidement** entre plusieurs processus.  
- Illusion que tout tourne en même temps (même sur un processeur simple cœur).  

### Concurrence  
- Plusieurs threads/processus progressent en même temps, mais pas forcément exactement au même instant.  
- Exemple : pendant qu’un thread attend une requête réseau, un autre continue un calcul.  

### Parallélisme  
- Exécution **réelle en simultané** grâce aux processeurs multi-cœurs.  
- Exemple : diviser un gros calcul en plusieurs threads exécutés sur différents cœurs.  

---

## 5. ⚙️ Context Switching  

- L’OS est responsable de gérer l’alternance entre processus/threads → c’est le **scheduler**.  
- Il sauvegarde l’état du processus (registres, pointeur d’instruction, pile) → **context switch**.  
- Très rapide mais pas gratuit → trop de threads/processus = surcharge.  

---

## 6. ⚠️ Problèmes fréquents avec les threads  

- **Conditions de course (race conditions)** : deux threads accèdent à la même variable en même temps → résultats imprévisibles.  
- **Deadlock** : deux threads s’attendent mutuellement → blocage.  
- **Starvation** : certains threads ne reçoivent jamais de temps processeur.  
- **Overhead** : trop de threads → surcharge CPU/mémoire due aux context switch.  

---

## 7. 👨‍💻 Outils pour observer processus et threads  

### Linux / Mac  
- `ps aux` → lister les processus.  
- `top` ou `htop` → surveiller utilisation CPU/RAM par processus.  
- `strace -p <pid>` → suivre les appels système d’un processus.  

### Windows  
- Gestionnaire des tâches → onglet *Détails*.  
- PowerShell → `Get-Process`.  

---

## 8. ✅ À retenir  
- Un **processus** = programme en cours d’exécution, isolé avec sa propre mémoire.  
- Un **thread logiciel** = flux d’exécution dans un processus, partageant la mémoire.  
- Un **thread matériel** = capacité d’un cœur physique à gérer plusieurs flux.  
- Dans une **application**, les threads permettent de **paralléliser des tâches** (Java, Python, etc.).  
- **Multitâche** = illusion de simultanéité, **concurrence** = progression simultanée, **parallélisme** = exécution réelle en même temps.  
- L’OS gère l’exécution via le **scheduler** et les **context switch**.  
- Attention aux problèmes classiques : race conditions, deadlocks, surcharge de threads.  

---

👉 Avec cette fiche, les étudiants comprennent la **différence entre processus et threads**, savent **comment utiliser les threads dans leurs applis**, et évitent la confusion avec la notion matérielle.  
