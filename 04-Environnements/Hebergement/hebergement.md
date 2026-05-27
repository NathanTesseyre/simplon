---
title: "Hébergement web"
tags:
  - hébergement
  - vps
  - cloud
  - déploiement
section: 04-Environnements
domaine: Hébergement
statut: actif
liens_connexes:
  - [[web_server_configs]]
  - [[fiche_serveurs_web]]
  - [[docker_fondamentaux]]
  - [[docker_compose]]
  - [[ssh_avance]]
  - [[fiche_securite_systeme]]
  - [[ansible_fondamentaux]]
  - [[terraform_fondamentaux]]
  - [[cloud_aws_fondamentaux]]
---

# Hosting

## Types d’hébergement : lequel choisir ?

| Type d’hébergement | Description | Pour qui ? | Exemples de fournisseurs |
| ----- | ----- | ----- | ----- |
| **Hébergement mutualisé** | Serveur partagé avec d’autres sites. Ressources limitées. | Débutants, petits projets, blogs, sites vitrines. | OVH, Hostinger, SiteGround |
| **VPS (Virtual Private Server)** | Machine virtuelle dédiée. Plus de contrôle et de ressources. | Développeurs intermédiaires, projets en croissance, besoin de personnalisation. | DigitalOcean, Linode, OVH, AWS Lightsail |
| **Hébergement cloud** | Ressources scalables, pay-as-you-go. | Projets variables, applications modernes, équipes agiles. | AWS, Google Cloud, Azure, Heroku |
| **Serveur dédié** | Machine physique entière dédiée. | Grosses applications, trafic élevé, besoin de sécurité maximale. | OVH, Online.net, Hetzner |
| **Hébergement spécialisé** | Optimisé pour un CMS ou un framework (WordPress, Node.js, etc.). | Projets spécifiques, gain de temps sur la configuration. | Kinsta (WordPress), Vercel (Next.js) |

### Conseil :

En formation, privilégiez un VPS ou un hébergement cloud gratuit/freemium (comme Heroku, Railway, ou GitHub Pages pour du statique) pour apprendre à configurer un environnement.

## Critères de choix techniques

### Performances

* **CPU/RAM** : Vérifiez les ressources allouées (ex : 1 vCPU, 1 Go RAM pour un petit projet).  
* **Stockage** : SSD recommandé pour la vitesse.  
* **Bande passante** : Illimitée ou suffisante pour votre trafic estimé.

### Accès et contrôle

* **[[ssh_avance|SSH]]** : Indispensable pour administrer votre serveur.  
* **Root/Sudo** : Nécessaire pour installer des logiciels (Nginx, Docker, etc.).  
* **Panneau de contrôle** : cPanel/Plesk (mutualisé) vs ligne de commande (VPS/cloud).

### Compatibilité technique

* **Langages** : PHP, Node.js, Python, Ruby, etc. Vérifiez les versions supportées.  
* **Bases de données** : MySQL, PostgreSQL, MongoDB.  
* **Outils DevOps** : Docker, CI/CD, Git intégrations.

### Sécurité

* **Certificats SSL** : Let’s Encrypt (gratuit) → [[fiche_securite_systeme]].  
* **Sauvegardes** : Automatiques et restaurables.  
* **Pare-feu** : Configurable (UFW, iptables) → [[fiche_securite_systeme]].

### Support

* Documentation technique complète.  
* Support réactif (chat, ticket) pour les urgences.

## Étapes clés pour configurer son hébergement

### A. Déploiement d’un projet

#### 1. Transférer les fichiers :

* Via SFTP/SCP (FileZilla, scp en CLI).
* Ou Git (GitHub/GitLab + déploiement automatique).

#### 2. Configurer le serveur web :

* Apache/Nginx : fichiers de config (sites-available, nginx.conf).
* Exemple pour Nginx :
  ```nginx
  server {
      listen 80;
      server_name monprojet.com;
      root /var/www/monprojet;
      index index.html;
  }
  ```

#### 3. Base de données :

* Créer un utilisateur et une base dédiés.
* Importer un dump SQL si nécessaire.

#### 4. Domaines et DNS :

* Pointer votre domaine vers l’IP du serveur (A/AAAA record).
* Configurer les sous-domaines si besoin.

### B. Automatisation

* **Scripts de déploiement** : [[shell_scripting|Bash]], [[ansible_fondamentaux|Ansible]], ou outils comme Deployer.
* **CI/CD** : GitHub Actions, GitLab CI → [[02_setup_gitea_jenkins]].

### C. Surveillance

* **Logs** : /var/log/nginx/error.log, journalctl pour les services.
* **Outils** : Netdata, Prometheus/Grafana pour le monitoring.

## Bonnes pratiques pour développeurs

* Isoler les environnements : Utilisez des conteneurs (Docker) ou des machines virtuelles.
* Gérer les dépendances : requirements.txt (Python), package.json (Node.js), composer.json (PHP).
* Sauvegarder régulièrement : rsync, mysqldump, ou outils comme Duplicati.
* Sécuriser :
  * Mettre à jour le système (apt update && apt upgrade).
  * Désactiver les services inutiles.
  * Utiliser des clés SSH plutôt que des mots de passe.

## Outils utiles

| Outil | Utilité | Lien |
| ----- | ----- | ----- |
| **[[docker_fondamentaux]]** | Conteneurisation des applications. | [docker.com](https://www.docker.com) |
| **PM2** | Gestionnaire de processus Node.js. | [pm2.io](https://pm2.io) |
| **Certbot** | Générer des certificats SSL Let’s Encrypt. | [certbot.eff.org](https://certbot.eff.org) |
| **NGINX Proxy Manager** | Gérer plusieurs sites/hôtes facilement. | [github.com](https://github.com/NginxProxyManager/nginx-proxy-manager) |

## Exemple de workflow pour un projet en formation

1. **Développement local** : Codez en local avec VS Code, testez avec Docker.
2. **Versionning** : Push sur GitHub/GitLab.
3. **Déploiement** :
    * Branche main → Déploiement automatique sur le VPS via GitHub Actions.
    * Ou utilisation de rsync pour copier les fichiers.
4. **Tests** : Vérifiez les logs, la disponibilité (curl -I votre-site.com).
5. **Itération** : Améliorez la config, optimisez les performances.

## Erreurs courantes à éviter

* **Négliger les sauvegardes** : Perte de données = catastrophe.
* **Ignorer les logs** : La plupart des problèmes y sont documentés.
* **Ouvrir tous les ports** : Pare-feu mal configuré = risques de hack.
* **Ne pas monitorer** : Un site lent ou down = mauvaise expérience utilisateur.