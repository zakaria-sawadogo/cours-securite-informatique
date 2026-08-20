# Chapitre 7 — Structures (`struct`) et fichiers

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/07-structures-fichiers.txt){ target=_blank }

## 1. Définir une structure

```c
struct Utilisateur {
    char nom[50];
    int age;
    float solde;
};

struct Utilisateur u1 = {"Zakaria", 30, 1500.0f};
printf("%s a %d ans\n", u1.nom, u1.age);
```

Une structure regroupe des données hétérogènes (de types potentiellement différents) sous un seul type nommé, accessible via l'opérateur `.` pour chaque membre. C'est la brique de base pour modéliser des objets métier cohérents (un utilisateur, un paquet réseau, un enregistrement de journal, un en-tête de fichier) plutôt que de manipuler des variables isolées dont la cohérence logique reposerait uniquement sur la discipline du programmeur.

### Disposition mémoire et alignement

Les membres d'une structure sont stockés en mémoire dans l'ordre de leur déclaration, mais le compilateur peut insérer des octets de remplissage (*padding*) entre certains membres, afin de respecter les contraintes d'alignement de l'architecture cible (par exemple, un `int` doit souvent commencer à une adresse multiple de 4). En conséquence, `sizeof(struct Utilisateur)` n'est pas nécessairement égale à la somme des `sizeof` de chaque membre.

```c
struct Exemple {
    char a;   // 1 octet
    int b;    // 4 octets, mais alignement à 4 : 3 octets de padding insérés avant b
    char c;   // 1 octet
};
printf("%zu\n", sizeof(struct Exemple)); // probablement 12, pas 6
```

Cette notion, souvent négligée, est essentielle dès qu'on sérialise une structure (l'écrire telle quelle dans un fichier ou sur le réseau, voir plus bas) : le padding fait partie du contenu binaire écrit, et diffère potentiellement d'une architecture ou d'un compilateur à l'autre.

## 2. Structures et pointeurs

```c
struct Utilisateur *pu = &u1;
printf("%s\n", pu->nom);   // équivalent à (*pu).nom
```

L'opérateur `->` déréférence un pointeur vers une structure puis accède directement à l'un de ses membres, en une seule opération plus lisible que `(*pu).nom`. C'est l'un des opérateurs les plus fréquents dans du code C réel, en particulier dans les structures de données chaînées (listes, arbres), le code de systèmes d'exploitation, ou l'analyse de protocoles réseau, où l'on manipule constamment des pointeurs vers des structures plutôt que des structures directement.

### Passage d'une structure en paramètre : par valeur ou par adresse

```c
void afficher_par_valeur(struct Utilisateur u) {
    // reçoit une COPIE complète de la structure : coûteux si elle est volumineuse
    printf("%s\n", u.nom);
}

void afficher_par_adresse(const struct Utilisateur *u) {
    // reçoit seulement une adresse (8 octets typiquement) ; const interdit toute modification
    printf("%s\n", u->nom);
}
```

Passer une structure « par valeur » à une fonction en copie l'intégralité de son contenu, ce qui peut être coûteux en performance pour de grandes structures, et surprenant si le programmeur s'attend implicitement à ce que la fonction puisse la modifier (elle ne modifiera que sa copie locale, comme vu au chapitre 4 pour les types de base). La convention la plus courante en C consiste donc à transmettre un pointeur vers la structure, en le qualifiant de `const` lorsque la fonction ne doit pas la modifier.

## 3. `typedef` et alias de type

```c
typedef struct {
    int x;
    int y;
} Point;

Point p = {3, 4};
```

