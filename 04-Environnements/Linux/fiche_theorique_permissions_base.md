---
title: "Permissions Linux — Bases"
tags:
  - linux
  - permissions
  - sécurité
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[utilisateurs_permissions]]
  - [[systeme_fichiers]]
  - [[fiche_theorique_grep_find]]
---

# 📘 Fiche théorique — Permissions de base (Linux)

> Portée : **notions de base uniquement**. Pas de liens symboliques, pas d’ACL, pas de bits spéciaux (setuid/setgid/sticky).

## 1) Qui a quels droits ?
- **Propriétaire (u)** : l’utilisateur qui possède le fichier.
- **Groupe (g)** : les membres du groupe du fichier.
- **Autres (o)** : tous les autres utilisateurs.
- On parle souvent de **u/g/o** et de **a** (*all* = u+g+o).

Affichage typique (`ls -l`) :
```
-rw-r--r-- 1 alice dev 1234  3 sept 10:00 fichier.txt
^ ^^^ ^^^ ^
| |   |   └── nom
| |   └────── groupe
| └────────── droits g (r--)
└──────────── droits u (rw-), puis droits o (r--)
```
Le premier caractère est le **type** (`-` fichier, `d` dossier).

## 2) Signification de r/w/x
### Fichier
- `r` : lire le contenu
- `w` : modifier le contenu
- `x` : exécuter (script/binaire)

### Dossier
- `r` : lister le contenu (`ls`)
- `w` : créer/supprimer/renommer des entrées
- `x` : **traverser**/entrer (`cd`) et accéder aux fichiers contenus

> Sur un dossier, **x** est indispensable pour y **entrer** même si on a `r`.

## 3) Notation **octale**
Chaque triplet (u, g, o) : `r=4`, `w=2`, `x=1` → on additionne.
- `600` = `rw- --- ---` (privé)
- `644` = `rw- r-- r--` (lecture pour tous, écriture propriétaire)
- `664` = `rw- rw- r--`
- `666` = `rw- rw- rw-`
- `700` = `rwx --- ---`
- `750` = `rwx r-x ---`
- `755` = `rwx r-x r-x`

## 4) Notation **symbolique**
Forme : `chmod [u|g|o|a][+|-|=][rwx] chemin`
- `+` ajoute, `-` retire, `=` remplace **exactement**.
Exemples :
```
chmod u+x script.sh          # ajoute l’exécution au propriétaire
chmod go-rw fichier.txt      # retire lecture/écriture à g et o
chmod u=rw,go=r fichier.txt  # fixe exactement: u (rw), g (r), o (r)
```

## 5) Voir et vérifier
- `ls -l` : droits des **fichiers**
- `ls -ld dossier` : droits du **dossier lui‑même**
- `stat fichier` : détails ; mode **octal** rapide (GNU) : `stat -c '%a %n' fichier`

## 6) Bonnes pratiques
- Préférez des changements ciblés (`chmod u+...`) plutôt que de tout écraser avec `=` si vous n’êtes pas sûr.
- Distinguez bien **fichiers** et **dossiers** : le `x` ne signifie pas la même chose.
- Évitez `chmod -R` à l’aveugle : vous pourriez donner l’exécution à des fichiers texte.
