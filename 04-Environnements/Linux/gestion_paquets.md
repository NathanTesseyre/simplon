---
title: "Gestion des paquets"
tags:
  - linux
  - apt
  - paquets
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[intro_linux]]
  - [[services_demarrage]]
  - [[shell_scripting]]
  - [[fiche_securite_systeme]]
---

# 📘 Fiche : Gestion des paquets

**Sommaire**
- [Gestionnaires par familles](#gpkg-fam)
- [apt (Debian/Ubuntu)](#gpkg-apt)
- [dnf/yum (RedHat/Fedora)](#gpkg-dnf)
- [pacman (Arch)](#gpkg-pacman)
- [Formats universels](#gpkg-univ)
- [Cas pratiques](#gpkg-cas)
- [✅ À retenir](#gpkg-ret)

<a id="gpkg-fam"></a>
## Gestionnaires par familles
Debian/Ubuntu → `apt` (+ `dpkg`)  
RedHat/Fedora → `dnf`/`yum` (+ `rpm`)  
Arch → `pacman`

<a id="gpkg-apt"></a>
## apt (Debian/Ubuntu)
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx
sudo apt remove nginx && sudo apt autoremove
apt search motcle
dpkg -l | grep nginx
```

<a id="gpkg-dnf"></a>
## dnf/yum (RedHat/Fedora)
```bash
sudo dnf check-update && sudo dnf upgrade -y
sudo dnf install nginx
sudo dnf remove nginx
dnf list installed | grep nginx
```

<a id="gpkg-pacman"></a>
## pacman (Arch)
```bash
sudo pacman -Syu
sudo pacman -S nginx
sudo pacman -R nginx
```

<a id="gpkg-univ"></a>
## Formats universels
**Snap**, **Flatpak**, **AppImage**.

<a id="gpkg-cas"></a>
## Cas pratiques
1. Installer `htop`, l’exécuter, le supprimer.  
2. Rechercher quel paquet fournit `curl`.  
3. Mettre à jour le système et nettoyer.

<a id="gpkg-ret"></a>
## ✅ À retenir
Cycle : **update → upgrade → install/remove → cleanup**.
