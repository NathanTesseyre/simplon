---
title: "Logs et supervision"
tags:
  - linux
  - logs
  - journald
  - monitoring
  - observabilité
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[services_demarrage]]
  - [[sonarqube-jenkins-node-doc]]
  - [[gestion_processus]]
---

# 📘 Logs & supervision

## Pourquoi les logs sont critiques

Les logs sont la première source d'information quand quelque chose ne fonctionne pas. En production, ils permettent de diagnostiquer des pannes, détecter des intrusions, auditer des accès, et comprendre le comportement d'une application sous charge.

Un système sans logs correctement gérés est aveugle.

## Architecture des logs sous Linux

```
Applications
    │
    ▼
systemd-journald  ←─── noyau (dmesg)
    │
    ├──► journal binaire (/run/log/journal/)
    │
    └──► rsyslog/syslog-ng (si configuré)
              │
              └──► /var/log/*.log
```

## journalctl — le point d'entrée principal

`journalctl` interroge le journal systemd. C'est la commande de référence sur les distributions modernes.

```bash
# Tout le journal (depuis le début, du plus ancien au plus récent)
journalctl

# Suivre en temps réel (comme tail -f)
journalctl -f

# Filtrer par service
journalctl -u nginx
journalctl -u nginx -f          # en temps réel pour nginx
journalctl -u ssh -u nginx      # plusieurs services

# Filtrer par temps
journalctl --since "1 hour ago"
journalctl --since "2024-01-15 08:00" --until "2024-01-15 09:00"
journalctl --since today
journalctl --since yesterday

# Filtrer par priorité (0=emerg, 1=alert, 2=crit, 3=err, 4=warning, 5=notice, 6=info, 7=debug)
journalctl -p err               # erreurs et plus grave
journalctl -p warning           # warnings et plus grave
journalctl -u nginx -p err      # erreurs nginx uniquement

# Afficher les N dernières lignes
journalctl -n 50
journalctl -u nginx -n 100

# Format JSON (utile pour traitement automatique)
journalctl -u nginx -o json-pretty | head -50

# Boot précédent (utile après un crash)
journalctl -b -1                # boot précédent
journalctl -b -2                # avant-avant-dernier boot
journalctl --list-boots         # liste des boots

# Taille du journal
journalctl --disk-usage
```

## Fichiers de logs classiques

Certaines applications écrivent directement dans des fichiers, pas dans journald.

| Fichier | Contenu |
|---------|---------|
| `/var/log/syslog` | Logs système généraux (Debian/Ubuntu) |
| `/var/log/messages` | Idem (RHEL/CentOS) |
| `/var/log/auth.log` | Authentifications, sudo, SSH |
| `/var/log/kern.log` | Messages du noyau |
| `/var/log/dpkg.log` | Installations/suppressions de paquets |
| `/var/log/apt/history.log` | Historique apt |
| `/var/log/nginx/access.log` | Requêtes HTTP Nginx |
| `/var/log/nginx/error.log` | Erreurs Nginx |
| `/var/log/postgresql/` | Logs PostgreSQL |

```bash
# Lire les logs en temps réel
tail -f /var/log/syslog
tail -f /var/log/nginx/access.log

# Suivre plusieurs fichiers simultanément
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Rechercher dans les logs
grep "ERROR" /var/log/syslog
grep "Failed password" /var/log/auth.log
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
# → liste des IPs qui ont tenté de se connecter par SSH avec les plus fréquentes en premier
```

## dmesg — messages du noyau

```bash
# Messages noyau (boot, matériel, drivers)
dmesg
dmesg -T              # avec timestamps lisibles
dmesg -T | tail -20   # 20 derniers messages
dmesg | grep -i error
dmesg | grep -i usb   # détection de périphériques USB
dmesg | grep -i "out of memory"  # détecter un OOM killer
```

## logrotate — rotation des logs

Sans rotation, les logs grossissent jusqu'à remplir le disque. `logrotate` les compresse et archive régulièrement.

