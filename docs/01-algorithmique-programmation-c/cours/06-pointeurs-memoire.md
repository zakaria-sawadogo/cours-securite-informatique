# Chapitre 6 — Pointeurs et gestion mémoire

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/06-pointeurs-memoire.txt){ target=_blank }

## 1. Qu'est-ce qu'un pointeur ?

Un pointeur est une variable dont la valeur est une adresse mémoire — c'est-à-dire l'emplacement où une autre donnée est stockée, plutôt que la donnée elle-même. Un pointeur est toujours typé : un `int *` pointe vers un emplacement censé contenir un `int`, ce qui permet au compilateur de savoir combien d'octets lire ou écrire, et comment interpréter ces octets.

```c
int x = 10;
int *p = &x;      // p contient l'adresse de x
printf("%d\n", *p); // déréférencement : affiche 10
*p = 20;            // modifie x via le pointeur : x vaut 20 après cette ligne
```

- `&x` : opérateur « adresse de », renvoie l'adresse mémoire où est stockée `x`.
- `*p` : opérateur de déréférencement (« valeur pointée par »), lit ou écrit la donnée à l'adresse contenue dans `p`.

Cette double nature de `*` (déclaration d'un pointeur vs déréférencement dans une expression) est une source fréquente de confusion pour les débutants : `int *p;` déclare `p` comme un pointeur vers un `int`, tandis que `*p = 20;` déréférence `p` pour écrire à l'adresse qu'il contient.

### Le pointeur nul

```c
int *p = NULL; // p ne pointe vers rien de valide
if (p != NULL) {
    printf("%d\n", *p);
} else {
    printf("p ne pointe vers rien\n");
}
```

`NULL` est une valeur conventionnelle signifiant « ce pointeur ne référence aucune adresse valide ». Déréférencer un pointeur nul (`*p` alors que `p == NULL`) est un comportement indéfini qui provoque, en pratique, l'arrêt brutal du programme sur la plupart des systèmes (violation d'accès mémoire). Vérifier systématiquement qu'un pointeur n'est pas nul avant de le déréférencer, en particulier lorsqu'il provient du retour d'une fonction susceptible d'échouer (`malloc`, `fopen`, une fonction de recherche qui peut ne rien trouver), est une habitude essentielle.

## 2. Les quatre zones de la mémoire d'un programme

| Zone | Contenu | Durée de vie |
|---|---|---|
| Code (text) | instructions du programme, généralement en lecture seule | statique, toute l'exécution |
| Données statiques | variables globales et `static`, initialisées ou non | toute l'exécution |
| Tas (heap) | allocation dynamique (`malloc`, `calloc`, `realloc`) | jusqu'à `free` explicite |
| Pile (stack) | variables locales, paramètres, adresses de retour des appels de fonction | durée du bloc / de l'appel |

Comprendre cette organisation est indispensable pour analyser un débordement de pile, un *use-after-free*, ou une fuite mémoire, car chaque classe de vulnérabilité correspond à une manipulation incorrecte d'une de ces zones :

- un débordement de tampon sur un tableau local déborde dans la **pile**, où peuvent se trouver d'autres variables locales et, plus loin, l'adresse de retour de la fonction ;
- un *use-after-free* ou une fuite mémoire concerne le **tas**, dont la gestion est entièrement manuelle en C ;
- une écriture dans une chaîne littérale (mémoire statique en lecture seule) provoque une violation d'accès.

La pile croît et décroît automatiquement au rythme des appels et retours de fonction (chapitre 4) ; le tas, en revanche, ne connaît aucune gestion automatique en C standard : chaque allocation doit être explicitement libérée par le programmeur.

## 3. Allocation dynamique

```c
#include <stdlib.h>

int *tableau = malloc(10 * sizeof(int));
if (tableau == NULL) {
    // échec d'allocation : toujours vérifier, ne jamais supposer que malloc réussit
    exit(EXIT_FAILURE);
}
for (int i = 0; i < 10; i++) tableau[i] = i * i;

free(tableau);
tableau = NULL;  // évite un « dangling pointer » (pointeur pendant)
```

- `malloc(taille)` : alloue `taille` octets sur le tas, dont le contenu initial n'est **pas** défini (contrairement à `calloc`) — l'utiliser sans initialiser explicitement les données revient au même risque qu'une variable locale non initialisée.
- `calloc(n, taille)` : alloue `n * taille` octets et les initialise à zéro ; calcule également en interne le produit `n * taille` en se protégeant d'un éventuel débordement d'entier, ce que ne fait pas un appel manuel `malloc(n * taille)`.
- `realloc(ptr, nouvelle_taille)` : redimensionne un bloc existant, en peut le déplacer ; renvoie un nouveau pointeur qu'il faut impérativement récupérer (voir piège ci-dessous).
- `free(ptr)` : libère la mémoire précédemment allouée ; **ne jamais l'appeler deux fois sur le même pointeur** (*double free*) ni utiliser le pointeur après (*use-after-free*).

### Piège classique avec `realloc`

```c
int *tableau = malloc(10 * sizeof(int));
// ...
int *tmp = realloc(tableau, 20 * sizeof(int));
if (tmp == NULL) {
    // échec : tableau reste valide et doit toujours être libéré
    free(tableau);
    exit(EXIT_FAILURE);
}
tableau = tmp; // ne réaffecter tableau qu'après avoir vérifié le succès
```

Écrire directement `tableau = realloc(tableau, ...)` est dangereux : si `realloc` échoue, elle renvoie `NULL` **sans libérer le bloc original**, et cette affectation directe écrase alors l'unique pointeur vers ce bloc, provoquant une fuite mémoire définitive (le bloc original devient inaccessible, donc impossible à libérer).

