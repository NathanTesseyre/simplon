---
title: "Introduction à Linux"
tags:
  - linux
  - OS
  - shell
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[commandes_base]]
  - [[systeme_fichiers]]
---

# 📘 Fiche : Introduction à Linux

**Sommaire**
- [Qu’est-ce que Linux ?](#sec-quest)
- [Philosophie UNIX](#sec-unix)
- [Différences avec Windows / macOS](#sec-diff)
- [Pourquoi c’est important pour un développeur ?](#sec-pourquoi)
- [Cas pratiques](#sec-cas)
- [✅ À retenir](#sec-ret)

<a id="sec-quest"></a>
## Qu’est-ce que Linux ?
- **Noyau Linux** : cœur du système, gère CPU, mémoire, périphériques et planification des processus.  
- **Distribution** : noyau + outils + gestionnaire de paquets + configuration et éventuelle interface graphique.  
  - Exemples : **Ubuntu, Debian, Fedora, Arch Linux, CentOS/Alma/Rocky**.

Linux n’est pas un produit unique mais une **famille** de systèmes bâtis autour du noyau.

<a id="sec-unix"></a>
## Philosophie UNIX
- **« Tout est fichier »** : périphériques (`/dev`), processus (`/proc`), sockets, tout est manipulable comme un fichier.  
- **Petits outils composables** : chaque commande fait une chose, **pipes** (`|`) pour les enchaîner.  
- **Text is the universal interface** : formats texte, configuration lisible.

💡 Exemple :
```bash
cat /var/log/syslog | grep "ERROR" | less
```

<a id="sec-diff"></a>
## Différences avec Windows / macOS
- **CLI** omniprésente et puissante (bash, zsh, fish).  
- **Ouverture** (open source), forte **personnalisation**.  
- **Prépondérance côté serveurs et cloud**, desktop selon les distributions.

<a id="sec-pourquoi"></a>
## Pourquoi c’est important pour un développeur ?
- La majorité des serveurs web et services cloud tournent sous Linux.  
- Outils modernes (Docker, Kubernetes, Git, CI/CD) y sont **natifs**.  
- Comprendre Linux = comprendre les **environnements de prod**.

<a id="sec-cas"></a>
## Cas pratiques
1. Distribution installée :
```bash
cat /etc/os-release
```
2. Version du noyau :
```bash
uname -r
```
3. Architecture CPU & infos système :
```bash
uname -a
```

<a id="sec-ret"></a>
## ✅ À retenir
- Linux = noyau + distributions.  
- Héritage UNIX : **fichiers + pipelines + outils simples**.  
- Incontournable pour dev backend, cloud et ops.
