# Chapitre 4 — Fonctions et portée des variables

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/04-fonctions-portee.txt){ target=_blank }

## 1. Pourquoi découper en fonctions ?

Une fonction encapsule un traitement réutilisable, sous un nom qui décrit son intention. Ce découpage apporte plusieurs bénéfices concrets :

- **Lisibilité** : un programme découpé en fonctions bien nommées se lit presque comme une table des matières (`lire_configuration()`, `valider_entree()`, `calculer_resultat()`).
- **Réutilisabilité** : une fonction correctement écrite peut être appelée depuis plusieurs endroits du programme, sans dupliquer le code.
- **Testabilité** : une fonction avec des entrées et une sortie clairement définies peut être testée isolément, indépendamment du reste du programme.
- **Limitation de la portée des erreurs** : un bug dans une fonction bien délimitée est plus facile à localiser qu'un bug perdu dans une longue suite d'instructions au sein d'un unique `main`.

C'est un principe de base du *Security by design* : plus une fonction est petite et ciblée (elle ne fait « qu'une seule chose »), plus elle est facile à auditer, à tester exhaustivement, et à faire évoluer sans introduire de régression. Un code composé de fonctions de plusieurs centaines de lignes, aux responsabilités mélangées, est statistiquement bien plus difficile à sécuriser qu'un code découpé en unités courtes et cohérentes.

## 2. Déclaration, prototype, définition

```c
// Prototype (souvent dans un .h)
int addition(int a, int b);

int main(void) {
    int resultat = addition(3, 4);
    printf("%d\n", resultat);
    return 0;
}

// Définition
int addition(int a, int b) {
    return a + b;
}
```

