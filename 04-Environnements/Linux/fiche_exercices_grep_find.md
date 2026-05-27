# Fiche d’exercices – grep & find (navigation incluse)

## Pré-requis
Exécutez d’abord le script `setup_exercices_grep_find.sh` pour créer l’environnement :
```shell
chmod +x setup_exercices_grep_find.sh
./setup_exercices_grep_find.sh
```
Le dossier de travail sera `~/exercices_grep_find` avec des **noms longs** pour vous entraîner à la **complétion par TAB**.

---

## Exercices `grep` (avec déplacements)

1. **Depuis** `~/exercices_grep_find`, affichez les lignes contenant **Linux** dans `informations_systemes_operatifs.txt` (numéros de ligne et couleurs).
2. **Allez dans** `dossiers_temps_de_travail_utilisateur/`, puis recherchez (insensible à la casse) le mot **root** dans `../donnees_utilisateurs_passwd.txt`.
3. **Depuis** la racine des exercices, affichez les lignes contenant **TODO** dans `suivi_developpement_logiciel.txt` avec le **numéro de ligne**.
4. **Allez dans** `journaux_applications_serveur_web/` et cherchez **récursivement** toutes les lignes contenant **error** dans les fichiers `*.log`.
5. **Depuis** `scripts_utilitaires_de_sauvegarde/`, cherchez toutes les lignes contenant **echo** dans les scripts du dossier courant.

---

## Exercices `find` (avec déplacements)

1. **Depuis** `~/exercices_grep_find`, trouvez tous les fichiers `*.txt`.
2. **Dans** `scripts_utilitaires_de_sauvegarde/`, trouvez tous les fichiers `*.sh`.
3. **Depuis** `journaux_applications_serveur_web/`, listez les fichiers de **plus de 10 Mo**.
4. **Dans** `dossiers_temps_de_travail_utilisateur/`, trouvez les fichiers modifiés dans les dernières **24 heures**.
5. **Depuis** la racine des exercices, listez **uniquement les dossiers** (type `d`).

---

## Exercices combinés (`find` + `grep`)

1. **Depuis** la racine des exercices, trouvez tous les fichiers `.log` dans `journaux_applications_serveur_web/` qui contiennent **error** (affichez le nom du fichier et le numéro de ligne).
2. **Dans** `scripts_utilitaires_de_sauvegarde/`, recherchez dans tous les scripts `.sh` les lignes contenant **echo**.
3. **Depuis** `dossiers_temps_de_travail_utilisateur/`, cherchez dans tout le dossier **parent** (`..`) les fichiers contenant le mot **backup** (n’affichez que les noms de fichiers).