## 4. Classes de vulnérabilités mémoire liées aux pointeurs

| Vulnérabilité | Cause |
|---|---|
| Déréférencement de pointeur nul | oubli de vérifier le retour de `malloc`/`fopen`, ou variable pointeur non initialisée |
| Fuite mémoire (*memory leak*) | `malloc` (ou équivalent) sans `free` correspondant, ou pointeur écrasé avant libération |
| Use-after-free | utilisation (lecture, écriture, ou nouveau `free`) d'un pointeur après que la mémoire pointée a été libérée |
| Double free | `free` appelé deux fois sur le même pointeur, corrompant les structures internes de l'allocateur |
| Dangling pointer (pointeur pendant) | pointeur conservé vers une variable locale dont la portée est terminée, ou vers une zone déjà libérée |
| Buffer overflow | écriture (ou lecture) au-delà de la mémoire réellement allouée, via arithmétique de pointeur ou indexation de tableau |

```c
int *pointeur_pendant(void) {
    int local = 42;
    return &local; // ATTENTION : adresse d'une variable locale qui va être détruite au retour
}

int main(void) {
    int *p = pointeur_pendant();
    printf("%d\n", *p); // comportement indéfini : local n'existe plus sur la pile
    return 0;
}
```

Dans cet exemple, `local` est une variable de la pile : son emplacement mémoire redevient disponible dès le retour de la fonction `pointeur_pendant`, alors que l'appelant continue de détenir son adresse. Le comportement observé peut sembler correct « par chance » (la valeur 42 peut encore s'y trouver, si rien ne l'a écrasée entre-temps), ce qui rend ce type de bug particulièrement traître : il peut passer inaperçu en test et se manifester de façon imprévisible en production, y compris de façon exploitable si un attaquant parvient à contrôler ce qui réutilise cette zone mémoire.

Ces classes de vulnérabilités constituent le socle du cours *Sécurité des applications* (exploitation) et sont directement liées aux outils d'audit statique/dynamique vus en *Audit organisation et technique* : des outils comme AddressSanitizer (`-fsanitize=address`, voir chapitre 1) ou Valgrind sont conçus précisément pour détecter ces erreurs à l'exécution, bien avant qu'elles ne soient exploitées.

## 5. Arithmétique des pointeurs

```c
int tab[5] = {10, 20, 30, 40, 50};
int *p = tab;      // p pointe sur tab[0] (un tableau "dégrade" en pointeur vers son premier élément)
printf("%d\n", *(p + 2)); // 30, équivalent à tab[2]
p++;                // p pointe maintenant sur tab[1]
printf("%d\n", *p); // 20
```

L'arithmétique de pointeur se fait en unités de la **taille du type pointé**, pas en octets bruts : `p + 1` avance l'adresse de `sizeof(*p)` octets (4 octets pour un `int` classique), pas d'un seul octet. C'est ce mécanisme qui rend l'équivalence `tab[i]` ↔ `*(tab + i)` cohérente quel que soit le type des éléments.

### Différence entre pointeur et tableau

Bien que `tab` et `p` puissent être utilisés de façon très similaire (`tab[i]`, `p[i]`, `*(tab + i)`, `*(p + i)` sont toutes des écritures valides), `tab` et `p` ne sont pas identiques : `tab` est un tableau, dont l'adresse est fixe et connue à la compilation ; `p` est une variable pointeur, qui peut être réaffectée pour pointer ailleurs. `sizeof(tab)` donne la taille totale du tableau, alors que `sizeof(p)` donne la taille d'un pointeur (typiquement 8 octets sur une architecture 64 bits), quel que soit le type pointé.

## 6. Pointeurs de fonction (aperçu)

```c
int addition(int a, int b) { return a + b; }
int soustraction(int a, int b) { return a - b; }

int (*operation)(int, int) = addition;
printf("%d\n", operation(2, 3)); // 5

operation = soustraction;
printf("%d\n", operation(5, 2)); // 3
```

Un pointeur de fonction stocke l'adresse d'une fonction, que l'on peut ensuite appeler indirectement, ou transmettre en paramètre à une autre fonction. Ce mécanisme est utilisé pour construire des *callbacks* (fonctions passées en paramètre pour être appelées plus tard, par exemple par une bibliothèque de tri générique) et des tables de dispatch (tableaux de pointeurs de fonction, indexés par exemple par un code d'opération).

Ce même mécanisme, extrêmement utile pour concevoir du code flexible, est aussi exploité par certaines techniques d'attaque de type détournement de flot de contrôle : si un attaquant parvient, via un débordement de tampon, à écraser un pointeur de fonction (ou une adresse de retour sur la pile) avec une valeur de son choix, il peut potentiellement rediriger l'exécution du programme vers un code arbitraire. C'est l'une des raisons pour lesquelles la sécurisation de la pile (protections *stack canaries*, ASLR, etc., abordées dans les cours de sécurité applicative) est aujourd'hui systématique dans les compilateurs modernes.

## À retenir

- Toujours vérifier le retour de `malloc`/`calloc`/`realloc` avant d'utiliser le pointeur obtenu.
- Toute allocation dynamique doit avoir une libération correspondante, une seule fois — ni oubliée (fuite mémoire), ni dupliquée (double free).
- Mettre un pointeur à `NULL` après `free` limite fortement les risques de use-after-free accidentel, car déréférencer `NULL` échoue immédiatement et de façon détectable, plutôt que de corrompre silencieusement la mémoire.
- L'arithmétique de pointeur se fait en unités du type pointé ; un tableau « dégrade » en pointeur vers son premier élément dès qu'il est manipulé comme tel.
- Ne jamais renvoyer l'adresse d'une variable locale : sa durée de vie se termine avec le bloc qui la contient.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
