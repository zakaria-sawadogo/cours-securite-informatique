# Chapitre 1 — Introduction à l'algorithmique et au langage C

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/01-introduction-algorithmique-et-c.txt){ target=_blank }

## 1. Qu'est-ce qu'un algorithme ?

Un algorithme est une suite finie et non ambiguë d'instructions permettant de résoudre un problème ou d'accomplir une tâche à partir de données d'entrée, en produisant un résultat en un temps fini. Trois propriétés le caractérisent :

- **Finitude** : l'algorithme se termine après un nombre fini d'étapes, quelles que soient les données d'entrée (dans leur domaine de validité).
- **Déterminisme** : chaque étape est définie sans ambiguïté ; à chaque instant, on sait exactement quelle instruction exécuter ensuite.
- **Effectivité** : chaque étape est suffisamment simple et concrète pour être réellement réalisable (par une machine ou, historiquement, par un humain suivant une procédure).

On distingue l'algorithme (la méthode, indépendante de tout langage) du **programme** (sa traduction dans un langage de programmation précis) et de l'**exécution** (le déroulement effectif du programme sur une machine, avec des données concrètes). Un même algorithme peut être implémenté dans des dizaines de langages différents ; c'est pourquoi il est fondamental d'apprendre à concevoir un algorithme *avant* de penser à sa syntaxe C.

Avant d'écrire du code, on décrit donc un algorithme en **pseudo-code**, une notation semi-formelle indépendante de tout langage, qui privilégie la clarté sur la rigueur syntaxique :

```
ALGORITHME Maximum
ENTRÉE : a, b (deux entiers)
SORTIE : le plus grand des deux
DÉBUT
    SI a > b ALORS
        RETOURNER a
    SINON
        RETOURNER b
    FIN SI
FIN
```

### Un second exemple : tester la parité d'un entier

```
ALGORITHME EstPair
ENTRÉE : n (un entier)
SORTIE : VRAI si n est pair, FAUX sinon
DÉBUT
    SI (n MOD 2) = 0 ALORS
        RETOURNER VRAI
    SINON
        RETOURNER FAUX
    FIN SI
FIN
```

### Correction d'un algorithme : une première intuition

