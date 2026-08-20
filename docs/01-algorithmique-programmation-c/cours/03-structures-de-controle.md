# Chapitre 3 — Structures de contrôle

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=01-algorithmique-programmation-c/slides/03-structures-de-controle.txt){ target=_blank }

## 1. Structures conditionnelles

Les structures conditionnelles permettent de faire dépendre l'exécution d'une ou plusieurs instructions de la valeur d'une expression booléenne. En C, toute expression peut être utilisée comme condition : elle est considérée fausse si elle vaut 0, vraie dans tous les autres cas. C'est une source fréquente d'erreurs subtiles (par exemple confondre `=` et `==`), détaillée plus bas.

### `if / else if / else`

```c
int note = 12;
if (note >= 16) {
    printf("Très bien\n");
} else if (note >= 10) {
    printf("Admis\n");
} else {
    printf("Ajourné\n");
}
```

Les blocs `{ }` sont facultatifs en C lorsque le corps du `if` (ou du `else`) ne comporte qu'une seule instruction, mais il est fortement recommandé de toujours les écrire, y compris pour une instruction unique. Cette discipline évite un piège classique :

```c
if (utilisateur_authentifie)
    printf("Accès autorisé\n");
    ouvrir_session();   // INDENTÉ MAIS PAS DANS LE if ! s'exécute toujours
```

Dans cet exemple, l'indentation suggère que `ouvrir_session()` ne s'exécute que si l'utilisateur est authentifié, alors qu'en réalité elle s'exécute **systématiquement**, car seule l'instruction `printf` appartient au `if` en l'absence d'accolades. Ce type d'erreur, purement visuel, a été à l'origine de vulnérabilités réelles dans du code historique (le bug dit « goto fail » d'une célèbre implémentation TLS en est l'exemple le plus souvent cité, bien qu'il s'agisse d'un `goto` sans accolade plutôt que d'un `if`). La règle à retenir : **toujours utiliser des accolades**, même pour un bloc d'une seule ligne.

### Piège classique : `=` au lieu de `==`

```c
int mot_de_passe_ok = 0;
if (mot_de_passe_ok = 1) {     // BUG : affectation, pas comparaison !
    printf("Accès autorisé\n"); // s'exécute toujours, quelle que soit la valeur initiale
}
```

`mot_de_passe_ok = 1` est une expression d'affectation, qui vaut elle-même 1 (donc « vraie ») après exécution : la condition est donc toujours satisfaite, indépendamment de la logique métier attendue. La plupart des compilateurs modernes émettent un avertissement pour ce genre de code avec `-Wall` (`-Wparentheses` notamment) : c'est une raison supplémentaire de toujours activer les avertissements de compilation.

### `switch`

```c
int jour = 3;
switch (jour) {
    case 1: printf("Lundi\n"); break;
    case 2: printf("Mardi\n"); break;
    case 3: printf("Mercredi\n"); break;
    default: printf("Jour invalide\n");
}
```

Le `switch` compare une expression entière (ou de type énumération) à une série de valeurs constantes. L'oubli d'un `break` provoque un **fall-through** : l'exécution continue dans le `case` suivant, sans re-tester sa condition. C'est parfois voulu (par exemple pour regrouper plusieurs cas qui partagent le même traitement), mais c'est le plus souvent une source de bug. Toujours documenter explicitement un fall-through intentionnel par un commentaire, par exemple `// fall-through volontaire`, afin qu'un futur relecteur ne le corrige pas par erreur.

```c
switch (niveau_journalisation) {
    case NIVEAU_CRITIQUE:
    case NIVEAU_ERREUR:
        // fall-through volontaire : critique et erreur partagent le même traitement
        enregistrer_dans_le_journal_urgent();
        break;
    case NIVEAU_INFO:
        enregistrer_dans_le_journal_standard();
        break;
    default:
        break;
}
```

Le cas `default` n'est pas obligatoire syntaxiquement, mais l'omettre revient à supposer implicitement que toutes les valeurs possibles ont été énumérées : une hypothèse souvent fausse (nouvelle valeur ajoutée plus tard, valeur issue d'une entrée non validée). L'inclure systématiquement, même pour simplement journaliser un cas inattendu, est une bonne pratique défensive.

### Opérateur ternaire

```c
int max = (a > b) ? a : b;
```

L'opérateur ternaire `condition ? valeur_si_vrai : valeur_si_faux` est une expression, pas une instruction : il produit une valeur et peut donc être utilisé partout où une expression est attendue (initialisation, argument de fonction). Il est adapté aux choix simples entre deux valeurs ; au-delà, un `if`/`else` classique reste plus lisible.

## 2. Boucles

Une boucle répète un bloc d'instructions tant qu'une condition reste vraie. Le choix de la structure de boucle dépend essentiellement de la connaissance ou non, *a priori*, du nombre d'itérations.

### `while`

```c
int i = 0;
while (i < 5) {
    printf("%d\n", i);
    i++;
}
```

La condition est testée **avant** chaque itération : si elle est fausse dès le départ, le corps de la boucle ne s'exécute jamais.

### `do ... while`

Exécute le corps au moins une fois avant de tester la condition — utile pour valider une saisie utilisateur, car il faut bien demander la saisie au moins une fois avant de pouvoir la vérifier.

```c
int choix;
do {
    printf("Choix (1-3) : ");
    scanf("%d", &choix);
} while (choix < 1 || choix > 3);
```

Cet exemple mérite d'être complété, en pratique, par une vérification de la valeur de retour de `scanf` (voir chapitre 2) : si l'utilisateur saisit une entrée non numérique, `scanf` échoue et `choix` peut conserver une valeur indéterminée à la première itération, ou provoquer une boucle infinie si le flux d'entrée n'est jamais « vidé » des caractères invalides.

### `for`

```c
for (int i = 0; i < 10; i++) {
    printf("%d ", i);
}
```

La boucle `for` regroupe en une seule ligne l'initialisation, la condition de poursuite et la mise à jour du compteur, ce qui la rend particulièrement adaptée lorsque le nombre d'itérations est connu ou calculable à l'avance (parcours de tableau, répétition d'un nombre fixe d'opérations).

```
for (initialisation ; condition ; mise_à_jour) {
    corps de la boucle
}
```

Les trois expressions sont facultatives (`for (;;)` est une boucle infinie valide), et la portée d'une variable déclarée dans l'initialisation (`int i`) est limitée au corps de la boucle depuis le standard C99.

## 3. Rupture de séquence

- `break` : sort immédiatement de la boucle englobante (ou du `switch`), sans exécuter le reste du bloc.
- `continue` : passe directement à l'itération suivante, en sautant le reste du corps de la boucle courante (mais en réévaluant la condition ou, pour un `for`, en exécutant d'abord la mise à jour).
- `goto` : transfère l'exécution vers une étiquette (`label:`) du même fichier. À éviter en général, car il nuit gravement à la lisibilité et à la prévisibilité du flot de contrôle, sauf cas très spécifiques (sortie d'imbrications profondes de boucles, ou gestion d'erreurs façon « cleanup » en C bas niveau, un idiome courant dans le code noyau Linux par exemple).

