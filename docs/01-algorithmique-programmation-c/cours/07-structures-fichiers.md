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

Une structure regroupe des données hétérogènes sous un seul type, brique de base pour modéliser des objets métier (utilisateur, paquet réseau, enregistrement de log, en-tête de fichier).

## 2. Structures et pointeurs

```c
struct Utilisateur *pu = &u1;
printf("%s\n", pu->nom);   // équivalent à (*pu).nom
```

L'opérateur `->` est l'un des plus fréquents dans du code C réel (listes chaînées, arbres, structures de données de systèmes d'exploitation ou de protocoles réseau).

## 3. `typedef` et alias de type

```c
typedef struct {
    int x;
    int y;
} Point;

Point p = {3, 4};
```

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

Cette structure combine tout ce qui a été vu précédemment : allocation dynamique, pointeurs, structures.

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
char ligne[100];
while (fgets(ligne, sizeof(ligne), lecture) != NULL) {
    printf("%s", ligne);
}
fclose(lecture);
```

Modes d'ouverture courants : `"r"` (lecture), `"w"` (écriture, écrase), `"a"` (ajout), `"rb"`/`"wb"` (binaire).

**Point de vigilance sécurité** : toujours vérifier la valeur de retour de `fopen`, borner la taille lue (`fgets` avec taille explicite plutôt que `gets`, qui est supprimée du standard C11 pour cette raison), et valider tout chemin de fichier construit à partir d'une entrée utilisateur (risque de *path traversal*).

## 6. Lecture/écriture binaire

```c
struct Utilisateur u;
fwrite(&u, sizeof(struct Utilisateur), 1, f);
fread(&u, sizeof(struct Utilisateur), 1, f);
```

Utile pour sérialiser des structures de données, mais sensible aux différences d'alignement mémoire et d'endianness entre systèmes — sujet repris en *Cryptographie* (encodage des données avant chiffrement) et *Technologie Blockchain* (sérialisation des transactions).

## À retenir

- `struct` modélise des données composites ; combinée à des pointeurs, elle permet de construire des structures de données dynamiques (listes, arbres).
- La gestion de fichiers en C impose une vérification systématique des erreurs (`fopen`, `fread`, `fwrite`) et une fermeture explicite (`fclose`).
- Ce chapitre clôt le socle du langage : les TP suivants consolident ces notions sur des cas pratiques complets.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
