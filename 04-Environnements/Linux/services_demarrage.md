---
title: "Services et démarrage (systemd)"
tags:
  - linux
  - systemd
  - services
  - boot
  - daemon
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[gestion_processus]]
  - [[logs_supervision]]
---

# 📘 Services & démarrage (systemd)

## La séquence de démarrage Linux

```
Mise sous tension
      │
      ▼
  BIOS / UEFI
  (initialise le matériel, cherche un périphérique de boot)
      │
      ▼
    GRUB
  (bootloader : charge le noyau Linux en mémoire)
      │
      ▼
  Noyau Linux
  (monte le filesystem racine, lance le premier processus)
      │
      ▼
   systemd  (PID 1)
  (orchestre le démarrage de tous les services)
      │
      ├── réseau
      ├── logs
      ├── SSH
      ├── base de données
      ├── serveur web
      └── ... tous les services configurés
```

`systemd` est le processus init (PID 1) sur la quasi-totalité des distributions Linux modernes (Ubuntu, Debian, RHEL, Fedora, Arch…). Il est responsable du démarrage, de l'arrêt, et de la supervision de tous les services.

## Gérer les services avec systemctl

```bash
# État d'un service
sudo systemctl status nginx
sudo systemctl status ssh

# Démarrer / arrêter / redémarrer
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Recharger la config sans redémarrer (si le service le supporte)
sudo systemctl reload nginx

# Activer au démarrage (démarre automatiquement à chaque boot)
sudo systemctl enable nginx

# Désactiver au démarrage
sudo systemctl disable nginx

# Activer ET démarrer immédiatement en une commande
sudo systemctl enable --now nginx

# Désactiver ET arrêter immédiatement
sudo systemctl disable --now nginx

# Voir si un service est actif / activé au boot
systemctl is-active nginx
systemctl is-enabled nginx

# Lister tous les services
systemctl list-units --type=service
systemctl list-units --type=service --state=running   # uniquement les actifs
systemctl list-units --type=service --state=failed    # uniquement les en échec
```

**Lire la sortie de `systemctl status` :**
```
● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled)  ← activé au boot
     Active: active (running) since Mon 2024-01-15 10:23:00 UTC   ← en cours
    Process: 1234 ExecStart=/usr/sbin/nginx
   Main PID: 1235 (nginx)
      Tasks: 2
     Memory: 4.5M
        CPU: 45ms
     CGroup: /system.slice/nginx.service
             ├─1235 nginx: master process
             └─1236 nginx: worker process
```

## Créer un service systemd personnalisé

Utile pour lancer une application Node.js, Python, ou n'importe quel programme comme un service système.

**Fichier d'unité** : `/etc/systemd/system/mon-api.service`

```ini
[Unit]
Description=Mon API Node.js
Documentation=https://github.com/monrepo
After=network.target       # démarre après le réseau
After=postgresql.service   # démarre après PostgreSQL

[Service]
Type=simple                # le processus principal reste en foreground
User=www-data              # utilisateur qui lance le processus (pas root !)
WorkingDirectory=/opt/mon-api
ExecStart=/usr/bin/node /opt/mon-api/server.js
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure         # redémarre automatiquement en cas de crash
RestartSec=5               # attendre 5s avant de redémarrer
StandardOutput=journal     # envoyer stdout vers journald
StandardError=journal      # envoyer stderr vers journald
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=/opt/mon-api/.env   # charger les variables d'un fichier

[Install]
WantedBy=multi-user.target  # démarrer en mode normal (avec réseau)
```

```bash
# Après création ou modification d'un fichier .service
sudo systemctl daemon-reload

# Activer et démarrer
sudo systemctl enable --now mon-api

# Vérifier
sudo systemctl status mon-api
journalctl -u mon-api -f
```

**Valeurs courantes pour `Restart=` :**
- `no` — ne jamais redémarrer (défaut)
- `on-failure` — redémarrer uniquement sur erreur (code de sortie non nul)
- `always` — redémarrer toujours, même sur arrêt propre
- `on-abnormal` — redémarrer sur signal ou timeout

## Les timers systemd (cron moderne)

Les timers systemd remplacent avantageusement cron : meilleur logging, gestion des services manqués, intégration avec journald.

**Créer un timer qui lance un script chaque heure :**

Fichier service (`/etc/systemd/system/backup.service`) :
```ini
[Unit]
Description=Sauvegarde quotidienne

[Service]
Type=oneshot
User=backup
ExecStart=/usr/local/bin/backup.sh
```

Fichier timer (`/etc/systemd/system/backup.timer`) :
```ini
[Unit]
Description=Déclencher backup toutes les heures

[Timer]
OnCalendar=hourly           # toutes les heures
OnCalendar=daily            # tous les jours à minuit
OnCalendar=Mon *-*-* 03:00  # tous les lundis à 3h
OnCalendar=*-*-* 02:30:00   # tous les jours à 2h30
Persistent=true             # si le timer a été manqué (machine éteinte), le lancer au prochain démarrage

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable --now backup.timer

# Voir tous les timers actifs
systemctl list-timers
systemctl list-timers --all
```

## Targets — les niveaux de démarrage

Les targets sont l'équivalent des runlevels SysV.

| Target | Description | Runlevel équivalent |
|--------|-------------|---------------------|
| `poweroff.target` | Extinction | 0 |
| `rescue.target` | Mode maintenance (root seul) | 1 |
| `multi-user.target` | Multi-utilisateurs, avec réseau, sans GUI | 3 |
| `graphical.target` | Multi-utilisateurs avec interface graphique | 5 |
| `reboot.target` | Redémarrage | 6 |

```bash
# Voir la target par défaut
systemctl get-default

# Changer la target par défaut
sudo systemctl set-default multi-user.target   # pas de GUI au démarrage

# Basculer vers une target immédiatement
sudo systemctl isolate rescue.target           # mode maintenance
```

## Analyser le temps de démarrage

```bash
# Temps total de boot
systemd-analyze

# Contribution de chaque service au temps de boot
systemd-analyze blame

# Graphique en SVG du démarrage
systemd-analyze plot > boot.svg
```

## Cas pratiques

**Déboguer un service qui ne démarre pas :**
```bash
sudo systemctl start mon-service
sudo systemctl status mon-service   # lire le message d'erreur
journalctl -u mon-service -n 50 -p err  # erreurs récentes
```

**Empêcher un service de démarrer sans le désactiver :**
```bash
sudo systemctl mask nginx    # crée un lien vers /dev/null, impossible à démarrer
sudo systemctl unmask nginx  # retire le masque
```

**Voir les dépendances d'un service :**
```bash
systemctl list-dependencies nginx
systemctl list-dependencies --reverse nginx  # qui dépend de nginx ?
```

---

### ✅ À retenir

- `systemctl enable --now service` = activer au boot ET démarrer maintenant
- `systemctl status service` = premier réflexe quand un service pose problème
- `journalctl -u service -f` = suivre les logs d'un service en temps réel
- Les fichiers `.service` vont dans `/etc/systemd/system/` + `daemon-reload` après modification
- `Restart=on-failure` dans un service = redémarrage automatique sur crash
- Les timers systemd remplacent cron avec une meilleure intégration

**Voir aussi →** [[logs_supervision]] pour approfondir journalctl, [[gestion_processus]] pour gérer les processus manuellement.
