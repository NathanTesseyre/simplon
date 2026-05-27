## Astuce générale

👉 Quand tu ne connais pas bien une commande, pense à utiliser :  
- `man commande` pour lire le manuel complet (ex: `man ls`)  
- `commande --help` pour une aide rapide (ex: `ls --help`)

---

## Exercices de base

1. **Premier script**  
   Crée un script qui affiche « Bonjour » suivi du nom de l’utilisateur connecté.  

2. **Variables et echo**  
   Crée un script qui définit deux variables : ton prénom et ton nom.  
   Le script doit ensuite afficher les deux sur une même ligne, séparés par un espace.  

3. **Utilisation de `date`**  
   Crée un script qui affiche la date et l’heure actuelles au format suivant : `Jour Mois Année - Heure:Minutes`.  
   🔹 *Indice* : la commande `date` peut être formatée, par exemple `date +%d-%m-%Y`.  

---

## Exercices intermédiaires

4. **Lecture utilisateur (read)**  
   Écris un script qui demande à l’utilisateur son plat préféré et affiche ensuite un message personnalisé.  

5. **Arguments**  
   Crée un script qui prend deux arguments et affiche :  
   - « Premier argument : … »  
   - « Deuxième argument : … »  

6. **Conditions**  
   Écris un script qui teste si l’utilisateur est **root** ou non, puis affiche un message adapté.  

7. **Boucle for simplifiée**  
   Écris un script qui affiche les nombres de 1 à 10 sur une ligne chacun.  

---

## Exercices avancés

8. **Boucle for classique**  
   Crée un script qui utilise une boucle `for ((…))` pour afficher la table de multiplication de 7 (de 1 à 10).  

9. **Test d’existence de fichier**  
   Écris un script qui demande à l’utilisateur un nom de fichier, puis teste s’il existe. Affiche un message différent selon le cas.  
   🔹 *Indice* : en bash, tu peux tester l’existence d’un fichier avec l’expression `[ -f nomfichier ]`.  

10. **Redirection d’erreurs**  
   Crée un script qui essaie de lister un fichier qui n’existe pas et redirige l’erreur dans un fichier `erreurs.log`.  

---

## Exercices pratiques

11. **Sauvegarde simple**  
   Écris un script qui copie un dossier de ton choix vers un répertoire `backup/` en ajoutant la date du jour dans le nom.  
   🔹 *Indice* : pour créer un dossier, utilise `mkdir -p`. Pour copier un dossier entier, regarde du côté de `cp -r`.  

12. **Mini-menu interactif**  
   Crée un script qui affiche un petit menu et propose à l’utilisateur trois choix :  

   ```
   1) Afficher la date
   2) Afficher les fichiers du répertoire courant
   3) Quitter
   ```

   L’utilisateur doit pouvoir entrer un nombre (1, 2 ou 3), et le script exécute l’action correspondante.  

   🔹 *Indice* : tu auras besoin d’une boucle `while` et d’un `case` pour gérer les choix.  
