# TP1 — Prise en main : compilation et premiers programmes (2h)

## Objectifs
- Installer/vérifier l'environnement (GCC).
- Compiler et exécuter un programme C.
- Manipuler variables, types et opérateurs de base.

## Exercice 1 — Hello World

Écrire un programme qui affiche `"Bonjour, [votre nom] !"`, le compiler avec `gcc -Wall -Wextra -std=c11` et corriger tous les avertissements éventuels.

## Exercice 2 — Calculatrice simple

Écrire un programme qui :
1. demande deux nombres entiers à l'utilisateur ;
2. affiche leur somme, différence, produit, quotient entier et reste ;
3. gère le cas de la division par zéro (afficher un message d'erreur au lieu de planter).

## Exercice 3 — Conversion de température

Écrire un programme convertissant une température de Celsius en Fahrenheit (`F = C * 9/5 + 32`) en utilisant des `float`. Vérifier ce qui se passe si l'on utilise `int` à la place (perte de précision).

## Exercice 4 — Échange de deux variables

Écrire une fonction `echanger` qui échange les valeurs de deux entiers **sans utiliser de variable temporaire supplémentaire visible côté appelant** (indice : ce sera repris avec les pointeurs au TP6 — ici, se contenter d'un échange arithmétique dans `main`).

## À rendre

Un fichier `tp1.c` par exercice (ou un seul fichier avec un `main` qui appelle chaque exercice), compilable sans avertissement.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
