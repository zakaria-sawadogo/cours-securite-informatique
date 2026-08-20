# TP3 — Fonctions (2h)

## Objectifs
- Écrire des fonctions avec prototypes, paramètres par valeur et par adresse.
- Comprendre la portée des variables et la récursivité.

## Exercice 1 — Bibliothèque de fonctions mathématiques

Écrire (avec prototypes séparés) :
- `int est_premier(int n)`
- `long factorielle(int n)` (version itérative)
- `long factorielle_recursive(int n)` (version récursive)
- `int pgcd(int a, int b)` (algorithme d'Euclide)

Comparer dans `main` les deux versions de factorielle pour n = 15 et n = 25 (observer le dépassement de capacité d'un `long` au-delà d'un certain n : lien avec les débordements d'entiers vus au chapitre 2).

## Exercice 2 — Passage par adresse

Écrire une fonction `void echanger(int *a, int *b)` qui échange réellement le contenu de deux variables, et une fonction `void minmax(int tab[], int taille, int *min, int *max)` qui renvoie le min et le max d'un tableau via deux pointeurs de sortie.

## Exercice 3 — Portée et `static`

Écrire une fonction `int compteur_appels(void)` utilisant une variable `static` locale qui s'incrémente à chaque appel. Vérifier expérimentalement que la valeur persiste entre les appels, contrairement à une variable locale ordinaire.

## Exercice 4 — Récursivité : suite de Fibonacci

Écrire une version récursive puis itérative du calcul du n-ième terme de Fibonacci. Mesurer (avec `clock()` de `time.h`) le temps d'exécution pour n = 30 dans les deux versions et expliquer la différence de complexité (O(2ⁿ) contre O(n)).

## À rendre

Fichier `tp3.c` avec toutes les fonctions prototypées en haut de fichier, définies après `main`.
