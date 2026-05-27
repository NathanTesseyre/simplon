---
title: "Introduction aux réseaux"
tags:
  - fondamentaux
  - réseaux
  - TCP/IP
  - HTTP
section: 01-Fondamentaux
domaine: Réseaux
statut: actif
liens_connexes:
  - [[fiche_architecture_infra]]
  - [[reseau_linux]]
  - [[fiche_serveurs_web]]
---

# 🌐 Fiche détaillée : Introduction aux réseaux informatiques  
*(à destination de développeurs en formation)*  

---

## 1. Pourquoi les réseaux ?
Un réseau permet à plusieurs machines de **communiquer et échanger des données**.  
Sans réseau, chaque ordinateur fonctionnerait **seul, isolé** → impossible d’accéder à des services distants.  

### ✨ Concrètement, les réseaux permettent de :  
- **Partager des ressources** → fichiers, imprimantes, bases de données.  
- **Communiquer** → e-mails, messageries instantanées, appels vidéo.  
- **Accéder à des services** → sites web, applications en ligne, APIs.  
- **Collaborer en temps réel** → travail en équipe sur GitHub, Google Docs, Figma.  
- **Interconnecter des systèmes** → microservices, architectures distribuées, IoT.  

### 🌍 Exemple dans la vie quotidienne d’un développeur :  
- Installer une dépendance avec `npm install` ou `pip install` → va chercher un package sur un serveur distant.  
- Récupérer du code sur GitHub avec `git clone`.  
- Faire tourner une API qui communique avec une base de données distante.  
- Tester une application web depuis ton navigateur → ton PC dialogue avec un serveur parfois situé à l’autre bout du monde.  

👉 En résumé :  
Un réseau, c’est ce qui **relie les machines et les humains**. Sans réseaux, pas d’Internet, pas d’applications distribuées, pas de cloud… et le métier de développeur serait très différent 😉  

---

## 2. Les modèles de communication

Quand deux ordinateurs communiquent à travers un réseau, il ne suffit pas d’envoyer des “0” et des “1” au hasard.  
👉 Il faut un **langage commun et des règles partagées** pour que chacun comprenne l’autre.  

Pour cela, on utilise des **modèles en couches** :  
- Chaque couche a un rôle précis (comme une usine avec plusieurs postes de travail).  
- Chaque couche **parle uniquement à la couche voisine**, ce qui simplifie la conception et le débogage.  
- Un problème réseau peut être isolé par couche :  
  - si le Wi-Fi tombe → problème couche 2 (liaison),  
  - si le site renvoie une erreur → problème couche 7 (application).  

Deux modèles servent de référence :  
- **OSI (7 couches)** → modèle théorique, surtout pédagogique.  
- **TCP/IP (4 couches)** → modèle simplifié, utilisé réellement sur Internet.  

---

### 🔹 Modèle OSI (7 couches – théorique)
| Couche | Rôle | Exemples |
|--------|------|----------|
| 7. Application | Interaction avec l’utilisateur | HTTP, FTP, SMTP |
| 6. Présentation | Format, chiffrement | TLS, SSL |
| 5. Session | Connexion entre applications | gestion login/session |
| 4. Transport | Fiabilité ou rapidité | TCP, UDP |
| 3. Réseau | Routage des paquets | IP |
| 2. Liaison | Transmission locale | Ethernet, Wi-Fi |
| 1. Physique | Support matériel | câble, fibre, ondes radio |

---

### 🔹 Modèle TCP/IP (4 couches – pratique, Internet)
| Couche TCP/IP | Rôle | Protocoles |
|---------------|------|------------|
| Application   | Les services utilisés | HTTP, DNS, SMTP |
| Transport     | Fiabilité & ports | TCP, UDP |
| Internet      | Adressage & routage | IP |
| Accès réseau  | Transmission locale | Ethernet, Wi-Fi |

👉 **OSI = pédagogique**, **TCP/IP = réalité**.  

---

## 3. Les protocoles essentiels

### 📌 IP (Internet Protocol)
- Identifie chaque machine avec une **adresse IP** (ex. `192.168.1.12` en IPv4).  
- Sert à **acheminer les paquets** entre machines, même à travers des dizaines de routeurs.  

### 📌 TCP (Transmission Control Protocol)
- **Fiable** : garantit que les données arrivent, dans le bon ordre.  
- Fonctionne comme une **conversation téléphonique** → “Tu m’as entendu ? Oui.”  
- Utilisé par HTTP, FTP, SSH.  

