---
title: "Firewall — iptables, nftables et UFW"
tags:
  - fondamentaux
  - réseaux
  - firewall
  - iptables
  - sécurité
section: 01-Fondamentaux
domaine: Réseaux
statut: actif
liens_connexes:
  - [[fiche_reseaux]]
  - [[fiche_adressage_ip]]
  - [[fiche_securite_systeme]]
  - [[reseau_linux]]
---

# 🔥 Fiche : Firewall — iptables, nftables et UFW
*(à destination d'administrateurs système en formation)*

---

## 1. Pourquoi un firewall ?

Un **firewall** (pare-feu) filtre le trafic réseau selon des règles définies par l'administrateur.  
Il décide pour chaque paquet : **laisser passer, bloquer ou rejeter**.

### Deux grandes familles
| Type | Description | Exemples |
|---|---|---|
| **Réseau (périmétrique)** | Protège un réseau entier à son entrée | Routeur, appliance dédiée |
| **Hôte (local)** | Protège une machine individuelle | iptables, nftables, ufw |

---

## 2. Architecture iptables

iptables fonctionne avec trois concepts emboîtés : **tables → chaînes → règles**.

### Tables
| Table | Usage |
|---|---|
| `filter` | Filtrage du trafic (**par défaut**) |
| `nat` | Translation d'adresses (NAT, DNAT, SNAT) |
| `mangle` | Modification des paquets (TTL, ToS…) |
| `raw` | Avant le suivi de connexion (conntrack) |

### Chaînes de la table `filter`
```
Paquet entrant
      │
      ▼
  [ PREROUTING ]  ← table nat (redirection de port)
      │
      ├─ Destiné à cette machine ? ──→ [ INPUT ] ──→ Processus local
      │
      └─ En transit ? ──→ [ FORWARD ] ──→ Vers une autre machine
                                │
Paquet sortant                  ▼
      ▲              [ POSTROUTING ] ← table nat (masquerading)
      │
  [ OUTPUT ]  ← trafic généré localement
```

| Chaîne | Trafic concerné |
|---|---|
| `INPUT` | Paquets **à destination** de la machine |
| `OUTPUT` | Paquets **générés par** la machine |
| `FORWARD` | Paquets **transitant** par la machine (routeur) |

### Targets (actions)
| Target | Effet |
|---|---|
| `ACCEPT` | Laisse passer le paquet |
| `DROP` | Jette le paquet silencieusement |
| `REJECT` | Jette et envoie un message d'erreur à l'émetteur |
| `LOG` | Enregistre dans les logs système puis continue |
| `RETURN` | Retourne à la chaîne appelante |

> **DROP vs REJECT** : DROP est furtif (l'émetteur attend le timeout), REJECT est plus clair (répond immédiatement). En sécurité, DROP est souvent préféré pour les attaquants.

---

## 3. Commandes iptables essentielles

```bash
# Lister les règles
iptables -L -v -n                    # table filter
iptables -L -v -n --line-numbers     # avec numéros de ligne
iptables -t nat -L -v -n             # table nat

# Politique par défaut (comportement si aucune règle ne correspond)
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

# Ajouter une règle (append = en fin de chaîne)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # Autoriser SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # Autoriser HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # Autoriser HTTPS

# Insérer en position (insert = numéro de ligne)
iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT

# Supprimer une règle
iptables -D INPUT -p tcp --dport 80 -j ACCEPT    # par contenu
iptables -D INPUT 3                               # par numéro de ligne

# Vider toutes les règles
iptables -F                          # flush (toutes les chaînes)
iptables -F INPUT                    # une seule chaîne
```

---

## 4. Règles courantes pour un serveur

### Configuration de base (serveur web)
```bash
#!/usr/bin/env bash
# Réinitialisation
iptables -F
iptables -X

# Politiques par défaut : tout bloquer en entrée
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Autoriser le loopback (indispensable)
iptables -A INPUT -i lo -j ACCEPT

# Autoriser les connexions déjà établies
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Services
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # HTTPS

# ICMP (ping)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
```

### Restreindre SSH à une IP spécifique
```bash
iptables -A INPUT -p tcp --dport 22 -s 203.0.113.10 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

### Limiter les tentatives de connexion (anti brute-force)
```bash
iptables -A INPUT -p tcp --dport 22 \
  -m state --state NEW \
  -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 \
  -m state --state NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH \
  -j DROP
```

### NAT / Masquerading (partage de connexion)
```bash
# Activer le forwarding IP
echo 1 > /proc/sys/net/ipv4/ip_forward

# Masquerading : les machines du réseau interne sortent avec l'IP de eth0
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# DNAT : rediriger le port 8080 externe vers le port 80 interne
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.10:80
```

---

## 5. Sauvegarder et restaurer les règles

Les règles iptables sont **perdues au redémarrage** si non sauvegardées.

```bash
# Debian/Ubuntu
apt install iptables-persistent
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6
# Les règles sont rechargées automatiquement au démarrage

# RHEL/CentOS
service iptables save           # sauvegarde dans /etc/sysconfig/iptables
iptables-restore < /etc/sysconfig/iptables  # restauration manuelle
```

---

## 6. nftables — le successeur moderne

**nftables** remplace iptables depuis Linux 3.13. Il est plus performant, plus lisible et unifie iptables/ip6tables/arptables/ebtables.

```bash
# Vérifier si nftables est actif
nft list ruleset

# Exemple de configuration complète
nft add table inet filter
nft add chain inet filter input   '{ type filter hook input priority 0 ; policy drop ; }'
nft add chain inet filter output  '{ type filter hook output priority 0 ; policy accept ; }'
nft add chain inet filter forward '{ type filter hook forward priority 0 ; policy drop ; }'

# Règles
nft add rule inet filter input iif lo accept
nft add rule inet filter input ct state established,related accept
nft add rule inet filter input tcp dport 22 accept
nft add rule inet filter input tcp dport { 80, 443 } accept

# Lister les règles
nft list ruleset

# Sauvegarder
nft list ruleset > /etc/nftables.conf
```

---

## 7. UFW — frontend simplifié (Ubuntu/Debian)

**UFW (Uncomplicated Firewall)** est une interface haut niveau qui génère des règles iptables/nftables. Idéal pour les serveurs Ubuntu.

```bash
# Activer/désactiver
ufw enable
ufw disable
ufw status verbose

# Politique par défaut
ufw default deny incoming
ufw default allow outgoing

# Autoriser des services
ufw allow ssh                        # par nom de service
ufw allow 22/tcp                     # par port/protocole
ufw allow 80
ufw allow 443
ufw allow from 203.0.113.10 to any port 22   # SSH depuis une IP spécifique

# Refuser
ufw deny 23                          # telnet

# Supprimer une règle
ufw delete allow 80
ufw delete 3                         # par numéro (ufw status numbered)

# Voir les règles avec numéros
ufw status numbered

# Logs
ufw logging on
tail -f /var/log/ufw.log
```

---

## 8. Ordre des règles — point critique

iptables lit les règles **dans l'ordre**, et s'arrête à la **première qui correspond** (first match wins).

```bash
# MAUVAIS : la règle DROP en premier bloque tout SSH
iptables -A INPUT -p tcp --dport 22 -j DROP
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.1 -j ACCEPT  # jamais atteinte !

# CORRECT : la règle spécifique d'abord
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.1 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

> Règle générale : **du plus spécifique au plus général**.

---

## 9. Diagnostic et débogage

```bash
# Compter les paquets correspondant à chaque règle
iptables -L -v -n

# Activer le logging pour une règle
iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH-ATTEMPT: " --log-level 4
tail -f /var/log/kern.log | grep SSH-ATTEMPT

# Tester la connectivité depuis l'extérieur
nmap -p 22,80,443 mon-serveur.example.com

# Voir les connexions actives
ss -tnp
conntrack -L            # si conntrack installé
```

---

## 10. À retenir pour un admin sys

- **Tables** (filter, nat, mangle) → **chaînes** (INPUT, OUTPUT, FORWARD) → **règles**.
- Toujours autoriser `lo` (loopback) et `ESTABLISHED,RELATED` avant de bloquer.
- Les règles sont **perdues au reboot** : utiliser `iptables-persistent` ou `nftables.conf`.
- L'ordre compte : **spécifique avant général**.
- Sur Ubuntu moderne : **UFW** pour la simplicité, **nftables** pour la puissance.
