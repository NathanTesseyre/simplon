---
title: "Système de fichiers Linux"
tags:
  - linux
  - filesystem
  - inodes
  - montage
  - fstab
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[utilisateurs_permissions]]
  - [[commandes_base]]
---

# 📘 Système de fichiers Linux

## Tout est fichier

Sous Linux, presque tout est représenté comme un fichier : les données, les répertoires, les périphériques, les sockets, les pipes. Cette abstraction uniforme est un des principes fondateurs du système.

Types de fichiers sous Linux :
- `-` fichier ordinaire (texte, binaire, image...)
- `d` répertoire
- `l` lien symbolique
- `c` périphérique caractère (terminal, clavier)
- `b` périphérique bloc (disque dur, partition)
- `p` pipe nommé (FIFO)
- `s` socket

```bash
ls -la /dev/sda     # b = périphérique bloc (disque)
ls -la /dev/tty     # c = périphérique caractère (terminal)
ls -la /tmp/ma.sock # s = socket
```

## Hiérarchie FHS (Filesystem Hierarchy Standard)

Tous les systèmes Linux suivent une structure de répertoires standardisée :

```
/
├── bin/       → commandes essentielles (ls, cp, bash...)
├── sbin/      → commandes système (root uniquement : fdisk, ip...)
├── etc/       → fichiers de configuration (nginx, ssh, cron...)
├── home/      → répertoires personnels des utilisateurs (/home/alice, /home/bob)
├── root/      → répertoire home de root
├── var/       → données variables (logs, bases de données, mail...)
│   ├── log/   → fichiers de log
│   └── www/   → fichiers web (convention)
├── tmp/       → fichiers temporaires (vidé au reboot)
├── usr/       → programmes et données utilisateurs
│   ├── bin/   → commandes non-essentielles (git, python, vim...)
│   ├── lib/   → bibliothèques partagées
│   └── share/ → données partagées (documentation, icônes...)
├── lib/       → bibliothèques essentielles au démarrage
├── boot/      → noyau Linux, GRUB, initramfs
├── dev/       → périphériques (sda, tty, null, random...)
├── proc/      → système de fichiers virtuel (infos processus et noyau)
├── sys/       → système de fichiers virtuel (infos matériel et noyau)
├── mnt/       → points de montage temporaires
├── media/     → montages automatiques (clés USB, CD...)
└── opt/       → logiciels tiers installés manuellement
```

Quelques fichiers de configuration importants dans `/etc/` :
- `/etc/hostname` — nom de la machine
- `/etc/hosts` — résolution DNS locale
- `/etc/fstab` — montages automatiques au démarrage
- `/etc/passwd` — base des utilisateurs
- `/etc/sudoers` — règles sudo
- `/etc/ssh/sshd_config` — configuration du serveur SSH

## Inodes — comment Linux stocke les métadonnées

Un inode est une structure de données qui contient toutes les métadonnées d'un fichier sauf son nom : type, permissions, propriétaire, taille, dates, et les pointeurs vers les blocs de données sur le disque.

Le nom du fichier est stocké dans le répertoire, qui fait le lien entre le nom et le numéro d'inode.

```
Répertoire :
  "rapport.pdf"  →  inode 42
  "photo.jpg"    →  inode 87

Inode 42 :
  type: fichier ordinaire
  permissions: rw-r--r--
  propriétaire: alice (UID 1000)
  taille: 152430 octets
  créé: 2024-01-15 10:23
  modifié: 2024-01-15 14:00
  accédé: 2024-01-20 09:00
  blocs de données: [4521, 4522, 4523...]
```

```bash
# Voir le numéro d'inode
ls -i fichier.txt
ls -li           # avec les inodes pour tous les fichiers

# Infos complètes d'un fichier (inode + métadonnées)
stat fichier.txt
# File: fichier.txt
# Size: 1234     Blocks: 8     IO Block: 4096   regular file
# Device: sda1   Inode: 1234   Links: 1
# Access: -rw-r--r--  Uid: 1000   Gid: 1000
# Modify: 2024-01-15 14:00:00
```

## Liens durs et liens symboliques

**Lien dur** : crée un deuxième nom pointant vers le même inode. Les deux noms sont équivalents — supprimer l'un ne supprime pas les données tant qu'il reste un lien.

```bash
ln fichier.txt lien-dur.txt       # crée un lien dur
ls -li                             # les deux ont le même numéro d'inode et Links: 2
rm fichier.txt                     # les données sont toujours accessibles via lien-dur.txt
```

Limites des liens durs : impossible entre partitions différentes, impossible sur les répertoires.

**Lien symbolique** : crée un fichier spécial qui pointe vers un chemin. Comme un raccourci Windows.

```bash
ln -s /var/log/nginx lien-sym     # crée un lien symbolique
ls -la lien-sym                   # lien-sym -> /var/log/nginx
ln -s /opt/node-18/bin/node /usr/local/bin/node  # rendre node accessible globalement
```

