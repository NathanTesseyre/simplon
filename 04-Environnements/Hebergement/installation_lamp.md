---
title: "Installation d'un serveur LAMP"
tags:
  - lamp
  - apache
  - mysql
  - php
  - linux
  - hébergement
  - serveur-web
section: 04-Environnements
domaine: Hébergement
statut: actif
liens_connexes:
  - [[Hosting]]
  - [[All-kinds-of-web-server-configs]]
  - [[intro_linux]]
  - [[gestion_paquets]]
  - [[services_demarrage]]
  - [[utilisateurs_permissions]]
  - [[fiche_securite_systeme]]
---

# Installation d'un serveur LAMP

**LAMP** = **L**inux + **A**pache + **M**ySQL (ou MariaDB) + **P**HP

C'est l'une des stacks web les plus répandues pour héberger des applications dynamiques (WordPress, Laravel, etc.).

---

## 1. Prérequis

- Serveur ou VM sous **Debian/Ubuntu** (ou dérivé)
- Accès `root` ou `sudo`
- Connexion internet active

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Apache — le serveur web

### Installation

```bash
sudo apt install apache2 -y
```

### Vérification et démarrage

```bash
sudo systemctl start apache2
sudo systemctl enable apache2   # démarrage automatique au boot
sudo systemctl status apache2
```

### Test rapide

Ouvrir `http://localhost` dans un navigateur → page par défaut Apache ("It works!").

### Fichiers clés

| Fichier / Dossier | Rôle |
|---|---|
| `/etc/apache2/apache2.conf` | Config principale |
| `/etc/apache2/sites-available/` | VirtualHosts disponibles |
| `/etc/apache2/sites-enabled/` | VirtualHosts actifs (symlinks) |
| `/var/www/html/` | Racine web par défaut |
| `/var/log/apache2/` | Logs (access.log, error.log) |

### Commandes utiles

```bash
sudo a2ensite monsite.conf      # activer un VirtualHost
sudo a2dissite 000-default.conf # désactiver le VirtualHost par défaut
sudo a2enmod rewrite            # activer le module rewrite (URLs propres)
sudo apache2ctl configtest      # vérifier la syntaxe de la config
sudo systemctl reload apache2   # recharger sans couper le service
```

---

## 3. MySQL / MariaDB — la base de données

### Installation

```bash
# MariaDB (recommandé, drop-in replacement de MySQL)
sudo apt install mariadb-server -y

# ou MySQL
sudo apt install mysql-server -y
```

### Démarrage et activation

```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

### Sécurisation initiale (indispensable en production)

```bash
sudo mysql_secure_installation
```

Répondre aux questions :
- Définir un mot de passe root
- Supprimer les utilisateurs anonymes → **Oui**
- Interdire la connexion root à distance → **Oui**
- Supprimer la base de test → **Oui**
- Recharger les privilèges → **Oui**

### Connexion au shell MySQL

```bash
sudo mysql -u root -p
```

### Créer une base et un utilisateur dédié

```sql
CREATE DATABASE monapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'monuser'@'localhost' IDENTIFIED BY 'motdepasse_solide';
GRANT ALL PRIVILEGES ON monapp.* TO 'monuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 4. PHP — le langage côté serveur

### Installation

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

### Extensions courantes

```bash
sudo apt install php-curl php-gd php-mbstring php-xml php-zip php-intl -y
```

### Vérifier la version installée

```bash
php -v
```

### Test PHP dans Apache

Créer un fichier de test :

```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

Ouvrir `http://localhost/info.php` → page d'infos PHP.

> **Supprimer ce fichier après vérification** — il expose des infos sensibles.

```bash
sudo rm /var/www/html/info.php
```

### Fichiers clés PHP

| Fichier | Rôle |
|---|---|
| `/etc/php/<version>/apache2/php.ini` | Config PHP pour Apache |
| `/etc/php/<version>/cli/php.ini` | Config PHP en ligne de commande |

Paramètres importants dans `php.ini` :

```ini
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 120
memory_limit = 256M
```

---

## 5. Configurer un VirtualHost

Exemple pour héberger `monsite.local` :

```bash
sudo nano /etc/apache2/sites-available/monsite.conf
```

```apache
<VirtualHost *:80>
    ServerName monsite.local
    ServerAdmin admin@monsite.local
    DocumentRoot /var/www/monsite

    <Directory /var/www/monsite>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/monsite_error.log
    CustomLog ${APACHE_LOG_DIR}/monsite_access.log combined
</VirtualHost>
```

```bash
sudo mkdir -p /var/www/monsite
sudo chown -R $USER:www-data /var/www/monsite
sudo chmod -R 755 /var/www/monsite

sudo a2ensite monsite.conf
sudo systemctl reload apache2
```

Ajouter au fichier hosts local :

```bash
echo "127.0.0.1 monsite.local" | sudo tee -a /etc/hosts
```

---

## 6. Pare-feu (UFW)

```bash
sudo ufw allow 'Apache Full'   # ports 80 et 443
sudo ufw allow OpenSSH         # ne pas se couper l'accès SSH !
sudo ufw enable
sudo ufw status
```

---

## 7. Vérification globale de la stack

```bash
# Statuts des services
sudo systemctl status apache2 mariadb

# Tester la connexion PHP → MySQL
php -r "new PDO('mysql:host=localhost;dbname=monapp', 'monuser', 'motdepasse_solide'); echo 'Connexion OK\n';"
```

---

## 8. Récapitulatif des services

| Composant | Service systemd | Port par défaut |
|---|---|---|
| Apache | `apache2` | 80 (HTTP), 443 (HTTPS) |
| MariaDB / MySQL | `mariadb` / `mysql` | 3306 |
| PHP | (module Apache, pas de service propre) | — |

---

## Aller plus loin

- [[All-kinds-of-web-server-configs]] — Nginx, configuration avancée
- [[ssh_avance]] — sécuriser l'accès au serveur
- [[fiche_securite_systeme]] — durcissement système
- Installer **phpMyAdmin** pour administrer MySQL via interface web
- Passer en HTTPS avec **Let's Encrypt / Certbot** : `sudo certbot --apache`
