---
title: "Gestion des processus Linux"
tags:
  - linux
  - processus
  - ps
  - kill
  - htop
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[fiche_processus_threads]]
  - [[services_demarrage]]
  - [[logs_supervision]]
---

# 📘 Gestion des processus Linux

## Qu'est-ce qu'un processus ?

Un processus est un programme en cours d'exécution. Chaque processus a un identifiant unique (PID), tourne sous un utilisateur, consomme du CPU et de la RAM, et peut avoir des processus enfants.

```
PID 1 (systemd)
├── PID 450 (sshd)
│   └── PID 1203 (sshd: alice@pts/0)
│       └── PID 1204 (bash)
│           └── PID 1250 (vim)
├── PID 512 (nginx: master)
│   ├── PID 513 (nginx: worker)
│   └── PID 514 (nginx: worker)
└── PID 620 (postgres)
    ├── PID 621 (postgres: checkpointer)
    └── PID 622 (postgres: writer)
```

Chaque processus a :
- **PID** : son identifiant unique
- **PPID** : PID de son parent
- **UID** : utilisateur sous lequel il tourne
- **État** : Running, Sleeping, Stopped, Zombie

## Voir les processus

```bash
# Snapshot de tous les processus
ps aux
# a = tous les utilisateurs, u = format détaillé, x = processus sans terminal

# Colonnes clés de ps aux :
# USER  PID  %CPU  %RAM  VSZ  RSS  TTY  STAT  START  TIME  COMMAND
# alice 1234  0.5   1.2  ...  ...  pts  S     10:00  0:01  node server.js

# Chercher un processus spécifique
ps aux | grep nginx
ps aux | grep -v grep | grep nginx   # sans la ligne grep elle-même

# Alternative : pgrep
pgrep nginx          # affiche uniquement les PIDs de nginx
pgrep -l nginx       # PID + nom
pgrep -a nginx       # PID + commande complète
pgrep -u alice       # tous les processus de alice

# Arbre des processus
pstree
pstree -p            # avec les PIDs
pstree -p alice      # uniquement les processus de alice
```

## Surveillance en temps réel

```bash
# top — surveillance basique (intégré partout)
top
# Commandes dans top :
# q = quitter
# k = kill (demande le PID)
# r = renice (changer la priorité)
# M = trier par RAM
# P = trier par CPU
# u = filtrer par utilisateur

# htop — version améliorée (sudo apt install htop)
htop
# Navigation à la souris, barres visuelles, plus intuitif

# Surveiller un processus spécifique
top -p 1234
top -p $(pgrep nginx | tr "\n" ",")  # surveiller tous les workers nginx
```

## Signaux et arrêt de processus

Un signal est un message envoyé à un processus pour lui demander de faire quelque chose.

```bash
# Les signaux les plus courants
kill -l       # liste tous les signaux

# SIGTERM (15) — arrêt propre (le processus peut faire le ménage avant de s'arrêter)
kill -TERM 1234
kill -15 1234
kill 1234        # SIGTERM par défaut

# SIGKILL (9) — arrêt forcé immédiat (ne peut pas être ignoré)
kill -KILL 1234
kill -9 1234

# SIGHUP (1) — recharger la configuration (utile pour nginx, sshd...)
kill -HUP 1234
kill -1 1234

# SIGUSR1/SIGUSR2 — signaux personnalisés (comportement défini par l'appli)
kill -USR1 1234

# Tuer par nom
killall nginx       # SIGTERM à tous les processus nommés "nginx"
killall -9 nginx    # SIGKILL
pkill nginx         # identique à killall
pkill -u alice      # tuer tous les processus de alice

# Règle : toujours essayer SIGTERM avant SIGKILL
# SIGKILL peut laisser des fichiers verrou, des connexions BD non fermées, etc.
```

## Priorités CPU (nice)

La valeur "nice" va de -20 (priorité maximale) à +19 (priorité minimale). Par défaut, un processus démarre à 0.