Usages courants des liens symboliques :
- `/etc/nginx/sites-enabled/monsite` → `../sites-available/monsite` (pattern nginx)
- `/usr/bin/python` → `/usr/bin/python3.11` (version par défaut)
- `/home/alice/.config` → `/mnt/data/config` (externaliser des données)

## Disques, partitions et montages

Sous Linux, un système de fichiers doit être "monté" sur un répertoire pour être accessible.

```bash
# Voir les disques et partitions
lsblk
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0   50G  0 disk
# ├─sda1   8:1    0   49G  0 part /
# └─sda2   8:2    0    1G  0 part [SWAP]
# sdb      8:16   0  500G  0 disk
# └─sdb1   8:17   0  500G  0 part

# Voir les partitions avec leur type (UUID, type FS)
blkid

# Espace disque disponible par partition montée
df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        49G   12G   35G  26% /
# tmpfs           7.8G  100M  7.7G   2% /dev/shm

# Taille occupée par un répertoire
du -sh /var/log
du -sh /var/log/*   # détail par sous-dossier
du -sh /* 2>/dev/null | sort -rh | head -10   # top 10 plus gros répertoires

# Monter une partition (temporairement)
sudo mount /dev/sdb1 /mnt
ls /mnt
sudo umount /mnt    # démonter

# Monter une clé USB
lsblk               # identifier le nom du périphérique (ex: sdc1)
sudo mount /dev/sdc1 /media/usb
sudo umount /media/usb
```

## /etc/fstab — montages automatiques

Pour qu'un système de fichiers soit monté automatiquement au démarrage, il faut l'ajouter dans `/etc/fstab`.

Format : `<périphérique> <point_de_montage> <type_fs> <options> <dump> <pass>`

```
# /etc/fstab
UUID=abc123   /           ext4   defaults          0  1
UUID=def456   /boot/efi   vfat   umask=0077        0  1
UUID=ghi789   /data       ext4   defaults,nofail   0  2
//192.168.1.10/partage  /mnt/nas  cifs  credentials=/etc/samba/creds,nofail  0  0
```

Bonnes pratiques :
- Utiliser les UUID plutôt que `/dev/sdX` (les noms de périphériques peuvent changer)
- Ajouter `nofail` pour les disques non-critiques (le système démarre même si le disque est absent)
- `0 2` en dernier = vérification filesystem au boot (1 = prioritaire pour la partition root)

```bash
# Vérifier que fstab est correct avant de rebooter
sudo mount -a   # monte tout ce qui est dans fstab (erreur si problème)
```

## Systèmes de fichiers courants

| FS | Usage | Caractéristiques |
|----|-------|-----------------|
| **ext4** | Partition Linux principale | Robuste, journalisé, inodes, bon choix par défaut |
| **xfs** | Gros volumes, haute performance | Très performant en écriture, pas de shrink |
| **btrfs** | Snapshots, RAID logiciel | Snapshots natifs, checksums, copie-on-write |
| **vfat/FAT32** | Clés USB, partitions EFI | Universel mais pas de permissions Unix |
| **tmpfs** | RAM filesystem | Tout en mémoire, vidé au reboot (`/tmp`, `/dev/shm`) |
| **NFS** | Partage réseau Unix/Linux | Monter un répertoire distant comme local |
| **CIFS/SMB** | Partage réseau Windows | Compatible Windows, Samba |

## Cas pratiques

**Auditer l'espace disque et trouver les gros fichiers :**
```bash
df -h                          # vue globale par partition
du -sh /var/* | sort -rh       # top des répertoires dans /var
find / -size +100M -not -path "/proc/*" 2>/dev/null   # fichiers > 100 Mo
find /var/log -name "*.log" -size +50M                 # gros fichiers de log
```

**Monter une image ISO :**
```bash
sudo mount -o loop ubuntu.iso /mnt/iso
ls /mnt/iso
sudo umount /mnt/iso
```

**Trouver sur quelle partition est un fichier :**
```bash
df /var/log/syslog    # affiche la partition qui contient ce fichier
```

**Vérifier un système de fichiers (disque démonté) :**
```bash
sudo fsck /dev/sdb1   # vérifier et réparer les erreurs
```

---

### ✅ À retenir

- Tout est fichier sous Linux — répertoires, périphériques, sockets
- La hiérarchie FHS est standardisée : `/etc` = config, `/var` = données variables, `/tmp` = temporaire
- Un inode contient les métadonnées, le répertoire contient le nom
- Lien dur = même inode, deux noms. Lien symbolique = raccourci vers un chemin
- `df -h` = espace par partition. `du -sh` = taille d'un répertoire
- `/etc/fstab` = montages automatiques au démarrage. Utiliser les UUID.

**Voir aussi →** [[utilisateurs_permissions]] pour les permissions sur les fichiers, [[commandes_base]] pour les commandes de navigation.
