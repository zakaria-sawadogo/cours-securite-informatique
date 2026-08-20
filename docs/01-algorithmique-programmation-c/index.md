# Algorithmique et Programmation en C

**Volume horaire :** 15h CM · 15h TP (30h)

## Objectifs pédagogiques

À l'issue du cours, l'étudiant est capable de :

- concevoir un algorithme structuré pour résoudre un problème donné, avant toute écriture de code ;
- écrire, compiler et déboguer des programmes en langage C ;
- manipuler les structures de données fondamentales (tableaux, chaînes, structures) et la mémoire (pointeurs, allocation dynamique) ;
- lire et modifier du code C existant, compétence indispensable pour l'analyse de vulnérabilités, l'exploitation logicielle et la rétro-ingénierie abordées plus loin dans le programme (Sécurité des applications, Cryptanalyse).

## Prérequis

Aucun prérequis en programmation n'est exigé ; des bases de logique mathématique (algèbre de Boole, notions d'arithmétique) sont utiles.

## Plan de séances

### Cours magistraux (15h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction à l'algorithmique et au langage C](cours/01-introduction-algorithmique-et-c.md) | 2h |
| 2 | [Variables, types, opérateurs et entrées-sorties](cours/02-variables-types-operateurs-es.md) | 2h |
| 3 | [Structures de contrôle](cours/03-structures-de-controle.md) | 2h |
| 4 | [Fonctions et portée des variables](cours/04-fonctions-portee.md) | 2h |
| 5 | [Tableaux et chaînes de caractères](cours/05-tableaux-chaines.md) | 3h |
| 6 | [Pointeurs et gestion mémoire](cours/06-pointeurs-memoire.md) | 2h |
| 7 | [Structures (`struct`) et fichiers](cours/07-structures-fichiers.md) | 2h |

### Travaux pratiques (15h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Prise en main : compilation et premiers programmes](tp/tp1-prise-en-main.md) | 2h |
| 2 | [Structures de contrôle](tp/tp2-structures-controle.md) | 2h |
| 3 | [Fonctions](tp/tp3-fonctions.md) | 2h |
| 4 | [Tableaux](tp/tp4-tableaux.md) | 2h |
| 5 | [Chaînes de caractères](tp/tp5-chaines.md) | 2h |
| 6 | [Pointeurs et allocation dynamique](tp/tp6-pointeurs.md) | 3h |
| 7 | [Mini-projet : gestionnaire de contacts](tp/tp7-mini-projet.md) | 2h |

## Évaluation

- Contrôle continu (exercices notés en TP) : 40 %
- Mini-projet final (TP7) : 20 %
- Examen final sur table (algorithmique + lecture/écriture de code C) : 40 %

## Environnement technique

- Compilateur : GCC (`gcc -Wall -Wextra -std=c11`)
- Éditeur recommandé : VS Code, ou tout éditeur avec coloration syntaxique C
- Système : Linux/macOS/WSL recommandé pour la cohérence des TP

## Bibliographie

- B. W. Kernighan, D. M. Ritchie, *The C Programming Language*, 2nd ed.
- C. Delannoy, *Programmer en langage C*, Eyrolles.
- Documentation : [cppreference.com/w/c](https://en.cppreference.com/w/c)
