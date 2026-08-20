# Chapitre 4 — Fonctions et portée des variables

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/04-fonctions-portee.txt){ target=_blank }

## 1. Pourquoi découper en fonctions ?

Une fonction encapsule un traitement réutilisable, améliore la lisibilité, facilite les tests unitaires et limite la portée des erreurs. C'est un principe de base du *Security by design* : plus une fonction est petite et ciblée, plus elle est facile à auditer.

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

Le prototype permet d'appeler une fonction avant sa définition dans le fichier, et de vérifier la cohérence des types passés — le compilateur détecte alors des erreurs qu'une absence de prototype laisserait passer silencieusement.

## 3. Passage de paramètres

- **Passage par valeur** (défaut en C) : la fonction reçoit une copie ; toute modification est locale.
- **Passage par adresse** (via pointeur) : permet à la fonction de modifier la variable de l'appelant.

```c
void incrementer(int *x) {
    (*x)++;
}

int main(void) {
    int n = 5;
    incrementer(&n);   // n vaut 6 après l'appel
    return 0;
}
```

## 4. Portée (scope) et durée de vie

| Type de variable | Portée | Durée de vie |
|---|---|---|
| locale | bloc où elle est déclarée | jusqu'à la fin du bloc (pile) |
| globale | tout le fichier (ou programme si non `static`) | toute l'exécution |
| `static` locale | bloc de déclaration | toute l'exécution, valeur conservée entre appels |
| paramètre | corps de la fonction | durée de l'appel |

```c
int compteur_appels(void) {
    static int n = 0;  // initialisée une seule fois
    n++;
    return n;
}
```

Les variables globales facilitent le partage d'état mais nuisent à la prévisibilité du code et compliquent l'analyse de sécurité (effets de bord non locaux) : à limiter au strict nécessaire.

## 5. Récursivité

```c
unsigned long factorielle(unsigned int n) {
    if (n == 0) return 1;
    return n * factorielle(n - 1);
}
```

Chaque appel récursif consomme de la pile ; une récursion non bornée ou trop profonde provoque un **stack overflow**, un cas concret de vulnérabilité par épuisement de ressource.

## 6. Bonnes pratiques

- Une fonction = une responsabilité claire.
- Toujours valider les paramètres reçus (pointeurs non nuls, tailles cohérentes).
- Documenter précondition/postcondition, y compris qui est responsable de libérer une mémoire éventuellement allouée.

## À retenir

- Le passage par adresse est le mécanisme C pour qu'une fonction modifie les données de l'appelant.
- La portée `static` locale crée un état persistant discret : à utiliser avec parcimonie.
- La récursivité est élégante mais bornée par la taille de la pile.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
