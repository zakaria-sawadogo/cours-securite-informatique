# Chapitre 5 — Tableaux et chaînes de caractères

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/05-tableaux-chaines.txt){ target=_blank }

## 1. Tableaux statiques

```c
int notes[5] = {12, 15, 8, 17, 10};
int i;
for (i = 0; i < 5; i++) {
    printf("%d\n", notes[i]);
}
```

En C, un tableau `notes[5]` occupe un bloc **contigu** de mémoire : les cinq entiers sont stockés côte à côte, sans espace ni structure supplémentaire. C'est cette contiguïté qui rend l'accès indexé aussi rapide : `notes[i]` est strictement équivalent à `*(notes + i)`, c'est-à-dire « ajouter `i` fois la taille d'un `int` à l'adresse de début du tableau, puis lire la valeur à cette adresse ». Cette équivalence sera reprise et approfondie au chapitre 6 sur les pointeurs.

**Le C n'effectue aucune vérification de borne, ni à la compilation, ni à l'exécution.** Lire ou écrire `notes[5]` ou `notes[-1]` compile sans erreur et s'exécute, en accédant à de la mémoire qui se trouve juste après ou juste avant le tableau — mémoire qui appartient potentiellement à une autre variable, ou qui n'est même pas allouée au processus. C'est la cause première des débordements de tampon (*buffer overflow*), l'une des vulnérabilités logicielles les plus exploitées historiquement dans les systèmes écrits en C : elle sera approfondie, avec des exemples d'exploitation, dans le cours *Sécurité des applications*.

```c
int notes[5] = {12, 15, 8, 17, 10};
notes[5] = 99;  // compile ! écrit hors du tableau : comportement indéfini
printf("%d\n", notes[10]); // compile ! lit hors du tableau : valeur imprévisible
```

### Initialisation et taille implicite

```c
int a[5] = {1, 2, 3};      // les 2 derniers éléments sont initialisés à 0
int b[] = {1, 2, 3, 4};    // taille déduite automatiquement : 4
int c[5] = {0};            // tous les éléments à 0
```

Lorsqu'une initialisation partielle est fournie, les éléments non explicitement initialisés sont mis à zéro — mais uniquement dans le cas d'une initialisation *avec accolades*. Un tableau local déclaré sans initialisation du tout (`int a[5];`) contient, comme pour une simple variable, des valeurs indéterminées.

### `sizeof` et taille d'un tableau

```c
int notes[5];
printf("%zu\n", sizeof(notes));            // taille totale en octets (5 * sizeof(int))
printf("%zu\n", sizeof(notes) / sizeof(notes[0])); // nombre d'éléments : 5
```

Cette technique (`sizeof(tableau) / sizeof(tableau[0])`) ne fonctionne que tant que `notes` désigne réellement le tableau complet, dans la portée où il a été déclaré. Dès qu'un tableau est transmis à une fonction (voir plus bas), cette astuce cesse de fonctionner : c'est l'un des pièges les plus fréquents pour un programmeur C débutant.

## 2. Tableaux à deux dimensions

```c
int matrice[3][4];
matrice[1][2] = 7;
```

Un tableau à deux dimensions est stocké en mémoire en *row-major order* (ordre ligne par ligne) : les éléments d'une même ligne sont contigus, et la ligne suivante commence immédiatement après la fin de la précédente. Concrètement, `matrice[3][4]` occupe le même espace mémoire contigu qu'un tableau à une dimension de 12 éléments, et `matrice[i][j]` est équivalent à un accès à l'indice `i * 4 + j` dans ce tableau linéaire.

```c
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 4; j++) {
        matrice[i][j] = i * 4 + j;
    }
}
```

## 3. Tableaux passés en paramètre de fonction

Lorsqu'un tableau est passé en argument à une fonction, il est en réalité converti (« dégradé ») en un simple pointeur vers son premier élément : la fonction ne reçoit **aucune information sur sa taille**.

```c
void afficher_tableau(int t[], int taille) {
    // ici, sizeof(t) donnerait la taille d'un pointeur (souvent 8), pas celle du tableau !
    for (int i = 0; i < taille; i++) {
        printf("%d ", t[i]);
    }
    printf("\n");
}

int main(void) {
    int notes[5] = {12, 15, 8, 17, 10};
    afficher_tableau(notes, 5); // la taille doit être transmise explicitement
    return 0;
}
```

Cette caractéristique du langage — un tableau C ne connaît pas sa propre taille à l'exécution, et cette information se « perd » dès qu'il est transmis à une fonction — oblige à transmettre systématiquement la taille en paramètre séparé. Oublier de le faire, ou transmettre une taille incorrecte, est une cause directe de débordement de tampon dans des fonctions qui manipulent des tableaux « à l'aveugle ».

## 4. Chaînes de caractères

En C, une chaîne est un tableau de `char` terminé par l'octet nul `'\0'` (valeur 0), qui marque la fin logique de la chaîne indépendamment de la taille physique du tableau qui la contient.

```c
char nom[20] = "Zakaria";
printf("%s\n", nom);
printf("Longueur : %zu\n", strlen(nom)); // 7, sans compter le '\0' final
```

Le tableau `nom` occupe 20 octets, mais la chaîne qu'il contient ne fait que 7 caractères utiles + 1 octet nul ; les 12 octets restants sont inutilisés (ou contiennent des valeurs résiduelles, selon le mode d'initialisation). C'est précisément cette convention — la longueur d'une chaîne est déterminée par la position du `'\0'`, pas par la taille du tableau — qui rend les fonctions de manipulation de chaînes potentiellement dangereuses : rien n'empêche une fonction d'écrire au-delà des limites physiques du tableau en cherchant, ou en plaçant, cet octet nul.

