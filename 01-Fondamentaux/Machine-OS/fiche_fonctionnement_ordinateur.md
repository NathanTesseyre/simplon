---
title: "Fonctionnement d'un ordinateur"
tags:
  - fondamentaux
  - hardware
  - OS
section: 01-Fondamentaux
domaine: Machine & OS
statut: actif
liens_connexes:
  - [[fiche_execution_applications]]
  - [[fiche_gestion_memoire]]
  - [[fiche_processus_threads]]
---

# 🖥️ Fiche complète : Comment fonctionne un ordinateur ?  
*(à destination de développeurs en formation)*  

---

## 1. Un ordinateur : une machine de traitement
Un ordinateur est une machine qui :  
1. **Reçoit des données** (entrée)  
2. **Les stocke** (mémoire)  
3. **Les transforme** (traitement)  
4. **Produit des résultats** (sortie)  

---

## 2. Les composants matériels essentiels
- **CPU (Processeur)** : exécute des instructions binaires (calculs, comparaisons, contrôles).  
  - **Unité de contrôle** → orchestre les opérations.  
  - **UAL (Unité Arithmétique et Logique)** → effectue les calculs.  
  - **Registres** → mini-mémoires ultra rapides.  
- **Mémoire** :
  - **RAM** → rapide et temporaire (perd son contenu à l’arrêt).  
  - **Stockage (SSD/HDD)** → persistant.  
  - **ROM/Flash** → contient le firmware.  
- **Bus** : transportent données, adresses et signaux de contrôle.  
- **Périphériques** : entrées (clavier, souris), sorties (écran, haut-parleurs), stockage, réseau…  

---

## 3. Le firmware : premier logiciel au démarrage
Le **firmware** est un logiciel **très bas niveau** stocké dans une mémoire non volatile (ROM/Flash).  
Il est fourni par le constructeur de la machine et sert d’intermédiaire entre le **matériel brut** et le **système d’exploitation**.  

### Exemple courant : **BIOS/UEFI**
- Au démarrage :  
  1. **POST (Power On Self Test)** → vérifie RAM, CPU, périphériques.  
  2. Configure le matériel essentiel (bus, contrôleurs).  
  3. Cherche un **support de démarrage** (disque, clé USB, réseau).  
  4. Charge en mémoire un petit programme : le **bootloader**.  

➡️ Sans firmware, l’ordinateur n’aurait aucun moyen de savoir comment initialiser ses composants ni où trouver l’OS.  

---

## 4. Le système d’exploitation (OS)
Un **système d’exploitation** est un logiciel complexe qui prend le relais du firmware pour **gérer les ressources matérielles** et fournir un environnement aux applications.  

### Rôles principaux :
1. **Gestion du matériel**  
   - CPU : ordonnanceur → qui s’exécute et quand.  
   - RAM : allocation, protection, mémoire virtuelle.  
   - Disques : gestion des systèmes de fichiers (NTFS, ext4, APFS…).  
   - Périphériques : via des pilotes (drivers).  

2. **Abstraction**  
   - Fournit des **API systèmes** (ex. : `open()`, `read()`, `write()` en POSIX).  
   - Permet d’utiliser fichiers, processus, sockets **sans connaître le matériel**.  

3. **Sécurité et isolation**  
   - Droits utilisateurs et permissions.  
   - Isolation des processus (pas d’accès mémoire sauvage).  

4. **Services**  
   - Réseau, threads, gestion d’événements, interfaces graphiques…  

### Architecture simplifiée :
- **Noyau (Kernel)** : cœur de l’OS, gère CPU, mémoire, drivers.  
- **Espace utilisateur** :  
  - Bibliothèques systèmes (libc, WinAPI, etc.).  
  - Applications (IDE, navigateur, terminal...).  

---

## 5. Du binaire au code
- Tout est représenté en **bits (0/1)**.  
- Une instruction machine est découpée en :  
  - **Opcode** (opération).  
  - **Opérandes** (données).  
- Le CPU exécute selon le **cycle de von Neumann** :  
  1. Fetch (récupérer)  
  2. Decode (décoder)  
  3. Execute (exécuter)  
  4. Store (stocker)  

---

## 6. Chaîne complète : de l’allumage au programme
1. **Allumage** → CPU exécute le firmware en ROM.  
2. **Firmware (BIOS/UEFI)** → initialise matériel + charge bootloader.  
3. **Bootloader** → charge le noyau de l’OS en mémoire.  
4. **Noyau (Kernel)** → prend le contrôle et initialise les ressources.  
5. **OS** → fournit bibliothèques, API et environnement.  
6. **Application** → exécute du code haut niveau (C, Python, Java...).  

---

## 7. Exemple concret : `printf("Hello\n");` en C

1. **Écriture du code source**  
   ```c
   #include <stdio.h>
   int main() {
       printf("Hello\n");
       return 0;
   }
   ```

2. **Compilation** → traduit en instructions machine adaptées au CPU.  
3. **Chargement par l’OS** → le noyau alloue une zone mémoire au processus.  
4. **Appel à `printf`** → utilise `libc` qui fait appel à une **syscall** (`write`).  
5. **Noyau** → reçoit la syscall, passe par le **driver** de sortie standard.  
6. **Matériel** → l’écran affiche le texte.  

👉 Résultat visible : `Hello` apparaît dans le terminal.  

---

## 8. Schéma global

```
[Programme C] 
   ↓ (compilation)
[Instructions machine] 
   ↓ (chargement par l’OS)
[Syscalls : write()]
   ↓ (noyau + drivers)
[Matériel : écran]
   ↓
[Affichage : Hello]
```

---

## 9. À retenir pour le développeur
- Vos programmes passent toujours par :  
  **Application → Bibliothèques → OS → Noyau → Firmware → Matériel**.  
- Le firmware prépare le terrain, l’OS orchestre, le CPU exécute.  
- Comprendre cette chaîne aide à :  
  - Optimiser (mémoire, CPU, I/O).  
  - Déboguer (comprendre où ça bloque).  
  - Sécuriser (gérer droits et isolement).  
