# Chapitre 2 — Variables, types, opérateurs et entrées-sorties

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/02-variables-types-operateurs-es.txt){ target=_blank }

## 1. Variables et types de base

Une variable est un emplacement mémoire nommé, dont le **type** détermine trois choses : la taille occupée en mémoire (en octets), l'interprétation binaire de son contenu (entier signé, flottant, caractère…), et l'ensemble des opérations autorisées sur elle. Déclarer une variable, c'est donc réserver un espace mémoire et indiquer au compilateur comment ce contenu doit être manipulé.

| Type | Taille usuelle | Exemple | Rôle |
|---|---|---|---|
| `int` | 4 octets | `int age = 25;` | entier signé |
| `unsigned int` | 4 octets | `unsigned int compteur = 0;` | entier non signé |
| `char` | 1 octet | `char lettre = 'A';` | caractère / petit entier |
| `float` | 4 octets | `float prix = 19.99f;` | flottant simple précision |
| `double` | 8 octets | `double pi = 3.14159265;` | flottant double précision |
| `long`, `short` | variable | `long total;` | variantes de taille de `int` |
| `_Bool` (`bool` via `stdbool.h`) | 1 octet | `bool actif = true;` | valeur booléenne (0 ou 1) |

La taille exacte de chaque type dépend de l'architecture et du compilateur : le standard C ne garantit que des tailles *minimales* relatives (par exemple `sizeof(short) <= sizeof(int) <= sizeof(long)`), pas des tailles absolues. On utilise l'opérateur `sizeof(type)` pour la vérifier concrètement :

```c
#include <stdio.h>

int main(void) {
    printf("sizeof(int)    = %zu\n", sizeof(int));
    printf("sizeof(long)   = %zu\n", sizeof(long));
    printf("sizeof(double) = %zu\n", sizeof(double));
    return 0;
}
```

Pour des programmes dont le comportement doit être **prévisible et portable** — ce qui est un impératif en sécurité, où une taille de type ambiguë peut ouvrir une faille sur une architecture donnée et pas sur une autre — on préfère les types à taille garantie de `<stdint.h>` : `int8_t`, `uint8_t`, `int16_t`, `uint32_t`, `int64_t`, etc. Le suffixe indique le nombre de bits, et le préfixe `u` signale un type non signé.

```c
#include <stdint.h>

uint8_t  octet   = 255;      // 0 à 255
int32_t  identifiant = -42;  // -2 147 483 648 à 2 147 483 647
uint32_t taille  = 4096;     // toujours 32 bits, quelle que soit l'architecture
```

### Limites des types numériques

Les en-têtes `<limits.h>` (entiers) et `<float.h>` (flottants) définissent les bornes de chaque type, par exemple `INT_MAX`, `INT_MIN`, `UINT_MAX`. Connaître ces bornes est indispensable pour raisonner sur les **débordements d'entier** (*integer overflow*), abordés en détail plus bas.

## 2. Déclaration, initialisation, portée

```c
int x;                // déclarée, valeur indéterminée (danger !)
int y = 0;             // déclarée et initialisée
const int MAX = 100;   // constante, non modifiable après initialisation
```

Une variable non initialisée contient une valeur indéterminée (souvent appelée « garbage » : ce qui se trouvait précédemment à cet emplacement mémoire). L'utiliser avant affectation est un comportement indéfini en C : le programme peut produire un résultat incohérent, planter, ou — plus insidieux — fonctionner « par chance » sur la machine de développement puis échouer en production. C'est également une source réelle de vulnérabilité : lire une zone mémoire non initialisée peut exposer des données résiduelles d'un traitement précédent (mot de passe, clé, fragment de fichier), une classe de vulnérabilité connue sous le nom de fuite d'information par mémoire non initialisée.

Le mot-clé `const` ne rend pas une variable « constante » au sens mathématique strict (elle occupe toujours un emplacement mémoire), mais interdit au compilateur d'accepter toute affectation ultérieure — une protection précieuse contre les modifications accidentelles.

