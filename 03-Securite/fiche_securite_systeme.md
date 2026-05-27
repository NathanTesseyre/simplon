---
title: "Sécurité système Linux"
tags:
  - sécurité
  - linux
  - firewall
  - hardening
  - TLS
section: 03-Securite
domaine: Sécurité
statut: actif
liens_connexes:
  - [[fiche_securite_dev]]
  - [[ssh_avance]]
  - [[utilisateurs_permissions]]
  - [[services_demarrage]]
---

# 📘 Fiche : Sécurité système Linux

> Cette fiche couvre la sécurisation du **système et de l'infrastructure**.  
> Pour la sécurité applicative (XSS, SQLi, OWASP...) → [[fiche_securite_dev]]

---

## 1. Firewall — ufw et iptables

### ufw (Uncomplicated Firewall) — recommandé pour débuter
Interface simplifiée au-dessus d'iptables. Adapté aux serveurs classiques.

```bash
# Activer ufw
ufw enable

# Politique par défaut : bloquer tout entrant, autoriser tout sortant
ufw default deny incoming
ufw default allow outgoing

# Autoriser des services
ufw allow ssh          # port 22
ufw allow 80/tcp       # HTTP
ufw allow 443/tcp      # HTTPS
ufw allow 2222/tcp     # SSH sur port custom

# Autoriser depuis une IP spécifique
ufw allow from 203.0.113.5 to any port 22

# Vérifier les règles
ufw status verbose

# Supprimer une règle
ufw delete allow 80/tcp
```

### iptables — outil bas niveau
Plus puissant mais plus complexe. ufw génère des règles iptables automatiquement.

```bash
# Voir les règles actuelles
iptables -L -n -v

# Bloquer une IP
iptables -A INPUT -s 203.0.113.50 -j DROP

# Autoriser le port 443
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Sauvegarder les règles (Debian/Ubuntu)
apt install iptables-persistent
netfilter-persistent save
```

> 💡 En production : préférer ufw pour la simplicité, ou nftables (successeur d'iptables).

---

## 2. fail2ban — protection contre le brute force

**fail2ban** surveille les logs système et bannit automatiquement les IPs qui font trop de tentatives échouées (SSH, HTTP, etc.).

### Installation
```bash
apt install fail2ban
systemctl enable --now fail2ban
```

### Configuration
```ini
# /etc/fail2ban/jail.local  (ne pas modifier jail.conf directement)
[DEFAULT]
bantime  = 1h        # durée du ban
findtime = 10m       # fenêtre de temps observée
maxretry = 5         # tentatives avant ban

[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
```

```bash
systemctl restart fail2ban

# Vérifier les bans actifs
fail2ban-client status sshd

# Débanner une IP
fail2ban-client set sshd unbanip 203.0.113.50
```

---

## 3. Hardening SSH (`sshd_config`)

Le fichier `/etc/ssh/sshd_config` contrôle le comportement du serveur SSH.

```bash
# /etc/ssh/sshd_config

Port 2222                        # changer le port par défaut (réduction du bruit)
PermitRootLogin no               # interdire la connexion root directe
PasswordAuthentication no        # forcer l'authentification par clé
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

MaxAuthTries 3                   # tentatives max avant déconnexion
LoginGraceTime 30                # délai max pour s'authentifier (secondes)
AllowUsers ubuntu deploy         # liste blanche d'utilisateurs autorisés

X11Forwarding no                 # désactiver si inutile
AllowTcpForwarding no            # désactiver si les tunnels ne sont pas nécessaires
```

```bash
# Vérifier la syntaxe avant de redémarrer
sshd -t

# Appliquer
systemctl restart ssh
```

> ⚠️ Toujours garder une session SSH ouverte pendant les modifications pour ne pas se bloquer.

---

## 4. Certificats TLS avec Let's Encrypt (Certbot)

**Let's Encrypt** fournit des certificats TLS gratuits via le protocole ACME.  
**Certbot** est le client officiel pour les obtenir et les renouveler automatiquement.

### Installation (Debian/Ubuntu)
```bash
apt install certbot python3-certbot-nginx
```

### Obtenir un certificat (avec Nginx)
```bash
certbot --nginx -d mondomaine.com -d www.mondomaine.com
```

### Obtenir un certificat (standalone — sans serveur web)
```bash
# Arrêter le serveur web si port 80 occupé
certbot certonly --standalone -d mondomaine.com
```

### Certificats générés
```
/etc/letsencrypt/live/mondomaine.com/
├── fullchain.pem   → certificat + chaîne (à utiliser dans Nginx/Apache)
├── privkey.pem     → clé privée
├── cert.pem        → certificat seul
└── chain.pem       → chaîne intermédiaire
```

### Renouvellement automatique
Certbot installe un timer systemd qui renouvelle automatiquement.

```bash
# Tester le renouvellement
certbot renew --dry-run

# Vérifier le timer
systemctl status certbot.timer
```

---

## 5. Mises à jour de sécurité automatiques

```bash
# Debian/Ubuntu — activer les mises à jour de sécurité automatiques
apt install unattended-upgrades
dpkg-reconfigure unattended-upgrades

# Vérifier la configuration
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

---

## 6. Audit rapide de sécurité

```bash
# Ports ouverts et services associés
ss -tlnp

# Connexions actives
ss -tnp

# Utilisateurs avec accès sudo
getent group sudo

# Dernières connexions
last -n 20
lastb -n 20   # tentatives échouées

# Processus qui écoutent sur le réseau
ss -tlnp | grep LISTEN
```

---

## 7. Bonnes pratiques

- **Ne jamais exposer SSH sur le port 22** en production — utiliser un port non standard + fail2ban.
- **Désactiver PasswordAuthentication** — clés uniquement.
- **Principe du moindre privilège** : chaque service tourne sous son propre utilisateur sans droits sudo.
- **Mettre à jour régulièrement** : `apt upgrade` et activer les mises à jour de sécurité automatiques.
- **Firewall dès l'installation** : n'ouvrir que les ports nécessaires.
- **Surveiller les logs** : `/var/log/auth.log`, `/var/log/syslog`, `journalctl -u ssh`.

---

## 8. ✅ À retenir

- **ufw** = firewall simple et efficace pour la majorité des cas.
- **fail2ban** = protection automatique contre le brute force.
- **sshd_config** = désactiver root + mot de passe, changer le port.
- **Certbot + Let's Encrypt** = HTTPS gratuit et renouvellement automatique.
- Audit rapide : `ss -tlnp`, `last`, `lastb`.
