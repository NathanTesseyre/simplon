---
title: "Stockage — partitions, LVM, RAID et systèmes de fichiers"
tags:
  - fondamentaux
  - OS
  - stockage
  - LVM
  - RAID
section: 01-Fondamentaux
domaine: Machine & OS
statut: actif
liens_connexes:
  - [[fiche_fonctionnement_ordinateur]]
  - [[fiche_boot_process]]
  - [[systeme_fichiers]]
---

# 💾 Fiche : Stockage — partitions, LVM, RAID et systèmes de fichiers
*(à destination d'administrateurs système en formation)*

---

## 1. Vue d'ensemble des couches de stockage

```
Applications / OS
       │
Système de fichiers  (ext4, XFS, Btrfs…)  — comment les données sont organisées
       │
Volume logique       (LVM)                — découpage flexible
       │
Disque physique / RAID                    — matériel ou logiciel
       │
Support physique     (SSD NVMe, HDD, SAN)
```

---

## 2. Partitions et tables de partition

### MBR vs GPT

| Critère | MBR | GPT |
|---|---|---|
| Taille max disque | 2 To | 9,4 Zo (pratiquement illimité) |
| Partitions primaires | 4 max | 128 max |
| Redondance | Non | Oui (table au début et à la fin) |
| Compatible UEFI | Non (legacy uniquement) | Oui (requis pour UEFI) |

### Commandes de partitionnement
```bash
# Lister les disques et partitions
lsblk                               # vue arborescente
fdisk -l                            # détails (MBR et GPT)
parted -l                           # partitions avec tailles

# Partitionner un disque (interactif)
fdisk /dev/sdb                      # MBR
gdisk /dev/sdb                      # GPT
parted /dev/sdb                     # les deux

# Formater une partition
mkfs.ext4 /dev/sdb1
mkfs.xfs /dev/sdb1
mkswap /dev/sdb2                    # partition swap

# Monter une partition
mount /dev/sdb1 /mnt/donnees
umount /mnt/donnees
```

### /etc/fstab — montage automatique
```bash
# /etc/fstab — format : <périphérique> <point montage> <type> <options> <dump> <pass>
UUID=a1b2c3d4-...  /mnt/donnees  ext4  defaults  0  2

# Trouver l'UUID d'une partition
blkid /dev/sdb1
```

> Toujours utiliser l'**UUID** plutôt que `/dev/sdb1` dans `/etc/fstab` : le nom du périphérique peut changer selon l'ordre de détection.

---

## 3. Systèmes de fichiers

### Comparaison des principaux FS Linux

| FS | Points forts | Usage recommandé |
|---|---|---|
| **ext4** | Stable, mature, bien supporté | Disque OS, usage général |
| **XFS** | Excellent pour gros fichiers et hauts débits | Serveurs de fichiers, BDD |
| **Btrfs** | Snapshots, RAID intégré, compression | NAS, données importantes |
| **tmpfs** | En RAM, volatile | `/tmp`, `/run` |
| **NFS** | Partage réseau | Stockage partagé entre serveurs |

### Opérations courantes
```bash
# Vérifier l'espace disque
df -h                               # par système de fichiers
df -h /var/log                      # pour un chemin précis

# Espace utilisé par un répertoire
du -sh /var/log
du -sh /* | sort -h                 # tri par taille

# Vérifier/réparer un FS (à faire hors montage)
umount /dev/sdb1
fsck /dev/sdb1
e2fsck -f /dev/sdb1                 # ext4

# Agrandir un FS ext4 (après agrandissement de la partition)
resize2fs /dev/sdb1

# Agrandir un FS XFS (à chaud, une fois monté)
xfs_growfs /mnt/donnees

# Inodes (nombre de fichiers)
df -i                               # utilisation des inodes
```

---

## 4. LVM — Logical Volume Manager

LVM ajoute une couche d'abstraction entre les disques physiques et les systèmes de fichiers. Il permet de **redimensionner les volumes à chaud** et de **combiner plusieurs disques**.

### Concepts

```
PV (Physical Volume)  →  VG (Volume Group)  →  LV (Logical Volume)
[/dev/sdb]                [vg_data]              [lv_logs]
[/dev/sdc]                                        [lv_db]
```

| Terme | Définition |
|---|---|
| **PV** (Physical Volume) | Un disque ou une partition initialisé pour LVM |
| **VG** (Volume Group) | Pool de stockage regroupant plusieurs PV |
| **LV** (Logical Volume) | Volume utilisable, formaté et monté |
| **PE** (Physical Extent) | Unité d'allocation (4 Mo par défaut) |

### Mise en place LVM
```bash
# 1. Initialiser les disques comme PV
pvcreate /dev/sdb /dev/sdc
pvs                                 # lister les PV

# 2. Créer un VG
vgcreate vg_data /dev/sdb /dev/sdc
vgs                                 # lister les VG

# 3. Créer des LV
lvcreate -L 50G -n lv_logs vg_data
lvcreate -L 100G -n lv_db vg_data
lvcreate -l 100%FREE -n lv_backup vg_data   # utiliser tout l'espace restant
lvs                                 # lister les LV

# 4. Formater et monter
mkfs.ext4 /dev/vg_data/lv_logs
mount /dev/vg_data/lv_logs /var/log
```

### Opérations courantes LVM
```bash
# Agrandir un LV (+ 20 Go)
lvextend -L +20G /dev/vg_data/lv_logs
resize2fs /dev/vg_data/lv_logs      # agrandir le FS ext4

# En une seule commande (LV + FS)
lvextend -L +20G -r /dev/vg_data/lv_logs

# Réduire un LV (risqué — faire un backup avant)
umount /dev/vg_data/lv_logs
e2fsck -f /dev/vg_data/lv_logs
resize2fs /dev/vg_data/lv_logs 30G
lvreduce -L 30G /dev/vg_data/lv_logs

# Ajouter un disque au VG
pvcreate /dev/sdd
vgextend vg_data /dev/sdd

# Snapshot LVM (sauvegarde cohérente)
lvcreate -L 5G -s -n lv_logs_snap /dev/vg_data/lv_logs
mount -o ro /dev/vg_data/lv_logs_snap /mnt/snap    # monter en lecture seule
lvremove /dev/vg_data/lv_logs_snap                  # supprimer le snapshot

# Voir toute la configuration LVM
pvdisplay
vgdisplay
lvdisplay
```

---

## 5. RAID — Redondance et performances

Le **RAID (Redundant Array of Independent Disks)** combine plusieurs disques physiques pour la **tolérance aux pannes** ou les **performances**.

### Niveaux RAID courants

| Niveau | Description | Disques min | Tolérance pannes | Capacité utile | Usage |
|---|---|---|---|---|---|
| **RAID 0** | Striping (répartition) | 2 | Aucune | 100% | Performances, pas de prod critique |
| **RAID 1** | Mirroring (miroir) | 2 | 1 disque | 50% | OS, petits volumes critiques |
| **RAID 5** | Striping + parité distribuée | 3 | 1 disque | (N-1)/N | Serveurs fichiers, équilibre |
| **RAID 6** | Striping + double parité | 4 | 2 disques | (N-2)/N | Stockage critique |
| **RAID 10** | RAID 1+0 (miroir + striping) | 4 | 1 par miroir | 50% | BDD haute dispo + perf |

```
RAID 0 (striping) :    [A1][A2]   [B1][B2]   → Rapide, zéro tolérance
                       Disque 1   Disque 2

RAID 1 (miroir) :      [AAAA]     [AAAA]     → Si disque 1 tombe, disque 2 prend le relais
                       Disque 1   Disque 2

RAID 5 (parité) :      [A][B][P]  [C][P][A]  [P][C][B]  → Perd un disque, reconstruit
                       Disque 1   Disque 2   Disque 3
```

### RAID logiciel Linux (mdadm)
```bash
# Créer un RAID 1 avec deux disques
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Voir l'état du RAID
cat /proc/mdstat
mdadm --detail /dev/md0

# Sauvegarder la configuration
mdadm --detail --scan >> /etc/mdadm/mdadm.conf

# Simuler une panne et remplacer un disque
mdadm --fail /dev/md0 /dev/sdb        # marquer comme défaillant
mdadm --remove /dev/md0 /dev/sdb      # retirer
mdadm --add /dev/md0 /dev/sdd         # ajouter le nouveau disque
```

> En production, le **RAID matériel** (contrôleur dédié) est préférable : il gère la reconstruction en fond de tâche sans impacter les performances du CPU.

---

## 6. Swap

Le swap est une zone disque utilisée comme extension de la RAM.

```bash
# Créer un fichier swap (alternative à une partition)
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# Vérifier le swap actif
swapon --show
free -h

# Ajouter dans /etc/fstab pour la persistance
/swapfile  none  swap  sw  0  0

# Paramètre swappiness (0 = évite le swap, 100 = swap agressif)
cat /proc/sys/vm/swappiness          # valeur actuelle (60 par défaut)
sysctl vm.swappiness=10              # changer temporairement
echo "vm.swappiness=10" >> /etc/sysctl.conf  # persistant
```

---

## 7. À retenir pour un admin sys

- **Toujours utiliser UUID** dans `/etc/fstab` — jamais `/dev/sdX`.
- **LVM** = flexibilité maximale pour redimensionner sans interruption.
- **RAID 1** pour la sécurité, **RAID 5/6** pour l'équilibre, **RAID 10** pour la performance + sécurité.
- RAID ne remplace **pas** les sauvegardes : il protège contre les pannes disque, pas contre les suppressions accidentelles.
- Avant tout `fsck` ou réduction de volume : **backup + démontage**.