`typedef` crée un alias de type, qui permet ici d'utiliser directement `Point` au lieu de `struct Point` (ou, dans cet exemple, de se passer entièrement d'un nom de structure explicite, puisque la structure est anonyme et n'est nommée qu'à travers son alias `Point`). Cette pratique est très répandue en C pour alléger l'écriture, mais elle a un revers : elle peut masquer la nature exacte d'un type (structure, pointeur, type de base) à la lecture, ce qui peut compliquer l'audit d'un code inconnu si les conventions de nommage ne sont pas suivies avec rigueur (par exemple, suffixer par `_t` les types issus d'un `typedef`, comme le fait la bibliothèque standard avec `size_t`).

## 4. Structures de données dynamiques : liste chaînée (introduction)

```c
typedef struct Noeud {
    int valeur;
    struct Noeud *suivant;
} Noeud;

Noeud* creer_noeud(int v) {
    Noeud *n = malloc(sizeof(Noeud));
    if (n == NULL) return NULL;
    n->valeur = v;
    n->suivant = NULL;
    return n;
}
```

Cette structure combine tout ce qui a été vu précédemment : allocation dynamique (chapitre 6), pointeurs (chapitre 6), et structures. Une liste chaînée se construit en reliant des nœuds entre eux via le champ `suivant`, chaque nœud étant alloué indépendamment sur le tas.

### Libération d'une liste chaînée : parcourir avant de détruire

```c
void liberer_liste(Noeud *tete) {
    Noeud *courant = tete;
    while (courant != NULL) {
        Noeud *suivant = courant->suivant; // sauvegarder AVANT de libérer courant
        free(courant);
        courant = suivant;
    }
}
```

Libérer chaque nœud d'une liste chaînée illustre un piège classique déjà évoqué au chapitre 6 : il faut impérativement sauvegarder le pointeur vers le nœud suivant *avant* d'appeler `free` sur le nœud courant, car une fois libéré, lire `courant->suivant` constituerait un accès à une mémoire déjà libérée (*use-after-free*). Cette structure de données est le premier exemple concret, dans ce cours, où l'oubli d'une libération correcte peut se propager en fuite mémoire à chaque insertion, si l'ensemble de la liste n'est jamais parcouru et libéré en fin de programme.

## 5. Manipulation de fichiers

```c
#include <stdio.h>

FILE *f = fopen("donnees.txt", "w");
if (f == NULL) {
    perror("Erreur d'ouverture");
    return 1;
}
fprintf(f, "Bonjour %s\n", "Zakaria");
fclose(f);

FILE *lecture = fopen("donnees.txt", "r");
if (lecture == NULL) {
    perror("Erreur d'ouverture");
    return 1;
}
char ligne[100];
while (fgets(ligne, sizeof(ligne), lecture) != NULL) {
    printf("%s", ligne);
}
fclose(lecture);
```

Le type `FILE` (défini dans `<stdio.h>`) représente un flux ouvert vers un fichier ; toutes les opérations de lecture/écriture passent par un pointeur `FILE *`, jamais par le fichier directement. Modes d'ouverture courants : `"r"` (lecture seule, échoue si le fichier n'existe pas), `"w"` (écriture, crée le fichier ou **écrase** son contenu existant sans avertissement), `"a"` (ajout à la fin du fichier), et leurs variantes binaires `"rb"`/`"wb"`/`"ab"` (qui, sur certains systèmes comme Windows, évitent une conversion automatique des fins de ligne appliquée en mode texte).

### Point de vigilance sécurité : erreurs, tailles, chemins

- **Toujours vérifier la valeur de retour de `fopen`** : elle vaut `NULL` en cas d'échec (fichier inexistant, droits insuffisants, disque plein), et poursuivre l'exécution avec un pointeur `NULL` provoquerait un déréférencement invalide dès le premier appel à `fprintf`/`fgets`.
- **Borner systématiquement la taille lue** : `fgets(ligne, sizeof(ligne), f)` prend explicitement la taille du tampon de destination et s'arrête avant de la dépasser. La fonction historique `gets`, qui ne prenait aucune limite de taille en paramètre, a été supprimée du standard C11 précisément parce qu'elle rendait tout débordement de tampon quasiment inévitable dès qu'une entrée dépassait la taille prévue.
- **Valider tout chemin de fichier construit à partir d'une entrée utilisateur.** Concaténer directement une entrée utilisateur à un chemin de base (par exemple `"/donnees/" + nom_fichier_utilisateur`) expose à une vulnérabilité de type *path traversal* : un attaquant qui fournit un nom contenant `../../etc/passwel` (ou équivalent) peut potentiellement accéder à des fichiers situés en dehors du répertoire prévu. La validation doit rejeter ou neutraliser les séquences de remontée de répertoire (`..`) avant toute utilisation du chemin.