### Fonctions de `string.h`

| Fonction | Rôle | Risque |
|---|---|---|
| `strlen` | longueur d'une chaîne | boucle indéfiniment (et lit hors mémoire) si le `'\0'` est absent |
| `strcpy(dst, src)` | copie | **aucune vérification de taille** → débordement si `src` est plus long que `dst` |
| `strncpy(dst, src, n)` | copie bornée à `n` octets | ne garantit pas la terminaison par `'\0'` si `src` fait `n` caractères ou plus |
| `strcat(dst, src)` | concaténation | même risque que `strcpy` : aucune vérification de la taille disponible dans `dst` |
| `strncat(dst, src, n)` | concaténation bornée | plus sûre, mais `n` doit être calculé avec soin (espace réellement disponible) |
| `strcmp(a, b)` | comparaison lexicographique | — (renvoie 0 si égales, une valeur non nulle sinon) |
| `snprintf` | formatage borné vers une chaîne | alternative sûre recommandée, y compris pour construire une chaîne à partir de plusieurs valeurs |

```c
char buffer[10];
strcpy(buffer, "Ceci dépasse dix caractères"); // comportement indéfini : écrase la mémoire adjacente
```

Dans cet exemple, la chaîne source fait bien plus de 10 caractères : `strcpy` continue d'écrire au-delà des limites de `buffer`, corrompant potentiellement d'autres variables locales, l'adresse de retour de la fonction sur la pile, ou toute autre donnée voisine en mémoire. C'est l'archétype du débordement de tampon exploitable.

### `strncpy` : un piège fréquent malgré son nom rassurant

```c
char destination[10];
strncpy(destination, "Un texte assez long", sizeof(destination));
// destination n'est PAS terminée par '\0' ici : les 10 octets sont remplis
// sans qu'aucun octet nul n'ait pu être copié.
printf("%s\n", destination); // comportement indéfini : strlen/printf liront au-delà du tableau
```

`strncpy` copie au plus `n` octets, mais si la source est plus longue (ou de longueur égale) que `n`, elle **ne termine pas** la destination par `'\0'`. Toute fonction utilisée ensuite sur `destination` en supposant qu'il s'agit d'une chaîne C valide (comme `printf("%s", ...)` ou `strlen`) lira alors au-delà du tableau. La bonne pratique consiste à forcer explicitement la terminaison :

```c
char destination[10];
strncpy(destination, "Un texte assez long", sizeof(destination) - 1);
destination[sizeof(destination) - 1] = '\0'; // terminaison garantie
```

### Bonne pratique : privilégier `snprintf`

```c
char message[50];
snprintf(message, sizeof(message), "Utilisateur : %s (id=%d)", nom, id);
```

`snprintf` prend explicitement la taille du buffer de destination et garantit qu'elle ne sera jamais dépassée, tout en garantissant la terminaison par `'\0'` (sauf cas de taille nulle). C'est l'une des fonctions les plus sûres et les plus flexibles pour construire des chaînes en C, et elle doit être préférée systématiquement à `strcpy`/`strcat` dans du code nouveau.

## 5. Tableaux de chaînes

```c
const char *jours[] = {"Lundi", "Mardi", "Mercredi"};
printf("%s\n", jours[1]); // Mardi
```

Ici, `jours` est un tableau de pointeurs vers des chaînes littérales (stockées en mémoire statique, en lecture seule). Le mot-clé `const` est important : une chaîne littérale ne doit jamais être modifiée via un pointeur non constant, car cela constitue un comportement indéfini (certaines plateformes stockent effectivement les littéraux en mémoire protégée en écriture).

## 6. Parcours et algorithmes classiques

- **Recherche** d'un élément : recherche séquentielle (O(n), fonctionne sur tout tableau), ou recherche dichotomique sur tableau **trié** (O(log n), en divisant à chaque étape l'intervalle de recherche par deux).
- **Tri** : tri par sélection, tri par insertion, tri à bulles (introduction pédagogique aux algorithmes de tri, tous en O(n²) dans le pire cas), complétés par des algorithmes plus efficaces — tri rapide (*quicksort*), tri fusion (*mergesort*), tous deux en O(n log n) en moyenne — étudiés en travaux pratiques.
- **Inversion** d'un tableau en place, par la technique dite des deux pointeurs (un indice au début, un à la fin, qui se rapprochent en échangeant les éléments) : une technique élémentaire mais réutilisée dans des contextes très variés, y compris en cryptographie pour l'endianness (l'ordre des octets d'un nombre) et le padding (remplissage) de blocs de données.

```c
void inverser(int t[], int taille) {
    int debut = 0, fin = taille - 1;
    while (debut < fin) {
        int temp = t[debut];
        t[debut] = t[fin];
        t[fin] = temp;
        debut++;
        fin--;
    }
}
```

## À retenir

- Un tableau C ne connaît pas sa propre taille à l'exécution : il faut la transmettre séparément, notamment lors du passage en paramètre de fonction, où il « dégrade » en simple pointeur.
- Aucune vérification de borne, ni à la compilation ni à l'exécution : la responsabilité de la validité des indices incombe entièrement au programmeur.
- Une chaîne C est un tableau de `char` terminé par `'\0'` ; toute fonction qui manipule des chaînes sans vérifier les tailles disponibles (`strcpy`, `strcat`, `strncpy` mal utilisée) est un point d'attention sécurité de premier plan.
- Préférer systématiquement les fonctions bornées et sûres (`snprintf`, `strncat` avec vérification manuelle du `'\0'` final) et toujours vérifier la taille des buffers avant toute copie.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
