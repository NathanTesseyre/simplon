
## Introduction

Le stack LAMP (Linux, Apache, MariaDB, PHP) reste la référence pour héberger des sites web dynamiques et des applications PHP. Ce tutoriel explique comment mettre en place un serveur Web sur Debian 13, afin de pouvoir y héberger l'application de votre choix : WordPress, Joomla, Drupal, Nextcloud, etc.

| Composant   | Rôle                                 |
| ----------- | ------------------------------------ |
| **Linux**   | Système d'exploitation               |
| **Apache**  | Serveur web (gère les requêtes HTTP) |
| **MariaDB** | Serveur de base de données           |
| **PHP**     | Langage de scripting côté serveur    |

---

## Configuration minimale d'un serveur LAMP

Avant de commencer l'installation, voici les prérequis recommandés pour faire tourner un serveur LAMP sur Debian 13 :

| Ressource | Minimum | Recommandé |
|---|---|---|
| **CPU** | 1 cœur | 2 cœurs |
| **RAM** | 512 Mo | 2 Go |
| **Stockage** | 10 Go | 20 Go |
| **OS** | Debian 13 (Trixie) | Debian 13 (Trixie) |
| **Accès** | Utilisateur avec droits `sudo` | Utilisateur avec droits `sudo` |

> Assurez-vous que le pare-feu autorise les ports **80** (HTTP) et **443** (HTTPS) si vous souhaitez rendre le serveur accessible depuis l'extérieur.

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

---

## I. Installation d'Apache2

Mettre à jour la machine Linux en premier :

```bash
sudo apt update
```

Ensuite installer Apache2 :

```bash
sudo apt install apache2
```

### Installation d'une version spécifique d'Apache2

**1. Télécharger la version souhaitée depuis le site officiel :**

```bash
# Télécharge le paquet Apache voulu
wget https://archive.apache.org/dist/httpd/httpd-2.4.58.tar.gz

# Décompresse l'archive
tar -xzf httpd-2.4.58.tar.gz

# Se positionner dans le dossier du paquet
cd httpd-2.4.58
```

**2. Installer les dépendances nécessaires :**

```bash
# Installe toutes les dépendances requises par Apache
sudo apt build-dep apache2

# Packages obligatoires pour la compilation d'Apache
sudo apt install libapr1-dev libaprutil1-dev
```

**3. Configurer, compiler et installer :**

```bash
# Exécute le script de configuration
./configure --prefix=/usr/local/apache2

# Compile le code source en binaires exécutables
make

# Copie les fichiers compilés dans les dossiers de destination
sudo make install
```

**4. Démarrer Apache :**

```bash
sudo /usr/local/apache2/bin/apachectl start
```

**Vérifier la version d'Apache installée :**

```bash
sudo apache2ctl -v
```

---

## II. Installation de MariaDB

MariaDB est le serveur de base de données du stack LAMP. Il remplace MySQL tout en restant compatible avec ses commandes et connecteurs.

### Installer MariaDB

```bash
sudo apt update

sudo apt install mariadb-server
```

Vérifier que le service est actif :

```bash
sudo systemctl status mariadb
```

### Sécuriser l'installation

Le script `mysql_secure_installation` permet de supprimer les comptes anonymes, désactiver l'accès root distant et supprimer la base de test :

```bash
sudo mysql_secure_installation
```

Répondre aux questions selon vos besoins. Pour un serveur de production, répondre **oui** à toutes les questions est recommandé.

### Se connecter à MariaDB

Par défaut, l'utilisateur `root` MariaDB s'authentifie via le plugin `unix_socket` (pas de mot de passe nécessaire avec `sudo`) :

```bash
sudo mariadb
```

Ou avec un mot de passe si vous l'avez défini :

```bash
mariadb -u root -p
```

### (Bonus) Créer une base de données et un utilisateur

Une fois connecté au shell MariaDB (`MariaDB [(none)]>`), exécuter :

```sql
-- Créer la base de données
CREATE DATABASE nom_de_la_bdd CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- Créer un utilisateur dédié
CREATE USER 'nom_utilisateur'@'localhost' IDENTIFIED BY 'mot_de_passe';

-- Lui accorder les droits sur la base
GRANT ALL PRIVILEGES ON nom_de_la_bdd.* TO 'nom_utilisateur'@'localhost';

-- Appliquer les changements
FLUSH PRIVILEGES;

EXIT;
```

