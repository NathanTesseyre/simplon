# Fiche théorique – grep & find (options nécessaires)

## Rappel `echo -e` et alternative portable
- `echo -e` (selon le shell) interprète les échappements : `\n` (nouvelle ligne), `\t` (tabulation)…
- Pour éviter les différences entre shells, **préférez `printf`** : `printf "ligne 1\nligne 2\n"`

---

## grep – rechercher du texte dans des fichiers
**Syntaxe :**
```
grep [options] "motif" [chemins|fichiers...]
```

**Options utiles pour ces exercices :**
- `-i` : insensible à la casse
- `-n` : afficher le **numéro de ligne**
- `-r` (ou `-R`) : recherche **récursive** dans les sous-dossiers
- `-H` / `-h` : forcer/masquer le nom de **fichier** dans la sortie
- `-w` : motif comme **mot entier**
- `-v` : **inverse** (ne pas correspondre)
- `-E` : **expressions régulières étendues** (alias `egrep`), ex. `user[0-9]+`
- `-o` : n’afficher que la **partie correspondante**
- `-A N` / `-B N` / `-C N` : N lignes **après / avant / de contexte**
- `--color=auto` : surlignage du motif
- `--include="*.ext"` / `--exclude="*.ext"` : filtrer les fichiers en récursif
- `--exclude-dir=DIR` : exclure des dossiers en récursif
- `-l` / `-L` : n’afficher que les **noms de fichiers** (qui correspondent / ne correspondent pas)
- `-c` : compter les correspondances par fichier
- `-m N` : **stopper après N** correspondances (utile sur gros arbres)
- `-s` : silencieux (masque certaines erreurs)
- `-I` : traiter les binaires comme sans texte (évite du bruit)

**Exemples :**
```bash
grep -n --color=auto "Linux" informations_systemes_operatifs.txt
grep -in "root" ../donnees_utilisateurs_passwd.txt
grep -n "TODO" ../suivi_developpement_logiciel.txt
grep -rn --include="*.log" --color=auto "error" journaux_applications_serveur_web
grep -n "echo" scripts_utilitaires_de_sauvegarde/*.sh
```

---

## find – rechercher des fichiers/dossiers par critères
**Syntaxe :**
```
find [chemin...] [tests/critères] [actions]
```

**Tests & filtres :**
- `-type f|d|l` : fichier | dossier | lien symbolique
- `-name "pat"` / `-iname "pat"` : par nom (sensible / insensible à la casse) – ex. `"*.txt"`
- `-size +10M` : taille (`k`=Kio, `M`=Mio, `G`=Gio, `c`=octets)
- `-mtime -1` / `-mmin -60` : modifiés il y a **moins d’1 jour / 60 min**
- `-user USER` / `-group GROUP` : propriétaire / groupe
- `-perm MODE` : permissions (ex. `-perm 0755` ou `-perm -u=x`)
- `-maxdepth N` / `-mindepth N` : profondeur de recherche
- Logique : `-not` (ou `!`), `-o` (OR), parenthèses `\( ... \)`

**Actions :**
- `-print` (par défaut)
- `-exec CMD {} \;` : exécuter **par fichier**
- `-exec CMD {} +` : exécuter par **lot** (plus efficace)
- `-delete` : supprimer (⚠️ dangereux, tester d’abord sans)
- `-ls`, `-printf` : affichage détaillé
- `-prune` : **ne pas descendre** dans certains dossiers

**Exemples :**
```bash
find . -type f -name "*.txt"
find scripts_utilitaires_de_sauvegarde -type f -name "*.sh"
find journaux_applications_serveur_web -type f -size +10M -print
find dossiers_temps_de_travail_utilisateur -type f -mtime -1 -print
find . -type d -print
```

---

## Combiner find + grep
```bash
find journaux_applications_serveur_web -type f -name "*.log" -exec grep -nH --color=auto "error" {} \;
find scripts_utilitaires_de_sauvegarde -type f -name "*.sh" -exec grep -nH "echo" {} \;
find .. -type f -exec grep -l "backup" {} \;
```