Le **prototype** (ou déclaration) annonce au compilateur la signature d'une fonction — son nom, le type de sa valeur de retour, et le nombre et le type de ses paramètres — sans fournir son corps. Il permet d'appeler une fonction avant sa définition complète dans le fichier, et surtout de vérifier la cohérence des types passés lors de chaque appel : le compilateur détecte alors des erreurs (mauvais nombre d'arguments, type incompatible) qu'une absence de prototype laisserait passer silencieusement, avec un comportement indéfini à l'exécution.

Dans un programme composé de plusieurs fichiers source, les prototypes des fonctions destinées à être partagées sont regroupés dans un fichier d'en-tête (`.h`), inclus par chaque fichier `.c` qui en a besoin :

```c
// operations.h
#ifndef OPERATIONS_H
#define OPERATIONS_H

int addition(int a, int b);
int soustraction(int a, int b);

#endif
```

Les directives `#ifndef` / `#define` / `#endif` forment un **garde d'inclusion** (*include guard*) : elles empêchent le contenu du fichier d'en-tête d'être inséré plusieurs fois si plusieurs fichiers `.c` l'incluent, ce qui provoquerait des erreurs de redéfinition.

## 3. Passage de paramètres

- **Passage par valeur** (comportement par défaut en C, pour tous les types de base) : la fonction reçoit une **copie** de la valeur de l'argument ; toute modification à l'intérieur de la fonction est locale et n'affecte pas la variable de l'appelant.
- **Passage par adresse** (en transmettant un pointeur vers la variable) : permet à la fonction de modifier la variable de l'appelant, puisqu'elle reçoit non pas une copie de la valeur mais l'adresse mémoire où cette valeur est stockée.

```c
void incrementer_par_valeur(int x) {
    x++;               // ne modifie que la copie locale
}

void incrementer_par_adresse(int *x) {
    (*x)++;             // modifie la variable pointée
}

int main(void) {
    int n = 5;
    incrementer_par_valeur(n);
    printf("%d\n", n);     // affiche toujours 5

    incrementer_par_adresse(&n);
    printf("%d\n", n);     // affiche 6
    return 0;
}
```

Cette distinction est fondamentale et sera approfondie au chapitre 6 (pointeurs), mais elle a déjà des implications en matière de conception sécurisée : une fonction qui prend un pointeur en paramètre peut, si elle est mal écrite, modifier ou lire au-delà de la zone mémoire prévue par l'appelant. Documenter clairement, pour chaque paramètre pointeur, ce que la fonction attend en entrée (taille du tampon pointé, état attendu) et ce qu'elle garantit en sortie fait partie des bonnes pratiques attendues dans ce cours.

### Paramètres constants : documenter l'intention de ne pas modifier

```c
void afficher(const char *message) {
    printf("%s\n", message);
    // message[0] = 'X'; // interdit par le compilateur : message est const
}
```

Préfixer un paramètre pointeur de `const` lorsque la fonction ne doit pas modifier les données pointées est une convention précieuse : elle documente l'intention dans la signature elle-même, et le compilateur la fait respecter — un exemple concret de « faire porter la contrainte de sécurité par le langage plutôt que par la seule discipline du développeur ».

## 4. Portée (scope) et durée de vie

| Type de variable | Portée | Durée de vie |
|---|---|---|
| locale | bloc où elle est déclarée | jusqu'à la fin du bloc (pile) |
| globale | tout le fichier (ou tout le programme si non `static`) | toute l'exécution |
| `static` locale | bloc de déclaration | toute l'exécution, valeur conservée entre appels |
| paramètre | corps de la fonction | durée de l'appel |

La **portée** (*scope*) définit où, dans le texte du programme, le nom d'une variable est visible et utilisable. La **durée de vie** définit pendant combien de temps, à l'exécution, l'emplacement mémoire associé à la variable reste valide. Ces deux notions ne coïncident pas toujours, ce qui est justement la source de certaines erreurs classiques (voir le pointeur pendant, abordé au chapitre 6).

```c
int compteur_appels(void) {
    static int n = 0;  // initialisée une seule fois, à la première exécution
    n++;
    return n;
}

int main(void) {
    printf("%d\n", compteur_appels()); // 1
    printf("%d\n", compteur_appels()); // 2
    printf("%d\n", compteur_appels()); // 3
    return 0;
}
```

Une variable `static` locale n'est initialisée qu'une seule fois (avant le premier appel de la fonction) et conserve sa valeur d'un appel à l'autre — contrairement à une variable locale ordinaire, réinitialisée à chaque appel et détruite à la fin de chaque exécution du bloc.

### Variables globales : commodité contre risque

```c
int solde_global = 0; // variable globale : visible et modifiable depuis tout le fichier

void deposer(int montant) {
    solde_global += montant; // effet de bord sur une variable globale
}
```

Les variables globales facilitent le partage d'état entre plusieurs fonctions sans avoir à le faire transiter explicitement par des paramètres. Mais elles nuisent fortement à la prévisibilité du code : n'importe quelle fonction du fichier (voire du programme, si la variable n'est pas `static`) peut la modifier, ce qui complique considérablement l'analyse de sécurité — on parle d'**effets de bord non locaux**, difficiles à tracer lors d'un audit ou d'un débogage. La bonne pratique consiste à limiter au strict nécessaire l'usage de variables globales, et à restreindre leur portée au fichier courant avec le mot-clé `static` lorsqu'elles n'ont pas besoin d'être visibles ailleurs :

```c
static int compteur_interne = 0; // static au niveau fichier : invisible depuis les autres fichiers .c
```

Attention : le mot-clé `static` a ici un sens différent de celui vu pour une variable locale — au niveau global, il restreint la **portée** (liaison interne au fichier) plutôt que de modifier la **durée de vie** (qui est déjà celle du programme entier pour une variable globale).

## 5. Récursivité

```c
unsigned long factorielle(unsigned int n) {
    if (n == 0) return 1;   // cas de base : arrête la récursion
    return n * factorielle(n - 1);  // appel récursif
}
```

Une fonction récursive s'appelle elle-même, directement ou indirectement, jusqu'à atteindre un **cas de base** qui arrête la récursion. Toute fonction récursive doit garantir deux propriétés : l'existence d'un cas de base atteignable, et la progression vers ce cas de base à chaque appel (ici, `n` décroît strictement).

Chaque appel récursif empile un nouveau cadre d'appel (*stack frame*) sur la pile d'exécution, contenant notamment les paramètres et variables locales de cet appel. Une récursion non bornée (cas de base jamais atteint, à cause d'une erreur de logique) ou simplement trop profonde (des milliers d'appels imbriqués sur une entrée légitime mais très grande) provoque un **débordement de pile** (*stack overflow*), qui termine brutalement le programme. C'est un cas concret de vulnérabilité par épuisement de ressource : une fonction récursive qui traite une structure de données fournie par l'utilisateur (par exemple un document imbriqué) sans limiter la profondeur de récursion peut être utilisée pour provoquer un déni de service.

```c
// Version itérative équivalente, sans consommation de pile supplémentaire
unsigned long factorielle_iterative(unsigned int n) {
    unsigned long resultat = 1;
    for (unsigned int i = 2; i <= n; i++) {
        resultat *= i;
    }
    return resultat;
}
```

Lorsque la profondeur de récursion peut être contrôlée par une entrée externe, il est prudent soit de borner explicitement cette profondeur, soit de préférer une version itérative, plus sûre vis-à-vis de la consommation de pile.

## 6. Bonnes pratiques

- Une fonction = une responsabilité claire, exprimée par son nom.
- Toujours valider les paramètres reçus (pointeurs non nuls, tailles cohérentes, valeurs dans les bornes attendues) avant de les utiliser — ne jamais supposer qu'un appelant respecte automatiquement le contrat implicite d'une fonction.
- Documenter précondition et postcondition, y compris qui est responsable de libérer une mémoire éventuellement allouée par la fonction (l'appelant ou la fonction elle-même).
- Limiter le nombre de paramètres d'une fonction ; au-delà de quatre ou cinq, envisager de les regrouper dans une structure (voir chapitre 7).
- Préférer plusieurs petites fonctions bien nommées à une seule fonction longue mêlant plusieurs responsabilités.

## À retenir

- Le passage par adresse (via pointeur) est le mécanisme C permettant à une fonction de modifier les données de l'appelant ; le passage par valeur, par défaut, ne le permet jamais pour les types de base.
- La portée `static` locale crée un état persistant discret entre les appels : à utiliser avec parcimonie et à documenter clairement.
- Les variables globales facilitent le partage d'état mais introduisent des effets de bord non locaux, difficiles à auditer.
- La récursivité est élégante et naturelle pour certains problèmes, mais bornée par la taille de la pile : toujours s'assurer qu'un cas de base est atteignable et que la profondeur reste raisonnable.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
