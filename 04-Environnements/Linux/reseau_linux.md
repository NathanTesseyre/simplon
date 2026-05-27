---
title: "Réseau sous Linux"
tags:
  - linux
  - réseau
  - ip
  - ss
  - firewall
  - ssh
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[fiche_reseaux]]
  - [[commandes_base]]
  - [[services_demarrage]]
---

# 📘 Réseau sous Linux

## Interfaces et adresses

```bash
# Voir toutes les interfaces et leurs adresses IP
ip addr
ip addr show eth0     # interface spécifique
ip -br a              # format condensé (brief)

# État des interfaces (up/down)
ip link
ip link show eth0

# Activer / désactiver une interface
sudo ip link set eth0 up
sudo ip link set eth0 down

# Ajouter une adresse IP temporairement
sudo ip addr add 192.168.1.50/24 dev eth0
sudo ip addr del 192.168.1.50/24 dev eth0
```

**Noms d'interfaces courants :**
- `lo` — loopback (127.0.0.1), interface virtuelle locale
- `eth0`, `enp3s0` — ethernet (câble)
- `wlan0`, `wlp2s0` — WiFi
- `docker0` — bridge Docker
- `tun0` — interface VPN

## Connectivité et diagnostic

```bash
# Tester la connectivité
ping -c 4 1.1.1.1          # 4 paquets vers Cloudflare DNS
ping -c 4 google.com       # test DNS + connectivité

# Résolution DNS
getent hosts google.com    # résoudre via /etc/nsswitch.conf
dig google.com             # requête DNS détaillée (apt install dnsutils)
dig google.com A           # enregistrement A uniquement
dig @8.8.8.8 google.com   # utiliser Google DNS comme resolver
nslookup google.com        # alternative à dig

# Voir les serveurs DNS configurés
resolvectl status
cat /etc/resolv.conf

# Tracer la route des paquets
traceroute google.com
tracepath google.com       # alternative sans root
mtr google.com             # traceroute continu, temps réel (apt install mtr)
```

## Routage

```bash
# Voir la table de routage
ip route
ip route show

# La passerelle par défaut (default gateway)
ip route | grep default
# exemple : default via 192.168.1.1 dev eth0

# Ajouter une route
sudo ip route add 10.0.0.0/8 via 192.168.1.1
sudo ip route del 10.0.0.0/8

# Changer la passerelle par défaut
sudo ip route replace default via 192.168.1.254
```

## Ports et connexions actives

`ss` est le remplaçant moderne de `netstat`.

```bash
# Voir tous les ports en écoute (listening)
ss -tlnp
# -t = TCP, -l = listening, -n = pas de résolution DNS, -p = processus

# Ports UDP en écoute
ss -ulnp

# Toutes les connexions TCP établies
ss -tnp

# Tout (listening + établies)
ss -tulnp

# Qui écoute sur le port 80 ?
ss -tlnp | grep :80

# Qui écoute sur le port 5432 (PostgreSQL) ?
ss -tlnp | grep :5432

# Combien de connexions établies ?
ss -tn | grep ESTAB | wc -l

# Connexions vers une IP spécifique
ss -tn dst 1.2.3.4
```