```c
#include <string.h>

int chemin_est_sur(const char *nom_fichier) {
    // Vérification simplifiée à but pédagogique : rejette toute présence de ".."
    return strstr(nom_fichier, "..") == NULL;
}
```

Cette vérification reste volontairement simplifiée à des fins pédagogiques ; une validation robuste en environnement réel s'appuie généralement sur des fonctions dédiées de normalisation de chemin fournies par le système d'exploitation, plutôt que sur une recherche de sous-chaîne, insuffisante face à certains contournements (chemins absolus, liens symboliques, encodages alternatifs).

## 6. Lecture/écriture binaire

```c
struct Utilisateur u = {"Zakaria", 30, 1500.0f};

FILE *f = fopen("utilisateur.bin", "wb");
if (f != NULL) {
    fwrite(&u, sizeof(struct Utilisateur), 1, f);
    fclose(f);
}

struct Utilisateur relue;
FILE *g = fopen("utilisateur.bin", "rb");
if (g != NULL) {
    size_t n = fread(&relue, sizeof(struct Utilisateur), 1, g);
    if (n == 1) {
        printf("%s, %d ans\n", relue.nom, relue.age);
    }
    fclose(g);
}
```

`fwrite`/`fread` écrivent et lisent des blocs bruts d'octets, sans aucune transformation ni interprétation : c'est ce qui les rend rapides, mais aussi rigides. Vérifier la valeur de retour de `fread` (le nombre d'éléments effectivement lus, qui peut être inférieur à ce qui était demandé si le fichier est plus court que prévu, par exemple corrompu ou tronqué) est indispensable avant d'utiliser les données lues.

Cette approche est utile pour sérialiser des structures de données, mais elle reste sensible :

- aux différences d'**alignement mémoire** entre systèmes ou compilateurs (le padding évoqué en section 1 fait partie des octets écrits, et peut varier) ;
- aux différences d'**endianness** (ordre des octets d'un nombre en mémoire : *little-endian* sur la plupart des architectures grand public, *big-endian* sur d'autres) entre la machine qui écrit le fichier et celle qui le relit.

Un fichier binaire écrit avec `fwrite` sur une architecture n'est donc pas garanti portable tel quel vers une architecture différente — un point repris en *Cryptographie* (encodage explicite des données avant chiffrement, souvent via un format indépendant de la plateforme) et en *Technologie Blockchain* (sérialisation déterministe des transactions, un prérequis pour que différents nœuds du réseau calculent exactement le même résultat).

## À retenir

- `struct` modélise des données composites cohérentes ; combinée à des pointeurs et à l'allocation dynamique, elle permet de construire des structures de données dynamiques (listes chaînées, arbres, tables).
- Le compilateur peut insérer du padding dans une structure pour respecter les contraintes d'alignement : `sizeof` d'une structure n'est pas toujours la somme des `sizeof` de ses membres.
- La gestion de fichiers en C impose une vérification systématique des erreurs (`fopen`, `fread`, `fwrite`) et une fermeture explicite (`fclose`) de chaque flux ouvert.
- Toujours borner la taille des données lues (`fgets` avec taille explicite, jamais `gets`) et valider tout chemin de fichier dérivé d'une entrée utilisateur.
- Ce chapitre clôt le socle du langage : les travaux pratiques suivants consolident ces notions sur des cas pratiques complets, combinant structures, pointeurs, tableaux et fichiers.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