```c
// Idiome "goto cleanup", courant en C bas niveau
FILE *f = fopen("donnees.txt", "r");
if (f == NULL) goto erreur;

int *tampon = malloc(100 * sizeof(int));
if (tampon == NULL) goto fermer_fichier;

// ... traitement ...

free(tampon);
fermer_fichier:
    fclose(f);
    return 0;
erreur:
    return 1;
```

Cet idiome, bien que dérogeant à la règle générale « éviter `goto` », est largement accepté car il centralise la libération des ressources en un seul endroit, évitant la duplication de code de nettoyage à chaque point de sortie en erreur — une préoccupation directement liée à la prévention des fuites de ressources.

## 4. Boucles infinies et conditions de terminaison

```c
for (;;) {
    // boucle infinie volontaire, sortie via break
}
```

Une condition de terminaison mal posée (par exemple un compteur jamais incrémenté, une condition toujours vraie, ou une comparaison qui ne peut jamais devenir fausse à cause d'un débordement d'entier) provoque une boucle qui ne se termine jamais, entraînant un déni de service par consommation de ressources (temps CPU, voire mémoire si la boucle alloue). Ce cas de figure — une entrée utilisateur qui provoque une boucle infinie ou une consommation excessive de ressources — est un cas particulier à garder en tête pour le cours *Security by design*, où l'on distingue les vulnérabilités qui compromettent la confidentialité ou l'intégrité de celles qui compromettent uniquement la disponibilité.

### Piège : boucle avec compteur non signé décroissant

```c
for (unsigned int i = 10; i >= 0; i--) {
    printf("%u\n", i);
} // boucle infinie : i >= 0 est toujours vrai pour un unsigned int (i "boucle" à un très grand nombre après 0)
```

Comme vu au chapitre 2, un type non signé ne peut jamais être négatif : la condition `i >= 0` est donc toujours vraie, et lorsque `i` atteint 0 puis est décrémenté, elle « boucle » vers la plus grande valeur représentable au lieu de devenir négative. Ce genre d'erreur, purement liée au choix du type, illustre pourquoi il faut choisir ses types entiers avec autant de soin que ses conditions de boucle.

## 5. Complexité algorithmique (introduction)

Le nombre d'itérations d'une boucle détermine la complexité temporelle d'un algorithme, notée en notation de Landau (grand O), qui exprime comment le temps d'exécution croît en fonction de la taille *n* des données, indépendamment des détails d'implémentation ou de la machine utilisée. Exemples :

- une recherche séquentielle dans un tableau de taille *n* est en O(n) : dans le pire cas, il faut examiner chaque élément une fois ;
- une recherche dichotomique dans un tableau **trié** est en O(log n) : chaque comparaison élimine la moitié des candidats restants ;
- une boucle imbriquée dans une autre (par exemple pour comparer chaque paire d'éléments d'un tableau) est typiquement en O(n²).

```c
// Complexité O(n) : une seule boucle, n itérations
for (int i = 0; i < n; i++) { /* ... */ }

// Complexité O(n * n) = O(n²) : boucle imbriquée
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) { /* ... */ }
}
```

Cette notion, encore intuitive à ce stade, sera systématiquement réutilisée pour analyser le coût des algorithmes cryptographiques (chapitre Cryptographie) et pour évaluer la faisabilité pratique des attaques par force brute (chapitre Cryptanalyse) : un espace de clés dont l'exploration exhaustive est en O(2ⁿ) devient rapidement impraticable lorsque *n* augmente, ce qui est précisément le principe de sécurité recherché par les algorithmes de chiffrement modernes.

## À retenir

- Choisir la structure adaptée : `for` pour un nombre d'itérations connu, `while`/`do while` pour une condition dynamique évaluée avant/après le corps.
- Toujours utiliser des accolades, même pour un bloc d'une seule instruction, afin d'éviter les erreurs d'indentation trompeuse.
- Ne jamais confondre `=` (affectation) et `==` (comparaison) dans une condition ; s'appuyer sur les avertissements du compilateur pour les détecter.
- `switch` sans `break` = piège classique de fall-through ; toujours prévoir un `default`.
- Toujours s'assurer qu'une boucle progresse réellement vers sa condition d'arrêt, en particulier avec des compteurs non signés.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