### Les commandes utiles

| Commande | Description |
|---|---|
| `sudo systemctl start mariadb` | Démarrer le service |
| `sudo systemctl stop mariadb` | Arrêter le service |
| `sudo systemctl restart mariadb` | Redémarrer le service |
| `sudo systemctl enable mariadb` | Activer au démarrage |
| `mariadb --version` | Vérifier la version installée |

---

## III. Installation de PHP

Pour utiliser PHP avec Apache2, il y a deux méthodes :

- **`mod_php`** — le module intégré à Apache
- **PHP-FPM** (*PHP **F**astCGI **P**rocess **M**anager*) — recommandé

> Il est recommandé d'utiliser **PHP-FPM** plutôt que `libapache2-mod-php`. Avec PHP-FPM, vous pourrez **gérer plus de connexions simultanées tout en utilisant moins de ressources**.

### Ajouter le dépôt PHP (sury.org)

```bash
sudo apt update

sudo apt install -y lsb-release ca-certificates curl

sudo curl -sSLo /tmp/debsuryorg-archive-keyring.deb https://packages.sury.org/debsuryorg-archive-keyring.deb

sudo dpkg -i /tmp/debsuryorg-archive-keyring.deb

sudo tee /etc/apt/sources.list.d/php.sources <<EOF
Types: deb
URIs: https://packages.sury.org/php/
Suites: $(lsb_release -sc)
Components: main
Signed-By: /usr/share/keyrings/debsuryorg-archive-keyring.gpg
EOF

sudo apt update
```

### Installer PHP

```bash
# Remplacez 8.5 par votre version cible
sudo apt install -y php8.5

sudo systemctl start php8.5-fpm
```

### Activer les modules et la configuration Apache

```bash
sudo a2enmod proxy_fcgi setenvif

sudo a2enconf php8.5-fpm

sudo systemctl restart apache2
```

### Installer les extensions PHP courantes

Ces extensions sont souvent indispensables selon l'application hébergée (ex. : `php8.5-mysql` permet la connexion à une base de données MySQL depuis PHP) :

```bash
sudo apt install php8.5-{mysql,curl,gd,zip,mcrypt}
```

---

## Bonus :  Installation de phpMyAdmin

### Qu'est-ce que phpMyAdmin ?

**phpMyAdmin** est une interface web open source qui permet de gérer MariaDB (ou MySQL) graphiquement, sans avoir à taper de commandes SQL à la main.

Depuis un navigateur, vous pouvez :

- Créer, modifier et supprimer des bases de données et des tables
- Exécuter des requêtes SQL
- Importer et exporter des données (`.sql`, `.csv`…)
- Gérer les utilisateurs et leurs permissions

> C'est l'outil idéal pour administrer facilement votre base de données, notamment lors du déploiement d'applications comme WordPress ou Nextcloud.

### Installer phpMyAdmin

```bash
sudo apt update

sudo apt install phpmyadmin
```

Durant l'installation, un assistant interactif se lance :

- **Serveur web à configurer** : sélectionner `apache2` avec `Espace` puis valider avec `Entrée`
- **Configurer la base de données avec dbconfig-common** : sélectionner `Oui`
- **Mot de passe** : définir un mot de passe pour l'utilisateur phpMyAdmin en base

### Activer l'extension PHP requise

Installer le module `mbstring` s'il n'est pas déjà présent :

```bash
sudo apt install php8.5-mbstring
```

Puis l'activer :

```bash
sudo phpenmod mbstring

sudo systemctl restart apache2
```

### Accéder à phpMyAdmin

Ouvrez un navigateur et rendez-vous sur :

```
http://adresse-ip-du-serveur/phpmyadmin
```

Connectez-vous avec un utilisateur MariaDB existant (ex. celui créé à la section II).

### Sécuriser l'accès (recommandé)

Par défaut, phpMyAdmin est accessible par n'importe qui. Pour restreindre l'accès à votre seule adresse IP :

```bash
sudo nano /etc/apache2/conf-available/phpmyadmin.conf
```

Ajouter dans le bloc `<Directory /usr/share/phpmyadmin>` :

```apache
Require ip 192.168.1.0/24
```

Puis redémarrer Apache :

```bash
sudo systemctl restart apache2
```
