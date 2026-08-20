# TP5 — Chaînes de caractères (2h)

## Objectifs
- Manipuler des chaînes C de façon sûre.
- Réimplémenter des fonctions de `string.h` pour en comprendre le fonctionnement interne.

## Exercice 1 — Réimplémentation de fonctions standards

Sans utiliser `string.h`, écrire :
- `size_t ma_strlen(const char *s)`
- `void ma_strcpy(char *dst, const char *src)` puis une version bornée `ma_strncpy(char *dst, const char *src, size_t n)`
- `int ma_strcmp(const char *a, const char *b)`

## Exercice 2 — Palindrome

Écrire une fonction `int est_palindrome(const char *s)` qui vérifie si une chaîne se lit de la même façon dans les deux sens (en ignorant la casse et les espaces).

## Exercice 3 — Comptage et statistiques

Écrire un programme qui, à partir d'une phrase saisie, affiche : le nombre de voyelles, le nombre de mots, et le mot le plus long.

## Exercice 4 — Démonstration encadrée d'un débordement

Sous supervision, comparer :

```c
char buffer[8];
strcpy(buffer, "Ceci est bien plus long que 8 caractères"); // dangereux
```

avec la version sûre :

```c
char buffer[8];
snprintf(buffer, sizeof(buffer), "%s", "Ceci est bien plus long que 8 caractères");
```

Compiler la première version avec `-fsanitize=address` pour observer la détection du débordement, puis expliquer par écrit pourquoi `snprintf` est préférable.

## Exercice 5 — Chiffrement de César (préparation au cours de Cryptographie)

Écrire une fonction `void cesar(char *texte, int decalage)` qui chiffre une chaîne de lettres majuscules par décalage circulaire dans l'alphabet (ex. décalage 3 : A→D, Z→C). Cette fonction sera réutilisée en TP de Cryptographie.

## À rendre

Fichier `tp5.c` avec toutes les fonctions, plus la note explicative de l'exercice 4.