```bash
# Lancer un processus avec une priorité basse (laisse les autres passer)
nice -n 15 tar -czf archive.tar.gz /données

# Modifier la priorité d'un processus en cours
renice -n 10 -p 1234        # baisser la priorité du PID 1234
renice -n -5 -p 1234        # augmenter la priorité (nécessite root)
sudo renice -n -10 -p 1234

# Voir les priorités dans ps
ps aux | head -1; ps aux | sort -k3 -rn | head -10  # top 10 par CPU
```

## Arrière-plan et jobs

```bash
# Lancer en arrière-plan (non-bloquant)
commande &
long_calcul.sh &
echo $!    # PID du dernier processus lancé en arrière-plan

# Suspendre un processus en cours (Ctrl+Z)
# Puis le mettre en arrière-plan
bg         # reprendre en arrière-plan
fg         # ramener au premier plan
fg %2      # ramener le job n°2

# Voir les jobs
jobs
jobs -l    # avec les PIDs

# Exemple pratique
vim fichier.txt   # ouvrir vim
# Ctrl+Z           # suspendre vim
# faire autre chose...
fg               # reprendre vim

# Lancer un processus qui survive à la fermeture du terminal
nohup long_script.sh &           # sortie dans nohup.out
nohup long_script.sh > log.txt 2>&1 &   # sortie dans log.txt
disown %1   # détacher le job 1 du terminal (après &)

# Outil dédié : screen ou tmux (persistance de session)
tmux new -s ma-session
# Ctrl+B D pour détacher, tmux attach -t ma-session pour reprendre
```

## Consommation de ressources

```bash
# RAM utilisée par processus
ps aux --sort=-%mem | head -10    # top 10 par RAM
ps aux --sort=-%cpu | head -10    # top 10 par CPU

# Détail mémoire d'un processus
cat /proc/1234/status | grep -E "VmRSS|VmSize"
# VmRSS = RAM physique réellement utilisée
# VmSize = mémoire virtuelle (inclut les fichiers mappés, libs partagées)

# Vue globale de la mémoire
free -h
# total     used    free    shared  buff/cache  available
# 16Gi      4.2Gi   8.1Gi   200Mi   3.7Gi       11Gi
# "available" = ce que le système peut donner à de nouveaux processus

# Charge système (load average)
uptime
# 10:30  up 5 days,  load average: 0.52, 0.48, 0.41
# Les 3 nombres = moyenne sur 1 min, 5 min, 15 min
# Un load average < nombre de CPUs = système sous contrôle
```

## Cas pratiques

**Trouver et tuer un processus qui bloque un port :**
```bash
ss -tlnp | grep :3000
# ou
sudo lsof -i :3000
# Récupérer le PID et kill
kill -TERM <PID>
```

**Identifier ce qui consomme le plus de ressources :**
```bash
# CPU
ps aux --sort=-%cpu | head -5

# RAM
ps aux --sort=-%mem | head -5

# En interactif avec htop : F6 pour trier, F9 pour kill
```

**Surveiller qu'un processus est bien vivant :**
```bash
# Script simple de watchdog
while true; do
    if ! pgrep -x "mon-api" > /dev/null; then
        echo "$(date): mon-api est mort, redémarrage..." >> /var/log/watchdog.log
        systemctl start mon-api
    fi
    sleep 30
done
```

**Lancer une tâche longue sur un serveur distant sans risque :**
```bash
ssh user@serveur
tmux new -s deploiement
./deploy.sh   # lancer le script
# Ctrl+B D  → détacher (le script continue même si la connexion SSH coupe)
# Plus tard : tmux attach -t deploiement pour reprendre
```

---

### ✅ À retenir

- `ps aux` = snapshot de tous les processus
- `pgrep -l nginx` = trouver un processus par nom
- `kill -TERM` avant `kill -9` (toujours essayer l'arrêt propre d'abord)
- `htop` = outil de surveillance interactif indispensable
- `nohup commande &` = processus qui survive à la fermeture du terminal
- `nice` et `renice` = contrôler la priorité CPU
- `tmux` = sessions persistantes sur serveur distant

**Voir aussi →** [[fiche_processus_threads]] pour les concepts (threads, concurrence), [[services_demarrage]] pour gérer les processus via systemd.