### 📌 UDP (User Datagram Protocol)
- **Rapide** mais sans garantie (pas de vérification).  
- Comme envoyer une **carte postale** → pas sûr qu’elle arrive.  
- Utilisé par streaming, jeux en ligne, DNS.  

### 📌 Ports (boîtes aux lettres du réseau)
- Une **IP** identifie une machine, un **port** identifie un service.  
- Exemples :  
  - `80` → HTTP  
  - `443` → HTTPS  
  - `22` → SSH  
  - `3306` → MySQL  
- Métaphore : **IP = adresse de la maison, Port = numéro de boîte aux lettres**.  

### 📌 DNS (Domain Name System)
- Traduit un **nom de domaine** (`www.google.com`) en **adresse IP**.  
- Fonctionne comme un **annuaire téléphonique** d’Internet.  

### 📌 HTTP / HTTPS
- **HTTP** = protocole du web (pages, API).  
- **HTTPS** = HTTP + chiffrement (TLS).  
- Exemple d’une requête simple :  
  ```
  GET / HTTP/1.1
  Host: www.example.com
  ```

---

## 4. Exemple concret : une requête web

👉 Quand tu tapes **`https://www.example.com`** dans ton navigateur :  

1. **DNS** → résolution du nom de domaine en adresse IP.  
2. **Connexion TCP** → ouverture d’une connexion fiable avec le serveur.  
3. **Port 443** → le service HTTPS écoute sur cette “porte” standard.  
4. **TLS** → négociation du chiffrement de la communication.  
5. **Requête HTTP** → le navigateur envoie `GET /`.  
6. **Réponse HTTP** → le serveur renvoie du HTML, CSS, JS…  
7. **Affichage** → le navigateur construit et affiche la page.  

---

## 5. Cas pratiques (commandes utiles)

### 🔎 Vérifier la connectivité
```bash
ping www.google.com
```
➡️ Vérifie si la machine répond (test réseau basique).  

### 🔎 Suivre le chemin
```bash
traceroute www.google.com   # Linux/macOS
tracert www.google.com      # Windows
```
➡️ Montre chaque routeur traversé jusqu’au serveur.  

### 🔎 Inspecter une requête web
```bash
curl -v https://www.example.com
```
➡️ Affiche les en-têtes envoyés et la réponse brute.  

### 🔎 Résolution DNS
```bash
nslookup www.google.com
dig www.google.com
```
➡️ Affiche l’adresse IP correspondant au domaine.  

### 🔎 Vérifier les ports ouverts (Linux/macOS)
```bash
netstat -tulnp
```
➡️ Liste des services en écoute sur des ports.  

---

## 6. Schéma simplifié d’une communication web

```
[ Navigateur ]                        (Client)
    │
    │ 1. DNS → "www.example.com" → 93.184.216.34
    ▼
[ Serveur DNS ]                       (Annuaire Internet)
    │
    │ 2. Connexion TCP vers le port 443 (service HTTPS)
    ▼
[ Serveur Web ]                       (Machine distante)
    │
    │ 3. Négociation TLS (connexion sécurisée)
    │ 4. Requête HTTP → "GET /"
    │ 5. Réponse HTTP ← HTML, CSS, JS...
    ▼
[ Navigateur ]                        (Affichage pour l'utilisateur)
```

---

## 7. Ce que doit retenir un développeur
- Les réseaux fonctionnent par **couches** → chaque couche a son rôle.  
- Les **ports** permettent d’adresser le bon service sur une machine.  
- Une simple page web implique **DNS + TCP + Port + TLS + HTTP**.  
- Les outils (`ping`, `traceroute`, `curl`, `dig`) permettent de **déboguer**.  
- Comprendre le réseau aide à :  
  - 🔧 Déboguer des applis (erreurs réseau, ports fermés, DNS).  
  - 🔒 Sécuriser (HTTPS obligatoire).  
  - ⚡ Optimiser (choix TCP/UDP selon le besoin).  

---

👉 Avec cette fiche, un développeur débutant comprend à la fois :  
- la théorie (modèles OSI/TCP-IP),  
- les concepts clés (IP, TCP, UDP, ports, DNS, HTTP),  
- et la pratique (cas concrets + commandes utiles).  
