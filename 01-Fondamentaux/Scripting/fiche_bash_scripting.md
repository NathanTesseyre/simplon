---
title: "Bash scripting — fondamentaux"
tags:
  - fondamentaux
  - bash
  - scripting
  - cron
section: 01-Fondamentaux
domaine: Scripting
statut: actif
liens_connexes:
  - [[commandes_base]]
  - [[fiche_processus_threads]]
  - [[services_demarrage]]
---

# 🐚 Fiche : Bash scripting — fondamentaux
*(à destination d'administrateurs système en formation)*

---

## 1. Structure d'un script

```bash
#!/usr/bin/env bash
# Shebang : indique l'interpréteur à utiliser

set -e          # Arrête le script si une commande échoue
set -u          # Erreur si une variable non définie est utilisée
set -o pipefail # Propage les erreurs dans les pipes

echo "Hello depuis le script"
```

```bash
# Rendre exécutable et lancer
chmod +x mon_script.sh
./mon_script.sh

# Ou sans rendre exécutable
bash mon_script.sh
```

> **Toujours mettre `set -euo pipefail`** en début de script : ça évite les comportements silencieux et dangereux.

---

## 2. Variables

### Déclaration et utilisation
```bash
nom="Alice"
age=30

echo "Bonjour $nom"          # Variable dans une chaîne
echo "Tu as ${age} ans"      # Accolades (obligatoires si suivi de lettres)
echo "Tu as $((age + 1)) ans dans un an"  # Arithmétique
```

### Variables d'environnement
```bash
export MA_VAR="valeur"       # Disponible pour les processus enfants
echo $PATH                   # Variable système
echo $HOME                   # Répertoire home de l'utilisateur
echo $USER                   # Nom d'utilisateur courant
```

### Variables spéciales
| Variable | Signification |
|---|---|
| `$0` | Nom du script |
| `$1`, `$2`... | Arguments positionnels |
| `$#` | Nombre d'arguments |
| `$@` | Tous les arguments (liste) |
| `$*` | Tous les arguments (chaîne) |
| `$?` | Code de retour de la dernière commande |
| `$$` | PID du script en cours |
| `$!` | PID de la dernière commande en arrière-plan |

```bash
#!/usr/bin/env bash
echo "Script : $0"
echo "1er arg : $1"
echo "Nb args : $#"
echo "Tous : $@"
```

### Quotes
```bash
nom="monde"
echo "$nom"        # → monde         (interpolation)
echo '$nom'        # → $nom          (littéral, pas d'interpolation)
echo "prix: $(( 3 * 5 ))€"  # → prix: 15€
```

---

## 3. Conditions

### if / elif / else
```bash
if [ "$age" -ge 18 ]; then
    echo "Majeur"
elif [ "$age" -ge 13 ]; then
    echo "Adolescent"
else
    echo "Enfant"
fi
```

### Opérateurs de comparaison

**Entiers :**
| Opérateur | Signification |
|---|---|
| `-eq` | égal |
| `-ne` | différent |
| `-lt` | inférieur strict |
| `-le` | inférieur ou égal |
| `-gt` | supérieur strict |
| `-ge` | supérieur ou égal |

**Chaînes :**
| Opérateur | Signification |
|---|---|
| `=` ou `==` | égal |
| `!=` | différent |
| `-z "$s"` | chaîne vide |
| `-n "$s"` | chaîne non vide |

**Fichiers :**
| Opérateur | Signification |
|---|---|
| `-f "$f"` | fichier ordinaire existe |
| `-d "$d"` | répertoire existe |
| `-e "$p"` | chemin existe (fichier ou dossier) |
| `-r "$f"` | fichier lisible |
| `-x "$f"` | fichier exécutable |

### Double crochets `[[ ]]` (préférable en bash)
```bash
if [[ "$nom" == "Alice" ]]; then echo "Bonjour Alice"; fi
if [[ "$fichier" =~ \.log$ ]]; then echo "C'est un log"; fi   # regex
if [[ -f "/etc/hosts" && -r "/etc/hosts" ]]; then echo "Lisible"; fi
```

---

## 4. Boucles

### for — itérer sur une liste
```bash
for fruit in pomme poire banane; do
    echo "Fruit : $fruit"
done

# Itérer sur des fichiers
for fichier in /var/log/*.log; do
    echo "Taille : $(du -sh "$fichier")"
done

# Boucle numérique (C-style)
for (( i=0; i<5; i++ )); do
    echo "i = $i"
done

# Avec seq
for i in $(seq 1 10); do
    echo "$i"
done
```

### while — tant que la condition est vraie
```bash
compteur=0
while [ $compteur -lt 5 ]; do
    echo "Compteur : $compteur"
    (( compteur++ ))
done

# Lire un fichier ligne par ligne
while IFS= read -r ligne; do
    echo "$ligne"
done < /etc/hosts
```

### until — jusqu'à ce que la condition soit vraie
```bash
until ping -c1 8.8.8.8 &>/dev/null; do
    echo "Réseau indisponible, attente..."
    sleep 5
done
echo "Réseau disponible !"
```

---

## 5. Fonctions

```bash
# Déclaration
saluer() {
    local prenom="$1"         # variable locale à la fonction
    echo "Bonjour, $prenom !"
    return 0                  # code de retour (0 = succès)
}

# Appel
saluer "Alice"
saluer "Bob"

# Récupérer une valeur de retour (via echo + substitution)
obtenir_date() {
    echo "$(date +%Y-%m-%d)"
}
date_aujourd_hui=$(obtenir_date)
echo "Aujourd'hui : $date_aujourd_hui"
```

> **`local`** est important : sans lui, les variables de fonction sont globales et peuvent écraser d'autres variables.

---

## 6. Exit codes et gestion d'erreurs

```bash
# 0 = succès, tout autre code = erreur
cp fichier.txt /backup/ && echo "Copie OK" || echo "Erreur de copie"

# Vérifier le code de retour
if ! mkdir /tmp/mondossier; then
    echo "Impossible de créer le dossier" >&2
    exit 1
fi

# Fonction d'erreur réutilisable
erreur() {
    echo "[ERREUR] $*" >&2
    exit 1
}

[ -f "$1" ] || erreur "Fichier $1 introuvable"
```

| Code | Signification courante |
|---|---|
| `0` | Succès |
| `1` | Erreur générique |
| `2` | Mauvaise utilisation (arguments) |
| `126` | Commande non exécutable |
| `127` | Commande introuvable |
| `130` | Interruption par Ctrl+C |

---

## 7. Redirections et pipes

```bash
# Rediriger la sortie standard
commande > fichier.txt         # écrase
commande >> fichier.txt        # ajoute

# Rediriger les erreurs
commande 2> erreurs.txt
commande 2>/dev/null           # ignorer les erreurs

# Tout rediriger
commande > tout.txt 2>&1
commande &> tout.txt           # raccourci bash

# Pipe : sortie d'une commande → entrée d'une autre
cat /etc/passwd | grep "root" | cut -d: -f1,3

# Substitution de commande
nb_lignes=$(wc -l < /etc/hosts)
echo "Il y a $nb_lignes lignes"

# Here document (texte multiligne)
cat <<EOF > /tmp/config.txt
serveur=192.168.1.1
port=8080
EOF
```

---

## 8. Tableaux (arrays)

```bash
# Déclaration
serveurs=("web01" "web02" "db01")

# Accès
echo "${serveurs[0]}"          # web01
echo "${serveurs[@]}"          # tous les éléments
echo "${#serveurs[@]}"         # nombre d'éléments

# Itérer
for srv in "${serveurs[@]}"; do
    echo "Ping $srv : $(ping -c1 -W1 "$srv" &>/dev/null && echo OK || echo KO)"
done

# Ajouter un élément
serveurs+=("db02")
```

---

## 9. Cron — planification de tâches

### Syntaxe crontab
```
┌─────── minute (0-59)
│ ┌───── heure (0-23)
│ │ ┌─── jour du mois (1-31)
│ │ │ ┌─ mois (1-12)
│ │ │ │ ┌ jour de la semaine (0-7, 0 et 7 = dimanche)
│ │ │ │ │
* * * * *  commande
```

### Exemples
```bash
# Ouvrir l'éditeur crontab de l'utilisateur courant
crontab -e

# Lister les tâches planifiées
crontab -l

# Exemples de tâches
0 2 * * *       /usr/local/bin/backup.sh         # tous les jours à 2h
*/5 * * * *     /usr/bin/check_disk.sh           # toutes les 5 minutes
0 0 * * 1       /usr/bin/rapport_hebdo.sh        # lundi à minuit
@reboot         /usr/local/bin/init_serveur.sh   # au démarrage
```

### Bonnes pratiques cron
```bash
# Toujours utiliser des chemins absolus
0 3 * * * /usr/bin/find /tmp -mtime +7 -delete

# Rediriger les sorties pour éviter les mails
0 4 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# Fichiers système (root) : /etc/cron.d/, /etc/cron.daily/, etc.
```

---

## 10. Exemple de script complet

```bash
#!/usr/bin/env bash
set -euo pipefail

# Script de sauvegarde simple
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"
DATE=$(date +%Y%m%d_%H%M%S)
ARCHIVE="${BACKUP_DIR}/www_${DATE}.tar.gz"

log() { echo "[$(date '+%H:%M:%S')] $*"; }
erreur() { echo "[ERREUR] $*" >&2; exit 1; }

[ -d "$SOURCE_DIR" ] || erreur "Répertoire source $SOURCE_DIR introuvable"
mkdir -p "$BACKUP_DIR"

log "Début de la sauvegarde de $SOURCE_DIR"
if tar -czf "$ARCHIVE" "$SOURCE_DIR"; then
    log "Archive créée : $ARCHIVE ($(du -sh "$ARCHIVE" | cut -f1))"
else
    erreur "Échec de la création de l'archive"
fi

# Supprimer les archives de plus de 30 jours
find "$BACKUP_DIR" -name "www_*.tar.gz" -mtime +30 -delete
log "Anciennes sauvegardes nettoyées"
```

---

## 11. À retenir pour un admin sys

- Toujours commencer par `#!/usr/bin/env bash` + `set -euo pipefail`.
- `$?` donne le code de retour ; `0` = succès, tout autre = erreur.
- Préférer `[[ ]]` à `[ ]` : plus robuste et lisible.
- Utiliser `local` dans les fonctions pour éviter les effets de bord.
- En cron, **toujours des chemins absolus** et **toujours rediriger les sorties**.
