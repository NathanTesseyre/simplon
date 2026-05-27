---
title: "Shell scripting bash"
tags:
  - linux
  - bash
  - scripting
  - automatisation
section: 04-Environnements
domaine: Linux
statut: actif
liens_connexes:
  - [[commandes_base]]
  - [[sauvegarde_restauration]]
  - [[services_demarrage]]
---

# 📘 Shell scripting bash

## Pourquoi scripter ?

Un script bash est une suite de commandes enregistrées dans un fichier et exécutées séquentiellement. L'intérêt : automatiser des tâches répétitives, orchestrer plusieurs commandes, créer des outils réutilisables.

Exemples d'usage réels : sauvegardes automatiques, déploiements, nettoyage de logs, monitoring, provisioning de serveurs.

## Structure d'un script

```bash
#!/usr/bin/env bash
# Le shebang (ligne 1) indique quel interpréteur utiliser

set -Eeuo pipefail
# -E : les fonctions héritent du trap ERR
# -e : quitter immédiatement si une commande échoue
# -u : erreur si une variable non définie est utilisée
# -o pipefail : un pipe échoue si l'une de ses commandes échoue
```

Rendre le script exécutable :
```bash
chmod +x mon_script.sh
./mon_script.sh
bash mon_script.sh
```

## Variables

```bash
# Déclaration (pas d'espace autour du =)
nom="Alice"
age=30

# Utilisation (toujours entre guillemets doubles)
echo "Bonjour $nom, tu as $age ans"

# Guillemets simples = pas d'interprétation
echo 'Bonjour $nom'   # affiche littéralement : Bonjour $nom

# Substitution de commande
date_actuelle=$(date +%Y-%m-%d)
nb_fichiers=$(ls /var/log | wc -l)

# Variables spéciales
echo $0    # nom du script
echo $1    # premier argument
echo $#    # nombre d'arguments
echo $?    # code de retour (0 = succès)
echo $$    # PID du script

# Valeurs par défaut
nom="${1:-inconnu}"   # "inconnu" si $1 absent
port="${PORT:-3000}"  # 3000 si PORT non défini
```

## Conditions

```bash
if [ -f "$fichier" ]; then echo "c'est un fichier"; fi
if [ -d "$dossier" ]; then echo "c'est un dossier"; fi
if [ -z "$var" ]; then echo "chaîne vide"; fi
if [ -n "$var" ]; then echo "chaîne non vide"; fi
if [ "$a" = "$b" ]; then echo "égaux"; fi
if [ "$n" -gt 5 ]; then echo "supérieur à 5"; fi
if [ "$n" -lt 5 ]; then echo "inférieur à 5"; fi

# Opérateurs logiques
if [ "$a" = "oui" ] && [ "$b" -gt 0 ]; then echo "les deux vrais"; fi

# Double crochets [[ ]] — bash uniquement, plus puissant
if [[ "$nom" == Alice* ]]; then echo "commence par Alice"; fi
if [[ "$email" =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]; then echo "email valide"; fi
```

## Boucles

```bash
# for sur une liste
for i in 1 2 3 4 5; do
    echo "Iteration $i"
done

# for sur des fichiers
for fichier in /var/log/*.log; do
    echo "$(du -sh "$fichier" | cut -f1)  $fichier"
done

# for style C
for ((i=0; i<10; i++)); do
    echo "i = $i"
done

# while
compteur=0
while [ $compteur -lt 5 ]; do
    echo "compteur = $compteur"
    ((compteur++))
done

# Lire un fichier ligne par ligne
while IFS= read -r ligne; do
    echo "Ligne : $ligne"
done < /etc/hosts

# continue et break
for i in 1 2 3 4 5; do
    [ $i -eq 3 ] && continue   # sauter 3
    [ $i -eq 5 ] && break      # arrêter à 5
    echo $i
done
```

## Fonctions

```bash
# Déclaration
saluer() {
    local nom="$1"    # variable locale
    local age="$2"
    echo "Bonjour $nom, tu as $age ans"
    return 0
}

# Appel
saluer "Alice" 30

# Retourner une valeur via echo
calculer_taille() {
    local chemin="$1"
    du -sh "$chemin" 2>/dev/null | cut -f1
}

taille=$(calculer_taille "/var/log")
echo "Taille : $taille"
```

## Redirections et pipes

```bash
commande > fichier.txt     # stdout vers fichier (écrase)
commande >> fichier.txt    # stdout vers fichier (ajoute)
commande 2> erreurs.txt    # stderr vers fichier
commande 2>&1              # stderr vers stdout
commande > tout.txt 2>&1   # stdout et stderr vers fichier
commande > /dev/null 2>&1  # tout ignorer

# Pipes
ps aux | grep nginx | grep -v grep
cat /var/log/auth.log | grep "Failed" | awk '{print $11}' | sort | uniq -c | sort -rn

# Here document
cat << 'END'
Ligne 1
Ligne 2
END
```

## Gestion des erreurs

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

# Trap — exécuter une fonction à la sortie ou sur erreur
cleanup() {
    echo "Nettoyage..."
    rm -f /tmp/mon_script_*.tmp
}
trap cleanup EXIT    # exécuté à la fin, quelle que soit la cause

# Sortir avec message d'erreur
mourir() {
    echo "ERREUR: $*" >&2
    exit 1
}

# Vérifier les dépendances
for outil in curl jq rsync; do
    command -v "$outil" &>/dev/null || mourir "$outil n'est pas installé"
done
```

## Exemple complet — script de sauvegarde

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

SOURCE="/var/www/app"
DEST="/backup"
DATE=$(date +%Y-%m-%d_%H-%M)
ARCHIVE="$DEST/app_$DATE.tar.gz"
RETENTION=7

[ -d "$SOURCE" ] || { echo "Source inexistante: $SOURCE" >&2; exit 1; }
mkdir -p "$DEST"

echo "Sauvegarde de $SOURCE..."
tar -czf "$ARCHIVE" "$SOURCE"
echo "Archive créée : $ARCHIVE ($(du -sh "$ARCHIVE" | cut -f1))"

echo "Nettoyage des sauvegardes de plus de $RETENTION jours..."
find "$DEST" -name "app_*.tar.gz" -mtime +$RETENTION -delete

echo "Terminé."
```

## Automatisation avec cron

```bash
# Éditer la crontab
crontab -e

# Format : minute heure jour mois jour_semaine commande
0 2 * * *       /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
*/5 * * * *     /opt/scripts/check_api.sh
0 9 * * 1       /opt/scripts/rapport.sh     # lundis à 9h
0 0 1 * *       /opt/scripts/archiver.sh    # 1er du mois à minuit

# Voir la crontab actuelle
crontab -l
```

---

### ✅ À retenir

- Commencer par `#!/usr/bin/env bash` et `set -Eeuo pipefail`
- Variables entre guillemets doubles : `"$var"` (évite les bugs avec espaces)
- `local` dans les fonctions pour ne pas polluer le scope global
- `trap cleanup EXIT` pour nettoyer même en cas d'erreur
- `$(commande)` pour capturer la sortie d'une commande
- `2>&1` pour capturer stdout et stderr ensemble

**Voir aussi →** [[Exercices_Scripts_Linux]] pour pratiquer, [[services_demarrage]] pour les timers systemd.
