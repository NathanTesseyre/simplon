---
title: "Algorithmes et complexité"
tags:
  - fondamentaux
  - algorithmique
  - Big-O
section: 01-Fondamentaux
domaine: Algorithmes
statut: actif
liens_connexes:
  - [[fiche_git]]
  - [[fiche_processus_threads]]
  - [[fiche_gestion_memoire]]
---

# 📘 Fiche : Introduction aux algorithmes  
*(à destination de développeurs en formation)*  

---

## 1. 🤔 Qu’est-ce qu’un algorithme ?  
Un **algorithme** est une **suite d’instructions** qui décrit comment résoudre un problème étape par étape.  

➡️ Métaphore : une recette de cuisine est un algorithme →  
- Ingrédients 🍅🥔 (entrées)  
- Étapes de préparation 👩‍🍳 (instructions)  
- Plat final 🍲 (sortie)  

Exemple simple : **trouver le plus grand nombre entre deux valeurs**  
1. Lire `a` et `b`.  
2. Si `a > b` → afficher `a`.  
3. Sinon → afficher `b`.  

---

## 2. 🔄 Algorithme vs Programme  
- **Algorithme** → logique pure, méthode abstraite (indépendante du langage).  
- **Programme** → implémentation concrète dans un langage donné.  

👉 Exemple :  
- Algorithme : “Pour trier une liste, comparer les éléments deux à deux et les échanger si nécessaire.”  
- Programme : version en Python, Java, C…  

---

## 3. 📝 Comment représenter un algorithme ?  
- **Pseudo-code** : proche du langage naturel.  
- **Diagramme de flux** : schéma visuel avec des boîtes et flèches.  
- **Code** : traduction dans un langage de programmation.  

📌 Exemple en pseudo-code → calculer la somme de 1 à N :  
```
Entrée : un entier N
Sortie : la somme de 1 à N
Début
    somme ← 0
    Pour i de 1 à N
        somme ← somme + i
    FinPour
    Afficher somme
Fin
```  

---

## 4. ⏱️ Notion de complexité  
Tous les algorithmes ne se valent pas : certains sont rapides ⚡, d’autres lents 🐢.  

### Complexité en temps  
Mesure du **nombre d’opérations** en fonction de la taille de l’entrée (n).  

- Parcourir une liste → **O(n)**  
- Recherche dichotomique (liste triée) → **O(log n)**  
- Double boucle imbriquée → **O(n²)**  

### Complexité en espace  
Mesure de la **mémoire nécessaire**.  
- Un algorithme peut être rapide mais consommer beaucoup de mémoire.  

📌 Notation **Big-O** → outil standard pour comparer les performances.  

---

## 5. 🔍 Zoom : Pourquoi O(log n) est intéressant ?  

### 1. Qu’est-ce que log(n) ?  
- En informatique, log(n) signifie généralement **logarithme en base 2**.  
- Intuition : `log₂(n)` = combien de fois on peut **diviser n par 2** avant d’arriver à 1.  

Exemple :  
- log₂(8) = 3 → car 8 → 4 → 2 → 1 (3 divisions).  
- log₂(16) = 4 → car 16 → 8 → 4 → 2 → 1.  

---

### 2. Où ça apparaît ?  
Un algorithme est en O(log n) quand il **réduit le problème de moitié à chaque étape**.  

Exemples :  
- 🔎 Recherche dichotomique dans une liste triée.  
- 📂 Parcours d’un arbre binaire équilibré.  
- 🔐 Algorithmes d’exponentiation rapide.  

---

### 3. Pourquoi c’est puissant ?  
Parce qu’O(log n) **grandit beaucoup plus lentement** que O(n).  

📊 Comparaison :  
- Si tu as **1 000 000 éléments** :  
  - O(n) = 1 000 000 opérations.  
  - O(log n) = log₂(1 000 000) ≈ 20 opérations seulement 🤯  

---

### 4. Métaphore simple  
Chercher un mot dans un dictionnaire papier 📖 :  
- Recherche naïve (O(n)) → lire les pages une par une.  
- Recherche dichotomique (O(log n)) → ouvrir au milieu, voir avant/après, diviser encore…  
- Résultat : tu trouves ton mot en quelques étapes seulement, même si le dictionnaire a des milliers de pages.  

👉 En résumé :  
- **O(log n)** = extrêmement efficace.  
- Dès qu’on peut diviser le problème en 2, on gagne un temps énorme.  

---

## 6. 📊 Exemples concrets d’algorithmes  
### 🔎 Recherche linéaire  
- Parcourt chaque élément un par un.  
- Complexité → **O(n)**.  

### 🔎 Recherche dichotomique  
- Nécessite une liste **triée**.  
- Divise la liste par 2 à chaque étape.  
- Complexité → **O(log n)**.  

### 🔎 Tri par insertion  
- Insère les éléments un par un au bon endroit.  
- Simple mais lent sur de grandes données.  
- Complexité → **O(n²)**.  

---

## 7. 🔁 La récursivité  
Un algorithme peut **s’appeler lui-même** → c’est la **récursivité**.  

📌 Exemple : calcul de la factorielle  
```
factorielle(n) =
    1                si n = 0
    n * factorielle(n-1)  sinon
```  

En Python :  
```python
def factorielle(n):
    if n == 0:
        return 1
    return n * factorielle(n-1)
```  

---

## 8. 📦 Algorithmes et structures de données  
Un algorithme travaille souvent **sur une structure de données** :  
- **Tableaux / Listes** → tri, recherche.  
- **Piles (Stack)** → dernier arrivé, premier sorti (LIFO).  
- **Files (Queue)** → premier arrivé, premier sorti (FIFO).  
- **Graphes** → recherche de chemin (GPS, réseaux sociaux).  
- **Arbres** → organisation hiérarchique (systèmes de fichiers, JSON).  

👉 La structure de données choisie influence directement la **performance** de l’algorithme.  

---

## 9. 🎯 Pourquoi c’est important pour un développeur ?  
- Choisir le bon algorithme = **gains énormes** en rapidité et en mémoire.  
- Comprendre les limites → éviter du code inefficace.  
- Indispensable en :  
  - Bases de données (indexation, recherche).  
  - Cryptographie (algorithmes sécurisés).  
  - IA et machine learning (optimisation).  
- Même avec des bibliothèques prêtes à l’emploi, savoir ce qu’il y a derrière permet de **mieux déboguer et optimiser**.  

---

## 10. 🧪 Cas pratique : factorielle  
**Problème :** Écrire un algorithme qui calcule la factorielle d’un nombre `n`.  

### Pseudo-code  
```
Entrée : n
Sortie : n!
Début
    résultat ← 1
    Pour i de 1 à n
        résultat ← résultat * i
    FinPour
    Afficher résultat
Fin
```  

### Implémentation Python  
```python
def factorielle(n):
    resultat = 1
    for i in range(1, n+1):
        resultat *= i
    return resultat
```  

---

## 11. ✅ À retenir  
- Un **algorithme** = une suite d’étapes pour résoudre un problème.  
- Il est **indépendant du langage** (contrairement à un programme).  
- Les performances se mesurent en **temps** (vitesse) et **espace** (mémoire).  
- Concepts clés : **Big-O, O(log n), récursivité, structures de données**.  
- Bien choisir un algorithme = **code plus efficace et plus robuste**.  

---

👉 Avec cette fiche, les étudiants comprennent :  
- ce qu’est un algorithme,  
- comment le représenter,  
- comment mesurer son efficacité (notamment pourquoi O(log n) est si puissant),  
- et pourquoi cela compte dans leur métier de développeur.  
