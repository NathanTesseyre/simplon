---
title: "Utilisateurs et permissions"
tags:
  - linux
  - permissions
  - sudo
  - groupes
  - chmod
  - chown
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[fiche_droits_autorisations]]
  - [[fiche_theorique_permissions_base]]
  - [[systeme_fichiers]]
---

# 📘 Utilisateurs et permissions

## Le modèle de sécurité Unix

Linux est un système multi-utilisateurs : plusieurs personnes (ou services) peuvent utiliser la machine simultanément. Chaque ressource (fichier, processus, socket) appartient à un utilisateur et un groupe, et un système de permissions contrôle qui peut faire quoi.

Trois entités :
- **Utilisateur (user/owner)** : la personne ou le service propriétaire du fichier
- **Groupe (group)** : un ensemble d'utilisateurs partageant des droits communs
- **Autres (others)** : tout le monde qui n'est ni owner ni dans le groupe

## Fichiers d'identité

```bash
# Qui suis-je ?
whoami          # alice
id              # uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo),998(docker)

# Fichiers système
cat /etc/passwd   # liste des utilisateurs : login:x:UID:GID:commentaire:home:shell
cat /etc/group    # liste des groupes : groupe:x:GID:membres
# Les mots de passe hashés sont dans /etc/shadow (root uniquement)
```

Format de `/etc/passwd` :
```
alice:x:1000:1000:Alice Martin:/home/alice:/bin/bash
nginx:x:33:33::/var/www:/usr/sbin/nologin
```
Le shell `/usr/sbin/nologin` ou `/bin/false` indique un compte de service sans accès interactif.

## Gérer les utilisateurs et groupes

```bash
# Créer un utilisateur
sudo useradd -m alice             # -m crée le répertoire home
sudo useradd -m -s /bin/bash alice   # spécifier le shell
sudo useradd -m -G docker,www-data alice  # avec des groupes supplémentaires

# Définir/changer le mot de passe
sudo passwd alice

# Modifier un utilisateur
sudo usermod -aG docker alice     # ajouter au groupe docker (-a = append, -G = groupes)
sudo usermod -s /bin/zsh alice    # changer le shell
sudo usermod -l nouveau_login alice  # changer le login

# Supprimer un utilisateur
sudo userdel alice               # supprimer sans toucher au home
sudo userdel -r alice            # supprimer avec le répertoire home

# Gérer les groupes
sudo groupadd devteam             # créer un groupe
sudo groupdel devteam             # supprimer
sudo gpasswd -a alice devteam     # ajouter alice au groupe
sudo gpasswd -d alice devteam     # retirer alice du groupe
groups alice                      # voir les groupes d'alice
```

**Important :** après `usermod -aG`, l'utilisateur doit se reconnecter pour que les nouveaux groupes soient effectifs. Pour les appliquer dans la session courante :
```bash
newgrp docker    # activer le groupe docker dans le shell courant
```

## Permissions Unix — lecture

Chaque fichier a un ensemble de permissions lisible avec `ls -l` :

```
-rw-r--r--  1  alice  web  1234  jan 15 10:00  rapport.txt
drwxr-xr-x  2  alice  web  4096  jan 15 09:00  docs/
lrwxrwxrwx  1  root   root   11  jan 10 08:00  logs -> /var/log
```

Décoder les 10 premiers caractères :
```
- r w - r - - r - -
│ │─│ │─│─│ │─│─│
│ │ │ │   │     └── others : r-- = lecture seule
│ │ │ └───┘─────── group   : r-- = lecture seule
│ └─┘─────────────  user   : rw- = lecture + écriture
└────────────────── type : - = fichier, d = dossier, l = lien
```

Les trois droits :
| Droit | Fichier | Répertoire |
|-------|---------|-----------|
| `r` (read, 4) | Lire le contenu | Lister les fichiers (`ls`) |
| `w` (write, 2) | Modifier le contenu | Créer/supprimer des fichiers |
| `x` (execute, 1) | Exécuter comme programme | Entrer dans le répertoire (`cd`) |

## chmod — modifier les permissions

```bash
# Notation symbolique
chmod u+x script.sh    # ajouter x pour le user (owner)
chmod g-w fichier      # retirer w pour le group
chmod o-r fichier      # retirer r pour others
chmod a+r fichier      # ajouter r pour tous (all)
chmod u+x,g-w fichier  # plusieurs changements en une fois

# Notation octale (plus rapide une fois mémorisée)
# Chaque groupe (u/g/o) = somme des droits : r=4, w=2, x=1
chmod 755 script.sh    # rwxr-xr-x : owner=7(rwx), group=5(r-x), others=5(r-x)
chmod 644 rapport.txt  # rw-r--r-- : owner=6(rw-), group=4(r--), others=4(r--)
chmod 600 .ssh/id_rsa  # rw------- : owner uniquement (clé privée SSH)
chmod 777 /tmp/dossier # rwxrwxrwx : tous les droits pour tous (dangereux)
chmod 000 secret       # --------- : aucun droit pour personne

# Valeurs octales courantes à retenir
# 755 → exécutables, scripts, répertoires publics
# 644 → fichiers de config, pages web
# 600 → fichiers privés (clés SSH, .env)
# 700 → répertoires privés

# Récursif (appliquer à un répertoire et tout son contenu)
chmod -R 755 /var/www/html
```

