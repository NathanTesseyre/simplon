---
title: "DNS avancé — résolution, zones et enregistrements"
tags:
  - fondamentaux
  - réseaux
  - DNS
  - infrastructure
section: 01-Fondamentaux
domaine: Réseaux
statut: actif
liens_connexes:
  - [[fiche_reseaux]]
  - [[fiche_adressage_ip]]
  - [[reseau_linux]]
---

# 🌍 Fiche : DNS avancé — résolution, zones et enregistrements
*(à destination d'administrateurs système en formation)*

---

## 1. Rappel : à quoi sert le DNS ?

Le **DNS (Domain Name System)** traduit des noms lisibles (`www.example.com`) en adresses IP (`93.184.216.34`).  
Sans DNS, il faudrait connaître l'adresse IP de chaque service — impossible à l'échelle d'Internet.

---

## 2. Architecture DNS : hiérarchie et délégation

```
                     . (root)
                    / \
                  .com  .fr  .org  .io  ...
                  /
             example.com
               /     \
          www       mail     api
```

Chaque niveau est géré par un **serveur autoritaire** différent.  
La **délégation** permet à chaque zone de confier la gestion de ses sous-domaines à d'autres serveurs.

### Types de serveurs DNS
| Type | Rôle |
|---|---|
| **Serveur racine** | 13 groupes de serveurs (a.root-servers.net… m.root-servers.net), point de départ de toute résolution |
| **Serveur TLD** | Gère un domaine de premier niveau (.com, .fr…) |
| **Serveur autoritaire** | Détient la vérité pour une zone donnée (ex : le DNS de votre hébergeur) |
| **Résolveur récursif** | Interroge les autres serveurs pour le compte du client (ex : 8.8.8.8, 1.1.1.1) |
| **Cache DNS** | Conserve les réponses pour éviter de refaire les requêtes (TTL) |

---

## 3. Résolution récursive — étape par étape

**Question : quelle est l'IP de `www.example.com` ?**

```
Client (PC)
  │  1. "Quelle est l'IP de www.example.com ?"
  ▼
Résolveur récursif (ex : 8.8.8.8)
  │  2. "Je ne sais pas, je vais chercher"
  │
  │  3. Interroge un serveur racine : "Qui gère .com ?"
  ▼
Serveur racine
  │  4. "c.gtld-servers.net gère .com"
  ▼
Résolveur récursif
  │  5. Interroge c.gtld-servers.net : "Qui gère example.com ?"
  ▼
Serveur TLD .com
  │  6. "ns1.example.com gère example.com"
  ▼
Résolveur récursif
  │  7. Interroge ns1.example.com : "Quelle est l'IP de www.example.com ?"
  ▼
Serveur autoritaire example.com
  │  8. "www.example.com → 93.184.216.34 (TTL: 3600)"
  ▼
Résolveur récursif
  │  9. Met en cache la réponse, la renvoie au client
  ▼
Client
  └─  10. Reçoit 93.184.216.34, se connecte
```

> Toute cette chaîne prend en général **< 100 ms**. Grâce au cache, les requêtes suivantes sont instantanées.

---

## 4. Les enregistrements DNS (Resource Records)

### Enregistrements essentiels

| Type | Rôle | Exemple |
|---|---|---|
| **A** | Nom → IPv4 | `www.example.com. 3600 IN A 93.184.216.34` |
| **AAAA** | Nom → IPv6 | `www.example.com. 3600 IN AAAA 2606:2800::1` |
| **CNAME** | Alias vers un autre nom | `blog.example.com. CNAME www.example.com.` |
| **MX** | Serveur de messagerie | `example.com. MX 10 mail.example.com.` |
| **NS** | Serveurs DNS autoritaires de la zone | `example.com. NS ns1.example.com.` |
| **TXT** | Texte libre (SPF, DKIM, vérification…) | `example.com. TXT "v=spf1 include:sendgrid.net ~all"` |
| **SOA** | Start of Authority — métadonnées de la zone | voir ci-dessous |
| **PTR** | IP → Nom (DNS inverse) | `34.216.184.93.in-addr.arpa. PTR www.example.com.` |
| **SRV** | Localisation d'un service (protocole, port, priorité) | `_http._tcp.example.com. SRV 10 5 80 www.example.com.` |
| **CAA** | Autorités de certification autorisées | `example.com. CAA 0 issue "letsencrypt.org"` |

### Détail des enregistrements courants

#### A et AAAA
```
; Un domaine peut avoir plusieurs A (round-robin DNS = répartition de charge basique)
www.example.com.  300  IN  A  93.184.216.34
www.example.com.  300  IN  A  93.184.216.35
```

#### CNAME — règles importantes
```
; Un CNAME pointe vers un nom, jamais vers une IP
blog.example.com.  3600  IN  CNAME  www.example.com.

; INTERDIT : un CNAME ne peut pas coexister avec d'autres enregistrements
; (sauf DNSSEC). Jamais de CNAME sur l'apex (@) si vous avez un MX ou NS !
```

#### MX — messagerie
```
; Priorité : plus le chiffre est bas, plus le serveur est prioritaire
example.com.  3600  IN  MX  10  mail1.example.com.
example.com.  3600  IN  MX  20  mail2.example.com.  ; secours
```

#### TXT — usages courants
```
; SPF : liste les serveurs autorisés à envoyer des mails pour ce domaine
example.com.  TXT  "v=spf1 ip4:203.0.113.0/24 include:sendgrid.net -all"

; DKIM : clé publique pour signature des mails
default._domainkey.example.com.  TXT  "v=DKIM1; k=rsa; p=MIGfMA0G..."

; DMARC : politique anti-spoofing
_dmarc.example.com.  TXT  "v=DMARC1; p=reject; rua=mailto:dmarc@example.com"

; Vérification de propriété (Google, Let's Encrypt...)
example.com.  TXT  "google-site-verification=abc123xyz"
```

#### SOA — métadonnées de zone
```
example.com.  IN  SOA  ns1.example.com.  hostmaster.example.com. (
    2024010101  ; Serial (YYYYMMDDNN) — incrémenté à chaque modification
    3600        ; Refresh — fréquence de vérification par les secondaires
    900         ; Retry — délai si échec du refresh
    604800      ; Expire — durée avant que les secondaires abandonnent
    300         ; Minimum TTL (TTL négatif)
)
```

#### PTR — DNS inverse
```
; Utilisé pour vérifier qu'une IP correspond bien au nom déclaré (anti-spam)
; La zone inverse est gérée par le fournisseur de l'IP (hébergeur, FAI)
34.216.184.93.in-addr.arpa.  PTR  www.example.com.
```

---

## 5. Le TTL (Time To Live)

Le **TTL** indique en secondes combien de temps un résolveur peut mettre la réponse en cache.

| TTL | Usage recommandé |
|---|---|
| `300` (5 min) | Avant une migration (permet de changer rapidement) |
| `3600` (1 h) | Valeur standard pour la plupart des enregistrements |
| `86400` (24 h) | Enregistrements stables (MX, NS) |

> **Avant de changer une IP** : abaisser le TTL à 300 au moins 48h avant, effectuer le changement, puis remonter le TTL.

---

## 6. Résolution locale (Linux)

L'ordre de résolution des noms est défini dans `/etc/nsswitch.conf` :

```bash
# /etc/nsswitch.conf
hosts:  files dns myhostname
#        │     │
#        │     └─ interroge les serveurs dans /etc/resolv.conf
#        └─ consulte /etc/hosts en premier
```

### Fichiers clés

```bash
# /etc/hosts — résolution locale statique (prioritaire)
127.0.0.1    localhost
192.168.1.10 mon-serveur mon-serveur.local

# /etc/resolv.conf — serveurs DNS à interroger
nameserver 8.8.8.8
nameserver 1.1.1.1
search     example.com    # suffixe ajouté aux noms courts
```

### systemd-resolved (Ubuntu moderne)
```bash
# Vérifier le résolveur actif
resolvectl status

# Vérifier quelle interface utilise quel DNS
resolvectl dns

# Vider le cache DNS local
resolvectl flush-caches

# Diagnostiquer une résolution
resolvectl query www.example.com
```

---

## 7. Outils de diagnostic

```bash
# dig — outil principal (le plus complet)
dig www.example.com                  # enregistrement A
dig www.example.com AAAA             # enregistrement AAAA
dig example.com MX                   # serveurs mail
dig example.com TXT                  # enregistrements TXT
dig example.com NS                   # serveurs DNS autoritaires
dig example.com SOA                  # SOA de la zone

# Interroger un serveur DNS spécifique
dig @8.8.8.8 www.example.com         # via Google DNS
dig @1.1.1.1 www.example.com         # via Cloudflare

# DNS inverse (PTR)
dig -x 93.184.216.34

# Tracer toute la chaîne de résolution
dig +trace www.example.com

# Réponse courte (juste l'IP)
dig +short www.example.com

# nslookup (interactif ou one-shot)
nslookup www.example.com
nslookup www.example.com 8.8.8.8

# host (simplifié)
host www.example.com
host -t MX example.com
```

### Exemple de sortie `dig +trace`
```
.                       518400  IN  NS  a.root-servers.net.
com.                    172800  IN  NS  a.gtld-servers.net.
example.com.            172800  IN  NS  ns1.example.com.
www.example.com.        3600    IN  A   93.184.216.34
```

---

## 8. DNS Split-Horizon

Le **split-horizon** permet de retourner des réponses différentes selon l'origine de la requête.

```
Depuis Internet       → www.example.com → 203.0.113.10 (IP publique)
Depuis le réseau LAN  → www.example.com → 192.168.1.10 (IP privée)
```

Utilisé pour éviter le **hairpinning** (sortir sur Internet pour accéder à un serveur local) et pour des raisons de sécurité (ne pas exposer les IP internes).

---

## 9. DNSSEC — intégrité des réponses

**DNSSEC** ajoute des signatures cryptographiques aux enregistrements DNS pour garantir qu'ils n'ont pas été altérés (protection contre le DNS spoofing / cache poisoning).

```bash
# Vérifier si un domaine est signé DNSSEC
dig +dnssec www.example.com
dig example.com DNSKEY

# Vérifier la chaîne DNSSEC complète
dig +sigchase www.example.com    # nécessite dig avec SIGCHASE
```

| Enregistrement DNSSEC | Rôle |
|---|---|
| `DNSKEY` | Clé publique de la zone |
| `RRSIG` | Signature d'un enregistrement |
| `DS` | Délégation de signature (lien entre zones) |
| `NSEC/NSEC3` | Preuve de non-existence d'un enregistrement |

---

## 10. À retenir pour un admin sys

- La résolution DNS est **hiérarchique** : root → TLD → autoritaire → client.
- Les enregistrements à maîtriser en priorité : **A, CNAME, MX, TXT, NS, PTR**.
- **TTL bas avant une migration**, TTL élevé pour la stabilité.
- `dig +trace` est l'outil de référence pour diagnostiquer une résolution complète.
- Sur Linux, l'ordre de résolution est dans `/etc/nsswitch.conf` : `files` avant `dns` → `/etc/hosts` est prioritaire.
