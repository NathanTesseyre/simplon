---
title: "Sauvegarde et restauration"
tags:
  - linux
  - backup
  - restauration
  - rsync
  - sysadmin
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[shell_scripting]]
  - [[services_demarrage]]
  - [[fiche_database]]
  - [[Docker]]
  - [[docker_compose]]
  - [[ssh_avance]]
---

# 📘 Fiche : Sauvegarde et restauration

---

## 1. Pourquoi sauvegarder ?

Une sauvegarde inutile est celle qu'on n'a jamais testée en restauration.

**Risques couverts :**
- Suppression accidentelle de données
- Corruption de fichiers
- Panne matérielle
- Ransomware / attaque
- Erreur humaine lors d'une mise à jour

---

## 2. La règle 3-2-1

| Règle | Signification |
|---|---|
| **3** copies des données | 1 originale + 2 sauvegardes |
| **2** supports différents | Ex : disque local + cloud |
| **1** copie hors site | Datacenter distant, cloud, stockage offline |

> ⚠️ Une sauvegarde sur le même serveur que les données ne compte pas — elle disparaît avec lui.

---

## 3. rsync — Synchronisation de fichiers

`rsync` copie et synchronise des fichiers efficacement : il ne transfère que les **différences**.

### Syntaxe de base
```bash
rsync [options] source/ destination/
```

### Options essentielles

| Option | Rôle |
|---|---|
| `-a` (archive) | Préserve permissions, dates, liens symboliques, récursif |
| `-v` | Verbose — affiche les fichiers transférés |
| `-z` | Compresse pendant le transfert |
| `-h` | Tailles lisibles (Mo, Go) |
| `--delete` | Supprime dans la destination les fichiers absents de la source |
| `--exclude` | Exclut des fichiers ou dossiers |
| `--dry-run` / `-n` | Simule sans rien modifier |
| `--progress` | Affiche la progression |

### Exemples courants

```bash
# Sauvegarder un dossier local
rsync -avh /var/www/monapp/ /backup/monapp/

# Sauvegarder vers un serveur distant via SSH
rsync -avzh /var/www/monapp/ user@backup-server:/backup/monapp/

# Synchronisation miroir (supprime ce qui n'est plus dans la source)
rsync -avh --delete /source/ /destination/

# Exclure des fichiers
rsync -avh --exclude='*.log' --exclude='.git/' /var/www/ /backup/www/

# Tester avant d'exécuter
rsync -avhn --delete /source/ /destination/
```

---

## 4. Sauvegarde de bases de données

### PostgreSQL

```bash
# Dump d'une base
pg_dump -U postgres -d ma_base > backup_$(date +%Y%m%d).sql

# Dump compressé
pg_dump -U postgres -d ma_base | gzip > backup_$(date +%Y%m%d).sql.gz

# Restauration
psql -U postgres -d ma_base < backup_20240101.sql

# Restauration depuis un fichier compressé
gunzip -c backup_20240101.sql.gz | psql -U postgres -d ma_base

# Dump de toutes les bases
pg_dumpall -U postgres > all_databases_$(date +%Y%m%d).sql
```

### MySQL / MariaDB

```bash
# Dump d'une base
mysqldump -u root -p ma_base > backup_$(date +%Y%m%d).sql

# Dump compressé
mysqldump -u root -p ma_base | gzip > backup_$(date +%Y%m%d).sql.gz

# Restauration
mysql -u root -p ma_base < backup_20240101.sql

# Dump de toutes les bases
mysqldump -u root -p --all-databases > all_databases_$(date +%Y%m%d).sql
```

---

## 5. Sauvegarde des volumes Docker

```bash
# Sauvegarder un volume nommé
docker run --rm \
  -v mon_volume:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/mon_volume_$(date +%Y%m%d).tar.gz -C /data .

# Restaurer un volume
docker run --rm \
  -v mon_volume:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/mon_volume_20240101.tar.gz -C /data
```

---

## 6. Automatisation avec cron

```bash
# Voir les cron jobs actifs
crontab -l

# Éditer les cron jobs
crontab -e
```

### Syntaxe cron
```
# ┌──────────── minute (0-59)
# │ ┌────────── heure (0-23)
# │ │ ┌──────── jour du mois (1-31)
# │ │ │ ┌────── mois (1-12)
# │ │ │ │ ┌──── jour de la semaine (0=dim, 6=sam)
# │ │ │ │ │
  * * * * * commande
```

### Exemples de cron jobs de sauvegarde

```bash
# Sauvegarde PostgreSQL tous les jours à 2h du matin
0 2 * * * pg_dump -U postgres ma_base | gzip > /backup/pg_$(date +\%Y\%m\%d).sql.gz

# Sauvegarde rsync vers serveur distant tous les soirs à 23h
0 23 * * * rsync -azh /var/www/ user@backup:/backup/www/ >> /var/log/backup.log 2>&1

# Nettoyage des sauvegardes de plus de 30 jours
0 3 * * 0 find /backup/ -name "*.sql.gz" -mtime +30 -delete
```

> 💡 Rediriger les sorties vers un log (`>> /var/log/backup.log 2>&1`) pour pouvoir auditer les exécutions.

---

## 7. Rétention et rotation

Conserver toutes les sauvegardes indéfiniment consomme trop d'espace. Stratégie courante :

```
Sauvegardes quotidiennes → garder 7 jours
Sauvegardes hebdomadaires → garder 4 semaines
Sauvegardes mensuelles → garder 12 mois
```

Script de nettoyage :
```bash
# Garder seulement les 7 derniers dumps
find /backup/daily/ -name "*.sql.gz" -mtime +7 -delete
```

---

## 8. Tester la restauration

> **Une sauvegarde non testée n'est pas une sauvegarde.**

Procédure minimale de test :
1. Restaurer sur un serveur de test (pas en production).
2. Vérifier l'intégrité des données (requêtes, checksums).
3. Documenter le temps de restauration (RTO — Recovery Time Objective).
4. Planifier des tests réguliers (mensuel au minimum).

```bash
# Vérifier l'intégrité d'une archive
gzip -t backup_20240101.sql.gz && echo "Archive OK"

# Tester la restauration PostgreSQL en local
createdb test_restore
psql -d test_restore < backup_20240101.sql
psql -d test_restore -c "SELECT COUNT(*) FROM ma_table;"
dropdb test_restore
```

---

## 9. Bonnes pratiques

- Appliquer la règle **3-2-1** systématiquement.
- Toujours utiliser `--dry-run` avant un rsync avec `--delete`.
- Chiffrer les sauvegardes sensibles avant de les envoyer hors site (`gpg`, `openssl`).
- Monitorer l'exécution des cron jobs — une sauvegarde qui échoue silencieusement est invisible.
- Tester la restauration régulièrement — au moins une fois par mois.
- Versionner les scripts de sauvegarde dans Git.

---

## 10. ✅ À retenir

- Règle **3-2-1** : 3 copies, 2 supports, 1 hors site.
- **rsync** = outil de référence pour synchroniser des fichiers (local ou SSH).
- **pg_dump / mysqldump** = dump standard pour les bases SQL.
- **cron** = automatisation des sauvegardes planifiées.
- Une sauvegarde non testée en restauration ne vaut rien.
