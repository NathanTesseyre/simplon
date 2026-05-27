---
title: "Commandes Linux de base"
tags:
  - linux
  - terminal
  - CLI
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[intro_linux]]
  - [[shell_scripting]]
  - [[fiche_theorique_grep_find]]
---

# 📘 Fiche : Commandes de base Linux

**Sommaire**
- [Introduction](#toc-intro)
- [Navigation](#toc-nav)
- [Manipulation de fichiers et dossiers](#toc-manip)
- [Consultation de fichiers](#toc-view)
- [Recherche](#toc-search)
- [Permissions et infos](#toc-perm)
- [Cas pratiques](#toc-cas)
- [✅ À retenir](#toc-ret)

<a id="toc-intro"></a>
## Introduction
La ligne de commande est la **boîte à outils** principale sous Linux.

<a id="toc-nav"></a>
## Navigation
- `pwd` : répertoire courant.  
- `ls` : lister le contenu. Options utiles :  
  - `-l` (long), `-a` (cachés), `-h` (tailles lisibles), `-t` (tri par date).  
- `cd <chemin>` : changer de répertoire. Exemples : `cd ~`, `cd -`, `cd ..`.

<a id="toc-manip"></a>
## Manipulation de fichiers et dossiers
- `cp SRC DEST` (copie), `mv SRC DEST` (déplacer/renommer), `rm FICHIER` (supprimer).  
- Dossiers : `mkdir`, `rmdir` (vide), `rm -r` (récursif).  
⚠️ `rm -rf` supprime **sans confirmation** (dangereux).

<a id="toc-view"></a>
## Consultation de fichiers
- `cat`, `less`, `head -n 20`, `tail -n 50`, `tail -f` (suivi temps réel).

<a id="toc-search"></a>
## Recherche
- `find /chemin -name "pat*"` : recherche par nom.  
- `grep -R "mot" /chemin` : rechercher du texte récursivement.  
- `which commande` / `type commande` : savoir ce qu’on exécute.

<a id="toc-perm"></a>
## Permissions et infos
- `ls -l` : permissions, propriétaire, groupe.  
- `stat fichier` : détails (inode, dates).

<a id="toc-cas"></a>
## Cas pratiques
1. Lister tous les fichiers cachés de `~`.  
2. Copier `notes.txt` vers `/tmp/` et vérifier.  
3. Suivre `/var/log/auth.log` en temps réel.  
4. Trouver toutes les lignes contenant `"ERROR"` dans `/var/log/syslog`.

<a id="toc-ret"></a>
## ✅ À retenir
- `ls`, `cd`, `pwd` → naviguer.  
- `cp`, `mv`, `rm` → manipuler.  
- `cat`, `less`, `tail` → lire.  
- `find`, `grep` → chercher.
