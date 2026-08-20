# TP6 — Pointeurs et allocation dynamique (3h)

## Objectifs
- Manipuler pointeurs et arithmétique de pointeurs.
- Allouer, redimensionner et libérer correctement de la mémoire dynamique.
- Détecter et corriger des erreurs mémoire classiques.

## Exercice 1 — Bases

Écrire un programme illustrant : déclaration d'un pointeur, affectation d'une adresse, déréférencement, modification d'une variable via son pointeur, affichage d'une adresse (`%p`).

## Exercice 2 — Tableau dynamique

Écrire un programme qui :
1. demande à l'utilisateur combien d'éléments il souhaite saisir ;
2. alloue dynamiquement un tableau d'`int` de cette taille avec `malloc` (en vérifiant le retour) ;
3. remplit le tableau ;
4. calcule la somme et la moyenne ;
5. libère la mémoire avec `free`.

## Exercice 3 — Redimensionnement

Étendre l'exercice 2 : permettre à l'utilisateur d'ajouter des éléments un par un, en utilisant `realloc` pour agrandir le tableau à chaque ajout (doubler la capacité plutôt que réallouer à chaque insertion, et expliquer pourquoi cette stratégie est plus efficace).

## Exercice 4 — Liste chaînée simple

À partir de la structure `Noeud` vue en cours, implémenter :
- `Noeud* creer_noeud(int valeur)`
- `void inserer_debut(Noeud **tete, int valeur)`
- `void afficher_liste(Noeud *tete)`
- `void liberer_liste(Noeud **tete)` (parcourt et libère chaque nœud, puis met `*tete = NULL`)

## Exercice 5 — Chasse aux bugs mémoire (analyse de code)

L'enseignant fournit un court programme C contenant volontairement un *use-after-free*, un *double free* et une fuite mémoire. Identifier chaque bug, l'expliquer par écrit, et proposer la correction. Vérifier avec `valgrind --leak-check=full ./programme` ou `-fsanitize=address,leak`.

```bash
gcc -Wall -Wextra -g -fsanitize=address,leak -o programme programme.c
./programme
```

## À rendre

Fichiers `tp6_ex2.c` à `tp6_ex4.c`, plus un fichier `tp6_ex5_analyse.md` documentant les bugs trouvés et leur correction.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
