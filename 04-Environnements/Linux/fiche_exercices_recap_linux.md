# 📘 Fiche d’exercices Linux

⚙️ Commandes autorisées :  
`su pwd cd ls mkdir touch nano rm rmdir cat echo date find grep chown chgrp chmod > >> 2> 2>> useradd passwd`  

👉 Les scripts **bash** et le **cron** sont aussi utilisés.  
👉 Tous les exercices doivent être faits dans un terminal Linux.  
👉 ⚠️ Les manipulations d’utilisateurs nécessitent les droits administrateur.

---

## Exercice 1 — Prise en main & navigation
1. Affiche ton répertoire courant.  
2. Va dans ton répertoire personnel puis crée un dossier `atelier_linux` et un sous-dossier `notes`.  
3. Vérifie la structure.  

---

## Exercice 2 — Fichiers simples
1. Dans `notes`, crée un fichier vide `todo.txt`.  
2. Ajoute une ligne “Acheter du café” sans ouvrir d’éditeur.  
3. Ajoute ensuite “Relire le cours” en append.  
4. Affiche le contenu.  

---

## Exercice 3 — nano & redirections
1. Ouvre `todo.txt` avec **nano** et ajoute une ligne “Sauvegarder ses fichiers”.  
2. Copie le contenu dans `todo_copie.txt` (en écrasant si besoin).  

---

## Exercice 4 — Nettoyage
1. Crée un dossier `a_supprimer`.  
2. Crée un fichier `temp.log` dedans puis supprime-le.  
3. Supprime ensuite le dossier vide.  

---

## Exercice 5 — Dates & logs
1. Crée un fichier `journal.txt` contenant la date du jour.  
2. Ajoute la date actuelle à la suite.  
3. Affiche le contenu.  

---

## Exercice 6 — Rechercher et filtrer
1. Crée trois fichiers : `cours_linux.txt`, `cours_bash.txt`, `image.png`.  
2. Ajoute la phrase “Linux est puissant” dans les deux `.txt`.  
3. Trouve tous les fichiers `.txt` sous `atelier_linux`.  
4. Affiche uniquement ceux qui contiennent le mot “puissant”.  

---

## Exercice 7 — Propriété & groupes
1. Change le propriétaire de `journal.txt` vers ton utilisateur.  
2. Change son groupe vers ton groupe principal.  
3. Vérifie.  

---

## Exercice 8 — Création et bascule d’utilisateur
### Objectif  
Créer un utilisateur, lui donner un mot de passe, vérifier son répertoire personnel et basculer dessus.

1. **Passe en super-utilisateur (root)** :  
   ```bash
   su
   ```
   (saisis le mot de passe root)

2. **Crée un utilisateur `etudiant1` avec dossier personnel et shell Bash** :  
   ```bash
   useradd -m -s /bin/bash etudiant1
   ```
   - `-m` : crée `/home/etudiant1`  
   - `-s` : définit le shell par défaut (`/bin/bash`)  

3. **Attribue un mot de passe** :  
   ```bash
   passwd etudiant1
   ```

4. **Vérifie la création** :  
   ```bash
   ls -ld /home/etudiant1
   grep etudiant1 /etc/passwd
   ```

5. **Bascule sur ce nouvel utilisateur** :  
   ```bash
   su - etudiant1
   ```
   Vérifie ensuite :  
   ```bash
   pwd
   ls -a
   ```

6. **Quitte la session de l’utilisateur `etudiant1`** :  
   ```bash
   exit
   ```
   👉 Tu es de retour sur la session **root**.

7. **Quitte ensuite la session root pour revenir à ton utilisateur initial** :  
   ```bash
   exit
   ```

---

### Note pédagogique
- `useradd` est une commande **bas niveau**, il faut préciser les options.  
- `adduser` (présent sur Debian/Ubuntu) est un **script interactif** qui pose des questions (mot de passe, shell, groupe, etc.) et configure tout automatiquement.  
- Quand on enchaîne `su` puis `su - etudiant1`, les sessions sont **imbriquées** : il faut donc **deux `exit` successifs** pour revenir au compte de départ.  

---

## Exercice 9 — Permissions
1. Retire l’exécution à tous sur `todo.txt`.  
2. Donne lecture/écriture à toi, lecture au groupe, rien aux autres.  
3. Vérifie.  

---

## Exercice 10 — Erreurs & redirections
1. Tente d’afficher un fichier inexistant et redirige l’erreur.  
2. Re-tente en ajoutant à la suite.  
3. Vérifie.  

---

## Exercice 11 — Script bash
1. Crée `compte_txt.sh` qui :  
   - affiche la date,  
   - liste tous les `.txt`,  
   - compte combien il y en a.  
2. Rends-le exécutable et lance-le avec redirection sortie et erreur.  

---

## Exercice 12 — Cron
1. Ouvre la crontab utilisateur.  
2. Programme l’exécution de `compte_txt.sh` toutes les minutes (test).  
3. Vérifie le fichier de log après quelques minutes.  

---

## Exercice BONUS (corsé) — Index multi-projets robuste + droits + cron

### Objectif
Écrire un **script Bash robuste** qui indexe des notes réparties dans des répertoires aux **noms très longs**, extrait les lignes taguées `#TODO` et `#IDEA`, gère les **erreurs** et **droits**, et se lance automatiquement via **cron**.  
➡️ Utilise la **tabulation** pour compléter les noms (ne copie-colle pas).

*(consignes détaillées comme vues ensemble)*
