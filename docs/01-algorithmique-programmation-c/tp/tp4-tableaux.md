# TP4 — Tableaux (2h)

## Objectifs
- Manipuler des tableaux à une et deux dimensions.
- Implémenter des algorithmes de recherche et de tri.

## Exercice 1 — Statistiques sur un tableau

Écrire des fonctions `somme`, `moyenne`, `min`, `max`, `ecart_type` opérant sur un tableau de `float` de taille donnée.

## Exercice 2 — Recherche

Implémenter :
- `int recherche_sequentielle(int tab[], int taille, int valeur)`
- `int recherche_dichotomique(int tab[], int taille, int valeur)` (le tableau doit être trié)

Comparer le nombre de comparaisons effectuées par les deux méthodes sur un tableau de 1000 éléments.

## Exercice 3 — Tri

Implémenter le **tri à bulles** et le **tri par insertion**. Afficher le tableau après chaque passe pour visualiser le déroulement de l'algorithme.

## Exercice 4 — Matrices

Écrire un programme qui remplit une matrice 3x3 saisie par l'utilisateur, calcule sa transposée, et affiche les deux matrices côte à côte.

## Exercice 5 — Détection de dépassement (démonstration pédagogique encadrée)

Sous supervision de l'enseignant, illustrer avec `-fsanitize=address` (AddressSanitizer) ce qui se passe lors d'un accès `tab[taille]` hors borne, pour observer concrètement un débordement de tampon détecté par l'outil :

```bash
gcc -Wall -Wextra -fsanitize=address -g -o demo demo.c
./demo
```

## À rendre

Fichier `tp4.c` et une courte note (5 lignes) expliquant la différence de complexité entre recherche séquentielle et dichotomique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
