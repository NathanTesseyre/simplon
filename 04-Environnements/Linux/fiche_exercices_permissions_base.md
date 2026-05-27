# 📄 Fiche d’exercices — Gestion des droits (basique)

## 0) Pré-requis
Exécutez le script `setup_permissions_base.sh` puis travaillez dans `~/exercices_permissions_base`.

## A. Lire & comprendre
1. **Depuis** `~/exercices_permissions_base`, affichez les droits **du dossier** `docs_projets_2025_partage_equipe`. Que signifient les 10 caractères de `ls -ld` ?
2. **Allez dans** `documents_confidentiels_direction` et listez en format long. Vérifiez que **seul le propriétaire** a `rw` sur `rapport_budgetaire_previsionnel_2025_confidentiel.txt`.
3. **Depuis** `ressources_formations_utilisateurs`, affichez le mode **octal** exact de `guide_utilisateur_demarrage_rapide_v1.txt` (avec `stat`).

## B. `chmod` symbolique
4. **Dans** `scripts_operations_quotidiennes`, rendez **exécutable** pour le **propriétaire** seulement `script_generation_rapport_quotidien.sh` sans toucher aux autres droits.
5. **Toujours ici**, donnez **lecture** au **groupe** sur `script_nettoyage_temporaire.sh` sans donner d’exécution aux autres.
6. **Revenez** à la racine et rendez `docs_projets_2025_partage_equipe/liste_taches_projet_alpha_sprint_12.txt` **non modifiable** par “autres” (conservez le reste).

## C. `chmod` octal
7. **Depuis** `documents_confidentiels_direction`, mettez `rapport_budgetaire_previsionnel_2025_confidentiel.txt` en **600** via octal.
8. **Depuis** `zone_echanges_temps_limite`, passez `fichier_partage_modifiable_par_tous.txt` en **664** via octal.
9. **Depuis** la racine, mettez `ressources_formations_utilisateurs/guide_utilisateur_demarrage_rapide_v1.txt` en **640** via octal.

## D. Répertoires : lecture/traversée
10. **Depuis** la racine, **retirez** temporairement `x` pour **groupe** et **autres** sur `zone_echanges_temps_limite`. Essayez `ls zone_echanges_temps_limite`. Qu’observez‑vous ?
11. **Rétablissez** `x` à **groupe** et **autres** sur ce dossier pour permettre la traversée. Vérifiez avec `ls`.
12. **Dans** `docs_projets_2025_partage_equipe`, créez un sous-dossier `documents_relecture_qualite` puis donnez‑lui, en **une seule commande octale**, les droits : **u=rwx, g=rx, o=r**.

## E. Mise en conformité
13. **Depuis** la racine, donnez **lecture** à tout le monde sur **tous les fichiers** du dossier `ressources_formations_utilisateurs` (sans toucher aux exécutions).
14. **Toujours depuis** la racine, assurez que **seuls** les scripts du dossier `scripts_operations_quotidiennes` soient **exécutables par le propriétaire** (pas par groupe/autres).
15. **Depuis** la racine, rendez tous les fichiers de `documents_confidentiels_direction` **inaccessibles** aux “autres” (ni r, ni w, ni x), en **symbolique**.

## F. Erreurs volontaires & corrections
16. **Dans** `docs_projets_2025_partage_equipe`, mettez `liste_taches_projet_alpha_sprint_12.txt` en **666**, puis corrigez en **664**.
17. **Dans** `scripts_operations_quotidiennes`, mettez par erreur `script_nettoyage_temporaire.sh` en **777**, puis **retirez** `x` pour groupe/autres en **symbolique** (le propriétaire garde `rwx`).

## G. Vérification finale
18. **Depuis** la racine, affichez un tableau des fichiers à **profondeur 2** (`find . -maxdepth 2 -type f -exec ls -l {} \;`) et repérez :
   - les fichiers où “autres” ont encore `w`,
   - les scripts qui ne sont **pas** exécutables par le propriétaire.
