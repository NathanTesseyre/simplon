---
title: "Adressage IP et sous-réseaux (CIDR)"
tags:
  - fondamentaux
  - réseaux
  - IP
  - CIDR
  - sous-réseaux
section: 01-Fondamentaux
domaine: Réseaux
statut: actif
liens_connexes:
  - [[fiche_reseaux]]
  - [[reseau_linux]]
  - [[fiche_architecture_infra]]
---

# 🌐 Fiche : Adressage IP et sous-réseaux (CIDR)
*(à destination d'administrateurs système en formation)*

---

## 1. Qu'est-ce qu'une adresse IP ?

Une **adresse IPv4** est un identifiant unique attribué à chaque interface réseau.  
Elle est composée de **32 bits**, notée en **décimale pointée** : 4 octets séparés par des points.

```
Adresse :    192     .    168    .     1     .    42
Binaire :  11000000  . 10101000 . 00000001 . 00101010
```

Chaque octet varie de **0 à 255** (2⁸ = 256 valeurs).

---

## 2. Le masque de sous-réseau

Le masque divise l'adresse IP en deux parties :
- **Partie réseau** → identifie le réseau (bits à 1 dans le masque).
- **Partie hôte** → identifie la machine sur ce réseau (bits à 0 dans le masque).

### Exemple avec `192.168.1.42 / 255.255.255.0`

```
Adresse IP :  192.168.1.42   →  11000000.10101000.00000001.00101010
Masque :      255.255.255.0  →  11111111.11111111.11111111.00000000
                                |←——— réseau (24 bits) ———→|← hôte →|
```

- **Réseau** : `192.168.1.0`
- **Broadcast** : `192.168.1.255`
- **Plage d'hôtes** : `192.168.1.1` → `192.168.1.254` (254 machines)

---

## 3. Notation CIDR

La notation **CIDR (Classless Inter-Domain Routing)** remplace le masque par un simple suffixe : le **nombre de bits à 1**.

| Notation longue | Notation CIDR | Équivalence |
|---|---|---|
| `255.255.255.0` | `/24` | 24 bits réseau |
| `255.255.0.0` | `/16` | 16 bits réseau |
| `255.0.0.0` | `/8` | 8 bits réseau |
| `255.255.255.128` | `/25` | 25 bits réseau |

> `192.168.1.0/24` se lit : "réseau `192.168.1.0` avec un masque de 24 bits".

---

## 4. Tableau de référence CIDR

| Préfixe | Masque | Nb hôtes utiles | Usage typique |
|---|---|---|---|
| `/8` | `255.0.0.0` | 16 777 214 | Grands blocs (classe A historique) |
| `/16` | `255.255.0.0` | 65 534 | Réseaux d'entreprise |
| `/24` | `255.255.255.0` | 254 | LAN standard, VLAN |
| `/25` | `255.255.255.128` | 126 | Découpage d'un /24 en 2 |
| `/26` | `255.255.255.192` | 62 | Petits segments |
| `/27` | `255.255.255.224` | 30 | DMZ, petits groupes |
| `/28` | `255.255.255.240` | 14 | Très petits segments |
| `/30` | `255.255.255.252` | 2 | Liens point à point (routeurs) |
| `/32` | `255.255.255.255` | 1 | Hôte unique (loopback, routes statiques) |

> **Formule** : nombre d'hôtes = **2^(32−préfixe) − 2** (−2 pour l'adresse réseau et le broadcast)

---

## 5. Calcul d'un sous-réseau

### Exemple : `10.0.5.75/27`

**Étape 1 — Taille du bloc**  
`/27` → 32 − 27 = 5 bits pour les hôtes → **2⁵ = 32 adresses** par bloc.

**Étape 2 — Trouver le bloc contenant `10.0.5.75`**  
Diviser le 4ème octet par 32 : `75 ÷ 32 = 2` reste `11` → bloc numéro 2.  
Le bloc commence à `2 × 32 = 64`.

**Résultat :**
| Élément | Valeur |
|---|---|
| Adresse réseau | `10.0.5.64` |
| Broadcast | `10.0.5.95` |
| Plage d'hôtes | `10.0.5.65` → `10.0.5.94` |
| Nombre d'hôtes | 30 |

---

## 6. Adresses spéciales et réservées