**Lecture d'une ligne `ss -tlnp` :**
```
State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:80          0.0.0.0:*          users:(("nginx",pid=1234))
```
- `0.0.0.0:80` = écoute sur toutes les interfaces, port 80
- `127.0.0.1:3000` = écoute uniquement en local (pas accessible de l'extérieur)

## SSH — connexion distante

```bash
# Connexion basique
ssh user@192.168.1.10
ssh user@server.example.com

# Port non standard
ssh -p 2222 user@server.example.com

# Avec une clé privée spécifique
ssh -i ~/.ssh/ma_cle user@server.example.com

# Générer une paire de clés SSH
ssh-keygen -t ed25519 -C "mon-commentaire"
# Crée ~/.ssh/id_ed25519 (privée) et ~/.ssh/id_ed25519.pub (publique)

# Copier sa clé publique sur un serveur
ssh-copy-id user@server.example.com
# Ajoute la clé dans ~/.ssh/authorized_keys sur le serveur

# Tunnel SSH (port forwarding local)
ssh -L 8080:localhost:80 user@server.example.com
# → localhost:8080 sur ta machine = port 80 du serveur distant

# Tunnel inverse (exposer un port local via un serveur distant)
ssh -R 9090:localhost:3000 user@server.example.com
# → port 9090 du serveur = port 3000 de ta machine locale

# Exécuter une commande sans ouvrir de shell
ssh user@server.example.com "df -h && free -h"
```

**Config SSH client** (`~/.ssh/config`) pour éviter de retaper les options :
```
Host monserveur
    HostName 192.168.1.10
    User admin
    Port 2222
    IdentityFile ~/.ssh/ma_cle

Host prod
    HostName prod.example.com
    User deploy
    IdentityFile ~/.ssh/prod_key
```
Ensuite : `ssh monserveur` au lieu de `ssh -p 2222 -i ~/.ssh/ma_cle admin@192.168.1.10`

## Pare-feu avec ufw

`ufw` (Uncomplicated Firewall) est le frontend convivial pour `nftables`/`iptables` sur Ubuntu/Debian.

```bash
# État du pare-feu
sudo ufw status
sudo ufw status verbose

# Activer / désactiver
sudo ufw enable
sudo ufw disable

# Politique par défaut (toujours définir avant d'activer)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Autoriser des services
sudo ufw allow ssh          # port 22
sudo ufw allow 80/tcp       # HTTP
sudo ufw allow 443/tcp      # HTTPS
sudo ufw allow 5432/tcp     # PostgreSQL

# Autoriser depuis une IP spécifique seulement
sudo ufw allow from 192.168.1.0/24 to any port 5432

# Refuser un port
sudo ufw deny 23/tcp        # bloquer Telnet

# Supprimer une règle
sudo ufw delete allow 80/tcp
sudo ufw delete allow ssh

# Voir les règles numérotées
sudo ufw status numbered
sudo ufw delete 3           # supprimer la règle n°3
```

## Configuration réseau persistante

Les paramètres définis avec `ip` sont temporaires (perdus au reboot). Pour les rendre persistants :

**Sur Ubuntu (Netplan)** — fichiers dans `/etc/netplan/` :
```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```
```bash
sudo netplan apply
```

**Sur Debian (interfaces)** — `/etc/network/interfaces` :
```
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1
```

## Cas pratiques

**Trouver quel processus écoute sur un port :**
```bash
ss -tlnp | grep :3000
# ou
sudo lsof -i :3000
```

**Tester si un port distant est accessible :**
```bash
nc -zv 192.168.1.10 5432    # PostgreSQL accessible ?
nc -zv google.com 443       # HTTPS accessible ?
# -z = scan sans envoyer de données, -v = verbose
```

**Diagnostiquer une connexion lente :**
```bash
mtr google.com   # voir où les paquets ralentissent ou sont perdus
```

**Voir toute l'activité réseau d'un processus :**
```bash
sudo strace -e trace=network -p PID
# ou avec nethogs (sudo apt install nethogs)
sudo nethogs eth0
```

---

### ✅ À retenir

- `ip addr` = voir les IPs des interfaces (remplace `ifconfig`)
- `ss -tlnp` = voir les ports en écoute avec le processus associé
- `ping` + `traceroute` + `mtr` = diagnostic de connectivité
- `dig` = interroger le DNS manuellement
- `ufw` = pare-feu simple sur Ubuntu/Debian
- SSH avec clés = standard de sécurité, jamais de mot de passe en production
- `~/.ssh/config` = économise du temps, rend SSH confortable

**Voir aussi →** [[fiche_reseaux]] pour les concepts (OSI, TCP/IP, DNS), [[services_demarrage]] pour gérer le service SSH.