```c
const int LIMITE = 10;
LIMITE = 20; // erreur de compilation : assignment of read-only variable
```

### Portée en bref

La portée d'une variable (l'endroit du programme où son nom est visible) est traitée en détail au chapitre 4 ; retenons pour l'instant qu'une variable déclarée à l'intérieur d'un bloc `{ ... }` (par exemple le corps de `main`, ou celui d'une boucle) n'existe que dans ce bloc.

## 3. Opérateurs

- **Arithmétiques** : `+ - * /  %` (le `%`, modulo, n'existe que pour les types entiers).
- **Relationnels** : `== != < > <= >=` (le résultat est un entier valant 0 — faux — ou 1 — vrai).
- **Logiques** : `&& || !` (évaluation en court-circuit : dans `a && b`, si `a` est faux, `b` n'est pas évalué).
- **Bit à bit** : `& | ^ ~ << >>` — fondamentaux en cryptographie (XOR, masques, décalages) et en manipulation bas niveau de protocoles réseau ; ils seront réutilisés en profondeur au chapitre Cryptographie.
- **Affectation composée** : `+= -= *= /= %= &= |= ^= <<= >>=`.
- **Incrémentation/décrémentation** : `++ --`, en forme préfixe (`++i`, incrémente puis évalue) ou postfixe (`i++`, évalue puis incrémente).

```c
int a = 5, b = 3;
int somme = a + b;      // 8
int reste = a % b;      // 2
int masque = a & b;     // ET bit à bit -> 1
int i = 0;
int avant = i++;        // avant = 0, i vaut 1 ensuite
int apres = ++i;        // i vaut 2, apres = 2
```

### Évaluation en court-circuit : un outil de sécurité défensive

L'évaluation en court-circuit de `&&` et `||` est souvent exploitée volontairement pour écrire du code défensif, en s'assurant qu'une condition dangereuse n'est testée que si une condition de garde préalable est vraie :

```c
char *chaine = NULL;
if (chaine != NULL && chaine[0] == 'A') {
    // chaine[0] n'est évalué que si chaine != NULL est vrai :
    // aucun déréférencement de pointeur nul possible ici.
}
```

Inverser l'ordre des deux conditions (`chaine[0] == 'A' && chaine != NULL`) provoquerait un déréférencement de pointeur nul si `chaine` vaut `NULL` — un piège classique pour un débutant.

### Débordement d'entier (*integer overflow*)

Un type entier ne peut représenter qu'un intervalle fini de valeurs. Dépasser cet intervalle produit un débordement :

```c
#include <limits.h>
#include <stdio.h>

int main(void) {
    int x = INT_MAX;
    printf("%d\n", x + 1); // comportement indéfini pour un int signé !
    return 0;
}
```

Pour les entiers **signés**, un débordement est un comportement indéfini par la norme C (le compilateur est libre d'optimiser en supposant qu'il ne se produit jamais). Pour les entiers **non signés**, le débordement est défini et « boucle » selon une arithmétique modulo 2ⁿ (`UINT_MAX + 1` vaut `0`). Cette différence de comportement est une source fréquente de vulnérabilités : un calcul de taille de tampon effectué en entier non signé peut « boucler » vers une petite valeur après une soustraction inattendue, provoquant ensuite une allocation trop petite et un débordement d'écriture.

## 4. Conversions de type (casting)

```c
int i = 7;
double d = (double) i / 2;   // conversion explicite : 3.5
int tronque = (int) 3.9;     // conversion explicite : 3 (troncature, pas arrondi)
```

On distingue :

- la **conversion explicite** (*cast*), écrite `(type) expression`, décidée volontairement par le programmeur ;
- la **conversion implicite**, effectuée silencieusement par le compilateur, par exemple lors du passage d'un `int` à un `double` dans une expression mixte, ou d'un `int` à un `char` lors d'une affectation.

```c
char c = 300;   // conversion implicite : dépasse la plage d'un char, résultat dépendant de l'implémentation
int i = 3.99;   // conversion implicite : i vaut 3 (troncature)
```

Les conversions implicites peuvent provoquer des pertes de précision ou des dépassements silencieux, sans le moindre avertissement à l'exécution : c'est une source fréquente de bugs de sécurité, par exemple la troncature d'une taille de buffer calculée en `size_t` (non signé, souvent 64 bits) vers un `int` (signé, 32 bits), qui peut transformer une taille légitime en valeur négative ou aberrante une fois réinterprétée.

### Bonne pratique : rendre les conversions explicites et vérifiées

```c
#include <stdio.h>

int main(void) {
    long valeur = 5000000000L; // dépasse la plage d'un int sur la plupart des systèmes
    if (valeur > 2147483647L || valeur < -2147483648L) {
        fprintf(stderr, "Valeur hors plage pour un int\n");
    } else {
        int v = (int) valeur;
        printf("%d\n", v);
    }
    return 0;
}
```

Documenter et vérifier explicitement une conversion potentiellement destructrice est une habitude directement transposable à l'audit de code sécurisé : un relecteur doit pouvoir repérer d'un coup d'œil les points où une donnée change de représentation.

## 5. Entrées-sorties standard

```c
#include <stdio.h>

int main(void) {
    int age;
    printf("Entrez votre âge : ");
    scanf("%d", &age);          // & : adresse de la variable
    printf("Vous avez %d ans.\n", age);
    return 0;
}
```

Spécificateurs de format courants : `%d` (int), `%u` (unsigned int), `%f` (float/double en écriture avec `printf`, `%lf` en lecture avec `scanf`), `%c` (char), `%s` (chaîne), `%p` (pointeur), `%x`/`%X` (hexadécimal), `%zu` (`size_t`, le type renvoyé par `sizeof`).

### Vérifier la valeur de retour de `scanf`

`scanf` renvoie le nombre de conversions réussies. Une saisie inattendue (l'utilisateur tape des lettres alors qu'un entier est attendu) fait échouer la conversion sans que la variable de destination soit modifiée : ignorer la valeur de retour, c'est risquer d'utiliser ensuite une variable non initialisée.

```c
int age;
if (scanf("%d", &age) != 1) {
    fprintf(stderr, "Saisie invalide\n");
    return 1;
}
```

### Point de vigilance sécurité : chaînes de format et tampons

`scanf("%s", ...)` sans limite de taille, ou une chaîne de format contrôlée par l'utilisateur passée directement à `printf` (par exemple `printf(entree_utilisateur);` au lieu de `printf("%s", entree_utilisateur);`), sont des vulnérabilités classiques : respectivement un dépassement de tampon (l'utilisateur peut saisir un texte plus long que le buffer alloué) et une vulnérabilité de type *format string*, qui peut permettre de lire, voire d'écrire, des zones arbitraires de la mémoire du processus. La règle à retenir dès maintenant : **une chaîne de format doit toujours être un littéral écrit par le développeur, jamais une donnée fournie par l'utilisateur**. Ce point sera repris et exploité en détail dans le cours *Sécurité des applications*.

```c
char nom[20];
printf("%s\n", nom);           // correct : format fixe, donnée en argument
printf(nom);                   // dangereux : nom devient la chaîne de format elle-même
```

## À retenir

- Toujours initialiser ses variables ; une valeur « indéterminée » n'est jamais un état sûr.
- Connaître la taille et les limites de chaque type (`sizeof`, `limits.h`, `stdint.h`), en particulier la distinction entre débordement signé (indéfini) et non signé (défini, modulo).
- Distinguer conversions explicites (voulues, documentées) et implicites (silencieuses, à surveiller), notamment entre types signés et non signés.
- Vérifier systématiquement la valeur de retour de `scanf`.
- Se méfier des chaînes de format non maîtrisées : jamais de donnée utilisateur directement comme premier argument de `printf`/`scanf`.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