Concevoir un algorithme ne suffit pas : il faut aussi être capable d'argumenter qu'il fait bien ce qu'on attend de lui, pour toutes les entrées valides — et pas seulement pour les quelques cas que l'on a testés « à la main ». Deux notions, formalisées plus tard dans le cursus (analyse d'algorithmes), sont utiles dès maintenant :

- la **précondition** : ce que l'on suppose vrai sur les données d'entrée avant l'exécution (ex. « `b` est différent de zéro » pour une division) ;
- la **postcondition** : ce que l'algorithme garantit être vrai en sortie, si la précondition était satisfaite.

Cette habitude de raisonner en préconditions/postconditions, encore artisanale à ce stade, est la même qui structure plus tard l'analyse de sécurité d'une fonction : une vulnérabilité est très souvent la conséquence d'une précondition supposée par le développeur mais jamais vérifiée dans le code (par exemple, supposer qu'une chaîne de caractères fournie par l'utilisateur ne dépassera jamais une certaine taille).

## 2. Pourquoi le langage C ?

Le C a été conçu par Dennis Ritchie chez Bell Labs au début des années 1970, en s'appuyant sur le langage B de Ken Thompson, lui-même dérivé du BCPL. Il a servi dès l'origine à réécrire le système d'exploitation Unix, ce qui explique sa proximité durable avec les systèmes d'exploitation et les architectures matérielles. Le langage a ensuite été normalisé par l'ANSI puis l'ISO, avec plusieurs révisions du standard : C89/C90, C99, C11, C17, et plus récemment C23. Dans ce cours, on s'appuie sur un sous-ensemble stable et largement supporté du langage (proche de C11), compatible avec la plupart des compilateurs modernes (GCC, Clang).

Le C reste incontournable, en particulier en sécurité informatique, pour plusieurs raisons :

- il offre un accès direct à la mémoire (arithmétique de pointeurs, allocation manuelle), ce qui en fait le meilleur terrain pédagogique pour comprendre en profondeur les vulnérabilités mémoire (*buffer overflow*, *use-after-free*, *double free*, etc.) plutôt que de les subir comme des boîtes noires ;
- une grande partie des systèmes d'exploitation (noyaux Linux, Windows, BSD), des interpréteurs d'autres langages, des bases de données et des outils bas niveau (compilateurs, pilotes, systèmes embarqués) sont écrits en C ou s'appuient dessus ;
- comprendre le C facilite la lecture de code décompilé ou désassemblé lors d'une analyse de vulnérabilités, d'une rétro-ingénierie de logiciel malveillant, ou d'une cryptanalyse appliquée ;
- de nombreuses bibliothèques cryptographiques de référence (OpenSSL, libsodium, etc.) sont écrites en C, précisément pour des raisons de performance et de contrôle fin de la mémoire.

### Ce que le C ne fait pas à votre place

Contrairement à des langages comme Python, Java ou Rust, le C ne comporte ni ramasse-miettes (*garbage collector*), ni vérification automatique des bornes de tableaux, ni système de types empêchant les conversions dangereuses. Cette absence de filet de sécurité est délibérée : elle donne un contrôle total sur la mémoire et les performances, au prix d'une responsabilité entière laissée au programmeur. C'est précisément cette caractéristique qui fait du C un langage à la fois puissant et propice aux vulnérabilités s'il est mal maîtrisé — d'où l'importance, dans ce cours, d'insister systématiquement sur les pièges et les bonnes pratiques associés à chaque construction du langage.

## 3. De l'algorithme au programme exécutable

La chaîne de compilation C comporte plusieurs étapes distinctes, généralement invisibles pour le débutant car enchaînées automatiquement par le compilateur :

1. **Préprocesseur** : traite les directives commençant par `#` (`#include`, `#define`, `#ifdef`, …) *avant* toute analyse du langage C proprement dit. Il s'agit d'une simple substitution textuelle : `#include` copie littéralement le contenu d'un fichier d'en-tête, `#define` remplace un identifiant par un texte.
2. **Compilation** : traduit le code C (après préprocession) en assembleur, puis génère un fichier objet (`.o`) contenant du code machine et des symboles non encore résolus.
3. **Édition de liens (linking)** : assemble un ou plusieurs fichiers objets avec les bibliothèques nécessaires (statiques `.a` ou dynamiques `.so`/`.dll`) pour produire un exécutable final, en résolvant les références entre fonctions/variables définies dans des fichiers différents.

```bash
gcc -Wall -Wextra -std=c11 -o programme programme.c
./programme
echo $?          # affiche le code de retour du programme (0 = succès)
```

### Les avertissements du compilateur comme première ligne de défense

Les options `-Wall` et `-Wextra` activent un grand nombre d'avertissements (variable non utilisée, comparaison suspecte, type incompatible, etc.). Il est vivement recommandé d'aller plus loin en développement :

```bash
gcc -Wall -Wextra -Werror -std=c11 -fsanitize=address,undefined -g -o programme programme.c
```

- `-Werror` transforme chaque avertissement en erreur bloquante : on ne peut pas « oublier » un avertissement.
- `-fsanitize=address` (AddressSanitizer) instrumente le programme pour détecter à l'exécution les débordements de tampon, les *use-after-free*, etc.
- `-fsanitize=undefined` détecte les comportements indéfinis (division par zéro, débordement d'entier signé, déréférencement de pointeur invalide…).
- `-g` conserve les informations de débogage, utiles avec un débogueur comme `gdb`.

Cette discipline — compiler avec un maximum d'avertissements et d'instrumentation dès le développement — est directement transposable au monde professionnel : la majorité des vulnérabilités mémoire en C auraient pu être détectées bien avant la mise en production, simplement en activant ces options. Des outils d'analyse statique complémentaires (`cppcheck`, `clang-tidy`, `clang --analyze`) et dynamiques (`valgrind`) seront réutilisés au fil des travaux pratiques.

## 4. Premier programme

```c
#include <stdio.h>

int main(void) {
    printf("Bonjour, monde !\n");
    return 0;
}
```

- `#include <stdio.h>` : inclut la bibliothèque d'entrées-sorties standard, nécessaire pour utiliser `printf`.
- `int main(void)` : point d'entrée du programme. Le type `int` en retour signifie que la fonction renvoie un entier au système d'exploitation ; `void` entre parenthèses indique explicitement qu'elle ne prend aucun paramètre (une signature `main()` sans `void` est tolérée mais moins précise).
- `printf` : affiche du texte formaté sur la sortie standard ; `\n` est un caractère d'échappement représentant un saut de ligne.
- `return 0;` : termine `main` en renvoyant le code 0, convention universelle signifiant « succès ». Toute autre valeur (souvent entre 1 et 255) signale une erreur au processus appelant (par exemple un script shell qui teste `$?`).

### Un deuxième exemple, avec variables et calcul

```c
#include <stdio.h>

int main(void) {
    int cote = 5;
    int perimetre = 4 * cote;

    printf("Le perimetre d'un carre de cote %d est %d.\n", cote, perimetre);
    return 0;
}
```

Ce court exemple illustre déjà l'essentiel du flux d'un programme C : déclarer des données, effectuer un calcul, produire un résultat observable. Les chapitres suivants détaillent chacun de ces aspects (types, structures de contrôle, fonctions).

## 5. Structure générale d'un programme C

```
directives de préprocesseur       (#include, #define, ...)
déclarations globales              (constantes, prototypes de fonctions)
int main(void) {
    déclarations locales
    instructions
    return code;
}
définitions des autres fonctions
```

Un fichier source C respecte généralement cet ordre, même si le langage autorise une certaine souplesse. Le prototype d'une fonction (sa signature, sans son corps) doit être connu du compilateur *avant* tout appel à cette fonction ; c'est pourquoi les prototypes sont placés en haut du fichier, ou dans un fichier d'en-tête (`.h`) inclus via `#include`.

### Conventions de nommage et lisibilité

Le C n'impose presque aucune convention de nommage, mais en adopter une de façon cohérente améliore fortement la lisibilité et la maintenabilité du code — deux qualités indissociables d'un code sûr, car un code difficile à lire est un code difficile à auditer :

- noms de variables et de fonctions en minuscules, mots séparés par `_` (`snake_case`), par exemple `calculer_moyenne` ;
- noms de constantes et de macros en majuscules, par exemple `TAILLE_MAX` ;
- noms explicites plutôt que des abréviations cryptiques (`compteur_erreurs` plutôt que `ce`).

### Les commentaires

```c
// Commentaire sur une seule ligne

/* Commentaire
   sur plusieurs lignes */
```

Un commentaire ne doit pas répéter ce que le code dit déjà, mais expliquer *pourquoi* un choix a été fait, ou documenter une hypothèse importante (par exemple une précondition non vérifiée par le code lui-même). En sécurité, un commentaire signalant explicitement « cette fonction suppose que `buffer` fait au moins `TAILLE_MAX` octets » a une réelle valeur pour un futur relecteur ou auditeur du code.

## À retenir

- Un algorithme se conçoit avant de se coder, et se raisonne en termes d'entrées, de sortie, de préconditions et de postconditions.
- Le C est un langage compilé, statiquement typé, proche de la machine, normalisé par l'ISO (C89 à C23).
- Le C ne protège pas automatiquement le programmeur : pas de vérification de bornes, pas de ramasse-miettes ; la rigueur doit donc venir du développeur et de ses outils.
- La chaîne de compilation comprend préprocesseur, compilation et édition de liens ; comprendre ces étapes aide à diagnostiquer les erreurs.
- Toujours compiler avec les avertissements activés (`-Wall -Wextra`, voire `-Werror` et les sanitizers) : la majorité des bugs de sécurité en C sont détectables dès la compilation ou par analyse statique/dynamique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
