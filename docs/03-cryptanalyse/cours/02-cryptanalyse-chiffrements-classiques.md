# Chapitre 2 — Cryptanalyse des chiffrements classiques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/02-cryptanalyse-chiffrements-classiques.txt){ target=_blank }

## 1. Rappel : chiffrement de César et substitution mono-alphabétique

Le chiffre de César décale chaque lettre d'un nombre fixe de positions (clé de 0 à 25). Une substitution mono-alphabétique généralise le principe en remplaçant chaque lettre par une autre selon une permutation quelconque de l'alphabet.

## 2. Analyse fréquentielle

Chaque langue possède une distribution caractéristique de fréquence des lettres (en français : E ≈ 14,7 %, A ≈ 7,6 %, S ≈ 7,9 %, etc.). Une substitution mono-alphabétique conserve cette distribution en la « déplaçant » : la lettre la plus fréquente du texte chiffré correspond très probablement à la lettre la plus fréquente de la langue.

### Méthode

1. Compter la fréquence de chaque lettre dans le texte chiffré.
2. Comparer au profil de fréquence connu de la langue.
3. Émettre des hypothèses de correspondance pour les lettres les plus fréquentes.
4. Affiner par l'analyse des digrammes/trigrammes fréquents (« ES », « LE », « ENT » en français) et par le sens du texte partiellement déchiffré.
5. Itérer jusqu'à obtenir un texte cohérent.

Pour le chiffre de César spécifiquement, il suffit de tester les 25 décalages possibles (attaque exhaustive triviale, espace de clé minuscule).

## 3. Le chiffre de Vigenère et son cassage

Le chiffre de Vigenère applique un décalage de César différent à chaque position, répété selon une clé de longueur *L*. Il résiste à l'analyse fréquentielle directe (la distribution des lettres chiffrées est « aplatie »), mais reste cassable :

### Étape 1 — Trouver la longueur de la clé

- **Méthode de Kasiski** : repérer les répétitions de séquences de lettres identiques dans le texte chiffré ; la distance entre deux occurrences est probablement un multiple de la longueur de clé. Le PGCD des distances observées donne une estimation de *L*.
- **Indice de coïncidence** : mesure statistique de la probabilité que deux lettres tirées au hasard dans un texte soient identiques. Un texte en langue naturelle a un indice de coïncidence élevé (≈ 0,0778 en français) ; un texte purement aléatoire a un indice bas (≈ 1/26 ≈ 0,0385). En découpant le texte chiffré en *L* sous-suites (une par position de clé) et en testant plusieurs valeurs de *L*, on retient la valeur pour laquelle l'indice de coïncidence moyen des sous-suites se rapproche de celui de la langue.

### Étape 2 — Casser chaque sous-alphabet

Une fois *L* connue, chaque sous-suite est un simple chiffre de César indépendant, cassable par analyse fréquentielle classique (chapitre précédent).

## 4. Autres chiffrements classiques

- **Chiffre de Playfair** (digrammes) : résiste mieux à l'analyse fréquentielle simple mais reste vulnérable à l'analyse des digrammes fréquents.
- **Chiffre de transposition** (permutation des lettres sans substitution) : la distribution de fréquence des lettres est conservée à l'identique (contrairement à la substitution), ce qui trahit immédiatement ce type de chiffrement et oriente vers une cryptanalyse par recherche d'anagrammes/permutations.

## 5. Ce que cette cryptanalyse historique enseigne

- Toute redondance statistique du texte clair qui « survit » au chiffrement est exploitable — principe qui reste valable pour évaluer des schémas modernes mal conçus (ex. mode ECB, chapitre TP3).
- Un espace de clé trop petit (chiffre de César : 25 clés) rend l'attaque exhaustive triviale, quelle que soit la qualité apparente de l'algorithme.
- La confusion (rendre la relation clé/texte chiffré complexe) et la diffusion (répartir l'influence de chaque bit du clair sur tout le chiffré), concepts formalisés plus tard par Claude Shannon, sont directement motivées par la nécessité de résister à l'analyse fréquentielle.

## À retenir

- L'analyse fréquentielle casse toute substitution mono-alphabétique.
- Le chiffre de Vigenère se casse en deux temps : détermination de la longueur de clé (Kasiski, indice de coïncidence), puis cryptanalyse fréquentielle de chaque sous-alphabet.
- Ces méthodes historiques posent les bases conceptuelles (confusion, diffusion, exploitation de redondance) de la cryptanalyse moderne.
