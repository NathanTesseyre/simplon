---
title: "Boot process — du démarrage à systemd"
tags:
  - fondamentaux
  - OS
  - boot
  - systemd
section: 01-Fondamentaux
domaine: Machine & OS
statut: actif
liens_connexes:
  - [[fiche_fonctionnement_ordinateur]]
  - [[fiche_processus_threads]]
  - [[services_demarrage]]
---

# ⚙️ Fiche : Boot process — du démarrage à systemd
*(à destination d'administrateurs système en formation)*

---

## 1. Vue d'ensemble : les 5 étapes

```
[Alimentation] → [BIOS/UEFI] → [Bootloader GRUB] → [Noyau Linux] → [systemd] → [Login]
```

Chaque étape passe le contrôle à la suivante. Un problème de démarrage se diagnostique en identifiant **à quelle étape ça bloque**.

---

## 2. BIOS vs UEFI

| Critère | BIOS (legacy) | UEFI (moderne) |
|---|---|---|
| Stockage config | Puce CMOS + pile | Mémoire NVRAM flash |
| Table de partition | MBR (max 2 To, 4 partitions primaires) | GPT (max 9,4 Zo, 128 partitions) |
| Interface | Texte uniquement | Graphique, souris possible |
| Secure Boot | Non | Oui (vérifie la signature du bootloader) |
| Architecture | 16 bits au démarrage | 64 bits natif |

### POST (Power-On Self Test)
Au démarrage, le firmware exécute le POST :
1. Vérifie la RAM, le CPU, les bus.
2. Détecte les périphériques de stockage.
3. Cherche un support de démarrage dans l'ordre configuré (disque, USB, réseau PXE...).
4. Charge le **bootloader** depuis ce support.

> En UEFI, la partition de démarrage est une partition FAT32 dédiée : l'**ESP (EFI System Partition)**, montée sous `/boot/efi`.

---

## 3. Schéma des partitions de démarrage

### MBR (BIOS legacy)
```
[ MBR — 512 octets ]
  └─ Stage 1 bootloader (446 octets)
  └─ Table de partitions (64 octets)
  └─ Signature (2 octets : 0x55AA)
```

### GPT (UEFI)
```
[ ESP — partition FAT32 ]
  └─ /EFI/ubuntu/grubx64.efi  ← fichier exécuté par l'UEFI
[ Partition /boot ]
  └─ vmlinuz-*     ← image du noyau
  └─ initrd.img-*  ← système de fichiers initial
[ Partition / (root) ]
```

---

## 4. GRUB2 — le bootloader

GRUB2 (GRand Unified Bootloader) est le bootloader standard sur Linux.

### Rôle
- Affiche un menu de démarrage (choix du noyau, mode recovery...).
- Charge le **noyau** (`vmlinuz`) et l'**initramfs** (`initrd.img`) en mémoire.
- Passe des paramètres au noyau (root device, quiet, splash...).

### Fichiers clés
| Fichier | Rôle |
|---|---|
| `/boot/grub/grub.cfg` | Configuration générée (ne pas éditer directement) |
| `/etc/default/grub` | Paramètres à modifier (timeout, options noyau) |
| `/etc/grub.d/` | Scripts qui génèrent grub.cfg |

### Commandes utiles
```bash
# Régénérer grub.cfg après modification de /etc/default/grub
update-grub                          # Debian/Ubuntu
grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS

# Réinstaller GRUB sur un disque (ex : après remplacement disque)
grub-install /dev/sda

# Vérifier la version
grub-install --version
```

### Mode recovery GRUB
Si le système ne démarre plus, au menu GRUB :
- `e` → éditer les paramètres de démarrage.
- Ajouter `init=/bin/bash` à la ligne `linux` → démarre directement un shell root.
- `c` → console GRUB (commandes manuelles).

---

## 5. Noyau Linux + initramfs

### vmlinuz
- Image compressée du noyau Linux.
- Nommée `vmlinuz-<version>` (ex. `vmlinuz-6.1.0-21-amd64`).
- GRUB la charge en RAM et lui passe le contrôle.

### initramfs (Initial RAM Filesystem)
- Système de fichiers temporaire chargé **avant** le vrai `/`.
- Contient les drivers indispensables (stockage, LVM, chiffrement...).
- Monte le vrai système de fichiers racine, puis le noyau bascule vers lui (`pivot_root`).

```bash
# Voir le contenu d'un initramfs
lsinitramfs /boot/initrd.img-$(uname -r)

# Régénérer l'initramfs
update-initramfs -u                  # Debian/Ubuntu
dracut --force                       # RHEL/CentOS
```

---

## 6. systemd — le processus init (PID 1)

Une fois le système de fichiers racine monté, le noyau lance **systemd** (PID 1). C'est lui qui démarre tous les services.

### Concepts clés

| Terme | Définition |
|---|---|
| **Unit** | Unité de configuration (service, socket, timer, mount...) |
| **Target** | Groupe d'units correspondant à un état du système |
| **Dependency** | Un unit peut `Require=`, `Want=` ou `After=` un autre |

### Targets principales
| Target | Équivalent runlevel | Usage |
|---|---|---|
| `poweroff.target` | 0 | Extinction |
| `rescue.target` | 1 | Mode maintenance (root seul) |
| `multi-user.target` | 3 | Texte multi-utilisateur (serveurs) |
| `graphical.target` | 5 | Interface graphique |
| `reboot.target` | 6 | Redémarrage |

```bash
# Voir la target active
systemctl get-default

# Changer la target par défaut (ex : serveur sans GUI)
systemctl set-default multi-user.target

# Basculer vers rescue sans redémarrer
systemctl isolate rescue.target
```

### Commandes systemctl essentielles
```bash
# Gérer un service
systemctl start|stop|restart|reload nginx
systemctl enable|disable nginx        # activer/désactiver au démarrage
systemctl status nginx

# Lister les services
systemctl list-units --type=service --state=running
systemctl list-units --failed          # services en erreur

# Voir les dépendances d'un service
systemctl list-dependencies nginx

# Analyser le temps de démarrage
systemd-analyze
systemd-analyze blame                  # temps par service
systemd-analyze critical-chain         # chemin critique
```

### Lire les logs de démarrage
```bash
# Logs du boot actuel
journalctl -b

# Logs du boot précédent
journalctl -b -1

# Logs d'un service spécifique
journalctl -u ssh --since "1 hour ago"

# Suivre en temps réel
journalctl -f
```

---

## 7. Séquence complète commentée

```
1. Alimentation
   └─ CPU démarre, exécute le firmware depuis la ROM

2. BIOS/UEFI
   └─ POST (test matériel)
   └─ Détecte les disques, cherche l'ESP ou le MBR
   └─ Charge grubx64.efi (UEFI) ou le stage 1 MBR (BIOS)

3. GRUB2
   └─ Affiche le menu
   └─ Charge vmlinuz + initrd.img en RAM
   └─ Passe les paramètres noyau (root=/dev/sda2, quiet...)

4. Noyau Linux
   └─ Se décompresse et initialise le matériel (CPU, mémoire, drivers)
   └─ Monte initramfs comme / temporaire
   └─ Charge les drivers pour accéder au vrai disque
   └─ Monte le vrai /
   └─ Lance /sbin/init → systemd (PID 1)

5. systemd
   └─ Lit les units dans /lib/systemd/system/ et /etc/systemd/system/
   └─ Démarre les services en parallèle selon les dépendances
   └─ Atteint la target par défaut (ex : multi-user.target)

6. Login
   └─ getty lance un invite de connexion (TTY) ou display manager (GUI)
```

---

## 8. Diagnostiquer un problème de démarrage

| Symptôme | Étape probable | Action |
|---|---|---|
| Écran noir immédiat, pas de GRUB | BIOS/UEFI ou MBR/GPT | Vérifier l'ordre de boot, réinstaller GRUB |
| Menu GRUB s'affiche puis erreur | GRUB → noyau | Vérifier grub.cfg, chemins vmlinuz/initrd |
| `Kernel panic — not syncing` | Noyau ou initramfs | Régénérer initramfs, vérifier UUID du disque root |
| `Failed to mount /` | Noyau → montage root | Vérifier /etc/fstab, UUID correct |
| Services qui échouent | systemd | `systemctl list-units --failed`, `journalctl -b` |

---

## 9. À retenir pour un admin sys

- **BIOS/MBR** = legacy, limité ; **UEFI/GPT** = standard actuel sur tout matériel récent.
- GRUB se reconfigure avec `update-grub` après modification de `/etc/default/grub`.
- Le noyau lance **systemd PID 1** qui démarre tout le reste.
- `systemd-analyze blame` révèle les services qui rallongent le démarrage.
- `journalctl -b` est le premier réflexe pour diagnostiquer un échec au boot.