### Plages privées (RFC 1918) — non routables sur Internet

| Plage | Notation CIDR | Usage |
|---|---|---|
| `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` | Grandes entreprises, datacenters |
| `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` | Réseaux moyens (Docker par défaut) |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` | Réseaux domestiques, labs |

### Autres adresses réservées

| Adresse | Usage |
|---|---|
| `127.0.0.1` (`127.0.0.0/8`) | Loopback — la machine elle-même |
| `0.0.0.0` | Adresse non spécifiée (écoute sur toutes les interfaces) |
| `169.254.x.x` (`169.254.0.0/16`) | APIPA — attribution automatique sans DHCP |
| `255.255.255.255` | Broadcast limité (toutes les machines du réseau local) |
| `224.0.0.0/4` | Multicast |

---

## 7. IPv6 — les bases

IPv6 utilise des adresses de **128 bits**, notées en **hexadécimal** séparé par `:`.

```
IPv6 complet  :  2001:0db8:0000:0000:0000:ff00:0042:8329
IPv6 abrégé   :  2001:db8::ff00:42:8329
```

### Règles d'abréviation
- Supprimer les zéros de tête dans chaque groupe : `0db8` → `db8`.
- Remplacer **une** séquence de groupes nuls consécutifs par `::`.

### Adresses courantes

| Adresse | Équivalent IPv4 | Usage |
|---|---|---|
| `::1` | `127.0.0.1` | Loopback |
| `fe80::/10` | `169.254.x.x` | Link-local (non routable) |
| `fc00::/7` | RFC 1918 | Privé (ULA — Unique Local Address) |
| `2000::/3` | Adresses publiques | Internet global |

> Avec IPv6, pas besoin de NAT : chaque machine peut avoir une adresse publique unique. Le sous-réseau standard alloué est un **/64**.

---

## 8. NAT — pourquoi on en a besoin (IPv4)

Le nombre d'adresses IPv4 publiques est **épuisé** (4,3 milliards en tout).  
Le **NAT (Network Address Translation)** permet à tout un réseau privé de partager **une seule IP publique**.

```
[ PC 1 : 192.168.1.10 ]  ─┐
[ PC 2 : 192.168.1.11 ]  ─┤─→ [ Routeur NAT : 203.0.113.1 ] ─→ Internet
[ PC 3 : 192.168.1.12 ]  ─┘
          (IP privées)              (IP publique)
```

Le routeur maintient une **table de translation** (IP privée + port → IP publique + port).

---

## 9. Commandes pratiques (Linux)

```bash
# Voir les interfaces et adresses IP
ip addr show
ip -4 addr                          # IPv4 uniquement
ip -6 addr                          # IPv6 uniquement

# Voir la table de routage
ip route show

# Ajouter une adresse IP temporaire
ip addr add 192.168.1.100/24 dev eth0

# Calculer un sous-réseau (outil ipcalc)
ipcalc 10.0.5.75/27

# Scanner les hôtes actifs sur un réseau (nmap)
nmap -sn 192.168.1.0/24

# Vérifier la connectivité
ping -c 4 192.168.1.1

# Tracer le chemin réseau
traceroute 8.8.8.8
```

### Exemple de sortie `ipcalc`
```
Address:   10.0.5.75            00001010.00000000.00000101. 01001011
Netmask:   255.255.255.224 = 27 11111111.11111111.11111111. 11100000
Network:   10.0.5.64/27         00001010.00000000.00000101. 01000000
HostMin:   10.0.5.65            00001010.00000000.00000101. 01000001
HostMax:   10.0.5.94            00001010.00000000.00000101. 01011110
Broadcast: 10.0.5.95            00001010.00000000.00000101. 01011111
Hosts/Net: 30
```

---

## 10. À retenir pour un admin sys

- Une IP seule ne suffit pas : il faut toujours connaître le **préfixe CIDR** pour savoir à quel réseau elle appartient.
- Les plages **10.x.x.x**, **172.16-31.x.x** et **192.168.x.x** sont privées → jamais routées directement sur Internet.
- `/24` = 254 hôtes, `/30` = 2 hôtes (liens point à point), `/32` = 1 hôte.
- `ip addr` et `ip route` sont les commandes de référence (remplacent `ifconfig`).
- `ipcalc` évite les erreurs de calcul manuel en production.
