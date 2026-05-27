---
title: "SSH avancé"
tags:
  - ssh
  - linux
  - réseau
  - sécurité
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[reseau_linux]]
  - [[utilisateurs_permissions]]
  - [[fiche_securite_systeme]]
---

# 📘 Fiche : SSH avancé

---

## 1. Rappel — Comment fonctionne SSH ?

SSH (Secure Shell) ouvre un tunnel chiffré entre un client et un serveur pour exécuter des commandes à distance.

```
Client (ton poste)  →  [tunnel chiffré SSH]  →  Serveur distant
```

**Authentification possible :**
- Par mot de passe (déconseillé en production)
- Par clé (recommandé) : paire clé privée (client) + clé publique (serveur)

---

## 2. Générer et gérer ses clés SSH

### Générer une clé
```bash
ssh-keygen -t ed25519 -C "commentaire@exemple.com"
# Fichiers créés :
# ~/.ssh/id_ed25519      → clé privée (ne jamais partager)
# ~/.ssh/id_ed25519.pub  → clé publique (à déposer sur les serveurs)
```

> Préférer **ed25519** à RSA — plus court, plus sûr, plus rapide.

### Déposer la clé publique sur un serveur
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@serveur
# Ou manuellement : ajouter le contenu de .pub dans ~/.ssh/authorized_keys sur le serveur
```

### SSH Agent — éviter de retaper la passphrase
```bash
eval "$(ssh-agent -s)"   # démarre l'agent
ssh-add ~/.ssh/id_ed25519  # charge la clé dans l'agent
ssh-add -l               # liste les clés chargées
```

---

## 3. Fichier de configuration `~/.ssh/config`

Ce fichier permet de définir des alias et des options par hôte — évite les longues commandes SSH.

```ssh-config
# ~/.ssh/config

# Serveur de prod
Host prod
  HostName 203.0.113.10
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
  Port 2222

# Serveur de dev
Host dev
  HostName 203.0.113.20
  User admin
  IdentityFile ~/.ssh/id_ed25519

# Options globales
Host *
  ServerAliveInterval 60
  ServerAliveCountMax 3
```

Utilisation :
```bash
ssh prod          # au lieu de : ssh -i ~/.ssh/id_ed25519 -p 2222 ubuntu@203.0.113.10
```

---

## 4. Jump Host (Bastion)

Un **jump host** (ou bastion) est un serveur intermédiaire pour atteindre des machines non exposées directement sur Internet.

```
Client → Bastion (IP publique) → Serveur interne (IP privée)
```

```bash
# Connexion via un bastion
ssh -J user@bastion user@serveur-interne

# Dans ~/.ssh/config
Host serveur-interne
  HostName 10.0.1.50
  User ubuntu
  ProxyJump bastion

Host bastion
  HostName 203.0.113.5
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
```

---

## 5. Tunnels SSH

### Tunnel local (Local Port Forwarding)
Redirige un port local vers un port distant via SSH.

```bash
ssh -L 8080:localhost:80 user@serveur
# → http://localhost:8080 sur ton poste accède au port 80 du serveur
```

**Cas d'usage** : accéder à une interface web (pgAdmin, Grafana) sur un serveur sans l'exposer.

### Tunnel distant (Remote Port Forwarding)
Expose un port local sur le serveur distant.

```bash
ssh -R 9090:localhost:3000 user@serveur
# → le port 9090 du serveur redirige vers le port 3000 de ton poste
```

### Tunnel dynamique (SOCKS Proxy)
Crée un proxy SOCKS5 pour router tout le trafic via SSH.

```bash
ssh -D 1080 user@serveur
# → configurer le navigateur pour utiliser SOCKS5 localhost:1080
```

---

## 6. Copier des fichiers avec SCP et rsync

```bash
# SCP — copie simple
scp fichier.txt user@serveur:/chemin/destination/
scp -r dossier/ user@serveur:/chemin/destination/

# rsync — synchronisation (plus efficace, reprend si coupure)
rsync -avz dossier/ user@serveur:/chemin/destination/
rsync -avz --delete dossier/ user@serveur:/chemin/  # supprime les fichiers absents en local
```

---

## 7. Bonnes pratiques

- Utiliser des clés **ed25519**, jamais de mot de passe en production.
- Définir une **passphrase** sur la clé privée + utiliser `ssh-agent`.
- Toujours passer par un **bastion** pour accéder aux serveurs internes.
- Ne jamais copier la clé privée sur un serveur distant.
- Configurer `~/.ssh/config` pour éviter les erreurs de frappe sur les longues commandes.
- Restreindre les accès SSH côté serveur (`sshd_config`) → voir [[fiche_securite_systeme]].

---

## 8. ✅ À retenir

- **ed25519** = standard pour les clés SSH aujourd'hui.
- `~/.ssh/config` = gain de temps majeur pour gérer plusieurs serveurs.
- **Jump host** = accéder aux serveurs internes via un bastion.
- **Tunnel local** = exposer un service distant en local sans ouvrir de port.
- `ssh-agent` = ne taper la passphrase qu'une fois par session.