Configuration type dans `/etc/logrotate.d/monapp` :
```
/var/log/monapp/*.log {
    daily           # rotation quotidienne
    missingok       # pas d'erreur si le fichier est absent
    rotate 14       # garder 14 fichiers
    compress        # compresser avec gzip
    delaycompress   # compresser le précédent, pas le courant
    notifempty      # ne pas tourner si le fichier est vide
    create 0640 www-data adm  # créer un nouveau fichier avec ces droits
    postrotate
        nginx -s reopen  # signaler à nginx de rouvrir ses fichiers
    endscript
}
```

```bash
# Tester la configuration sans l'appliquer
logrotate -d /etc/logrotate.d/nginx

# Forcer la rotation immédiatement
sudo logrotate -f /etc/logrotate.conf
```

## Supervision système en temps réel

```bash
# Vue globale CPU, RAM, processus
top
htop                  # plus lisible (à installer : sudo apt install htop)

# Utilisation mémoire
free -h
vmstat 2              # toutes les 2 secondes : CPU, mémoire, I/O

# Utilisation disque
df -h                 # espace disque par partition
du -sh /var/log/*     # taille des dossiers de logs
iostat -x 2           # I/O disque (à installer : sudo apt install sysstat)

# Réseau
ss -s                 # statistiques réseau globales
nload                 # bande passante en temps réel (à installer)
iftop                 # par connexion (à installer)

# Tout en un
watch -n 1 'df -h && free -h'   # actualiser toutes les secondes
```

## Cas pratiques

**Diagnostiquer un service qui ne démarre pas :**
```bash
sudo systemctl status nginx
journalctl -u nginx --since "5 minutes ago" -p err
# Lire le message d'erreur → souvent un problème de config ou de port déjà utilisé
```

**Traquer les tentatives d'intrusion SSH :**
```bash
grep "Failed password" /var/log/auth.log | \
  awk '{print $11}' | sort | uniq -c | sort -rn | head -10
# → Top 10 des IPs qui essaient de se connecter
```

**Trouver ce qui remplit le disque :**
```bash
df -h                           # voir quelle partition est pleine
du -sh /var/log/* | sort -rh | head -10   # trouver les gros fichiers de log
journalctl --disk-usage         # taille du journal systemd
sudo journalctl --vacuum-size=500M  # limiter le journal à 500 Mo
```

**Suivre les logs d'une appli Node.js lancée avec systemd :**
```bash
journalctl -u mon-api -f -n 50
```

## Observabilité en production

Pour aller au-delà des logs fichiers, les équipes ops utilisent des stacks d'observabilité :

**ELK / EFK** : collecte centralisée des logs
- Elasticsearch (stockage et recherche) + Logstash/Fluentd (collecte) + Kibana (visualisation)
- Tous les logs de tous les serveurs dans une interface web

**Prometheus + Grafana** : métriques et alertes
- Prometheus scrape des métriques (CPU, RAM, requêtes/sec, latence)
- Grafana affiche des dashboards
- Alertmanager envoie des alertes (mail, Slack) quand un seuil est dépassé

**Les trois piliers de l'observabilité :**
- **Logs** : ce qui s'est passé (événements discrets)
- **Métriques** : comment le système se comporte dans le temps (séries temporelles)
- **Traces** : comment une requête traverse les services (distribué)

---

### ✅ À retenir

- `journalctl -u service -f` = suivre les logs d'un service en temps réel
- `journalctl -p err` = voir uniquement les erreurs
- `journalctl -b -1` = logs du boot précédent (après un crash)
- `/var/log/auth.log` = tentatives de connexion SSH
- `logrotate` = indispensable pour ne pas remplir le disque
- En prod : ELK/EFK pour la centralisation, Prometheus/Grafana pour les métriques

**Voir aussi →** [[services_demarrage]] pour gérer les services, [[gestion_processus]] pour les processus.