## chown et chgrp — changer le propriétaire

```bash
# Changer le propriétaire
sudo chown alice fichier.txt
sudo chown alice:web fichier.txt    # owner alice, groupe web
sudo chown :web fichier.txt         # changer seulement le groupe

# Changer le groupe
sudo chgrp web fichier.txt

# Récursif
sudo chown -R alice:alice /home/alice
sudo chown -R www-data:www-data /var/www
```

## umask — permissions par défaut

L'umask définit les permissions retirées automatiquement à la création de nouveaux fichiers et dossiers.

```bash
umask        # voir le umask actuel (souvent 022)
umask 022    # définir temporairement

# Calcul :
# Fichier : permissions max = 666 (pas de x par défaut)
# 666 - 022 = 644 → rw-r--r-- (fichier créé par défaut)

# Répertoire : permissions max = 777
# 777 - 022 = 755 → rwxr-xr-x (répertoire créé par défaut)

# umask 027 → fichiers en 640, répertoires en 750 (plus restrictif)
```

## sudo — élever ses privilèges

```bash
# Exécuter une commande en tant que root
sudo apt update
sudo systemctl restart nginx

# Ouvrir un shell root
sudo -i     # shell root complet (avec l'environnement de root)
sudo -s     # shell root (avec l'environnement courant)

# Exécuter en tant qu'un autre utilisateur
sudo -u alice commande
sudo -u www-data php script.php

# Voir ce que l'on peut faire avec sudo
sudo -l

# Vérifier que sudo fonctionne (rafraîchit le ticket sans commande)
sudo -v
```

**Configurer sudo** (`sudo visudo` — ne jamais éditer `/etc/sudoers` directement) :
```
# Syntaxe : utilisateur  hôte=(user_cible)  commandes
alice   ALL=(ALL:ALL)   ALL                # alice peut tout faire
bob     ALL=(ALL)       /usr/bin/apt       # bob peut uniquement apt
www-data ALL=(root)     NOPASSWD: /usr/sbin/service nginx *  # sans mot de passe

# Groupe (préfixé par %)
%sudo   ALL=(ALL:ALL)   ALL               # tous les membres de sudo
%docker ALL=(ALL)       NOPASSWD: /usr/bin/docker
```

Fichiers dans `/etc/sudoers.d/` : bonne pratique pour ajouter des règles sans modifier le fichier principal.

## Bits spéciaux

**SetUID (s sur user)** : le programme s'exécute avec les droits du propriétaire, pas de l'appelant.
```bash
ls -la /usr/bin/passwd
# -rwsr-xr-x root root /usr/bin/passwd
# Le "s" = setuid : passwd tourne en root pour écrire dans /etc/shadow
chmod u+s programme   # activer setuid
```

**SetGID (s sur group)** : sur un répertoire, les fichiers créés dedans héritent du groupe du répertoire.
```bash
chmod g+s /var/www/html
# Tout fichier créé dans ce dossier aura le groupe www-data (utile pour le web)
```

**Sticky bit (t)** : sur un répertoire, seul le propriétaire d'un fichier peut le supprimer.
```bash
ls -la /tmp
# drwxrwxrwt ... /tmp   ← le "t" = sticky bit
chmod +t /dossier-partage
# Tout le monde peut créer des fichiers, personne ne peut supprimer ceux des autres
```

## Cas pratiques

**Préparer un répertoire web partagé :**
```bash
sudo mkdir /var/www/monsite
sudo chown -R alice:www-data /var/www/monsite
sudo chmod -R 775 /var/www/monsite   # alice et www-data peuvent tout écrire
sudo chmod g+s /var/www/monsite      # nouveaux fichiers héritent du groupe www-data
```

**Sécuriser un fichier .env :**
```bash
chmod 600 .env        # lisible uniquement par le propriétaire
chown alice:alice .env
```

**Donner accès à Docker sans sudo :**
```bash
sudo usermod -aG docker alice
newgrp docker   # ou se reconnecter
docker ps       # fonctionne sans sudo
```

---

### ✅ À retenir

- `ls -l` → lire les permissions (type + user + group + others)
- `chmod 755` script, `chmod 644` fichier, `chmod 600` fichier privé
- `chown user:group fichier` → changer le propriétaire
- `usermod -aG groupe user` → ajouter un utilisateur à un groupe (reconnecter pour que ce soit effectif)
- `sudo -l` → voir ce qu'on peut faire avec sudo
- SetUID = programme tourne avec les droits du propriétaire
- Sticky bit sur `/tmp` = protection contre la suppression par autrui

**Voir aussi →** [[fiche_theorique_permissions_base]] pour les exercices, [[fiche_droits_autorisations]] pour la gestion des droits au niveau applicatif.
