# Chapitre 2 — Cryptanalyse des chiffrements classiques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/02-cryptanalyse-chiffrements-classiques.txt){ target=_blank }

## 1. Rappel : chiffrement de César et substitution mono-alphabétique

Le chiffre de César décale chaque lettre d'un nombre fixe de positions (clé de 0 à 25). Une substitution mono-alphabétique généralise le principe en remplaçant chaque lettre par une autre selon une permutation quelconque de l'alphabet.

Il est instructif de comparer les deux espaces de clés : le chiffre de César n'offre que 26 clés possibles, tandis qu'une substitution mono-alphabétique générale sur un alphabet de 26 lettres en offre 26! (factorielle de 26), soit environ 4 × 10^26 permutations — un nombre bien supérieur à l'espace de clé d'AES-128 (2^128 ≈ 3,4 × 10^38, certes plus grand, mais du même ordre de grandeur de démesure apparente). Ce constat est pédagogiquement important : **un espace de clé gigantesque ne garantit en rien la sécurité d'un algorithme** si sa structure laisse filtrer de l'information exploitable — ici, la distribution de fréquence des lettres, totalement indépendante de la taille de l'espace de clé.

## 2. Analyse fréquentielle

Chaque langue possède une distribution caractéristique de fréquence des lettres (en français : E ≈ 14,7 %, A ≈ 7,6 %, S ≈ 7,9 %, etc.). Une substitution mono-alphabétique conserve cette distribution en la « déplaçant » : la lettre la plus fréquente du texte chiffré correspond très probablement à la lettre la plus fréquente de la langue.

### Méthode

1. Compter la fréquence de chaque lettre dans le texte chiffré.
2. Comparer au profil de fréquence connu de la langue.
3. Émettre des hypothèses de correspondance pour les lettres les plus fréquentes.
4. Affiner par l'analyse des digrammes/trigrammes fréquents (« ES », « LE », « ENT » en français) et par le sens du texte partiellement déchiffré.
5. Itérer jusqu'à obtenir un texte cohérent.

Pour le chiffre de César spécifiquement, il suffit de tester les 25 décalages possibles (attaque exhaustive triviale, espace de clé minuscule) ; l'analyse fréquentielle n'est même pas strictement nécessaire, mais elle permet d'automatiser le choix du bon décalage sans intervention humaine, ce qui devient précieux dès que le volume de texte à traiter augmente.

### Illustration : automatiser le choix du décalage par score fréquentiel

Le programme suivant illustre comment transformer l'étape « émettre des hypothèses » en un score numérique, en comparant la distribution du texte déchiffré candidat à la distribution théorique du français (valeurs arrondies, à but strictement illustratif).

```python
# Score fréquentiel pédagogique pour départager les hypothèses de décalage

FREQ_FR = {  # fréquences approximatives en français (%), à but illustratif
    'a': 7.6, 'e': 14.7, 'i': 7.5, 's': 7.9, 't': 7.2, 'n': 7.1, 'r': 6.6,
    'u': 6.3, 'l': 5.5, 'o': 5.3, 'd': 3.7, 'c': 3.3, 'p': 3.0, 'm': 3.0,
}

def score_frequentiel(texte):
    """Somme des fréquences théoriques des lettres présentes : un texte
    en français plausible obtient un score élevé, un charabia un score
    plus faible (heuristique simple, pas une preuve)."""
    return sum(FREQ_FR.get(c, 0) for c in texte.lower())

def meilleure_hypothese(texte_chiffre, dechiffrer_cesar):
    scores = []
    for cle in range(26):
        clair_candidat = dechiffrer_cesar(texte_chiffre, cle)
        scores.append((score_frequentiel(clair_candidat), cle, clair_candidat))
    scores.sort(reverse=True)
    return scores[0]  # meilleure hypothèse selon le score fréquentiel
```

Ce type de score, combiné à un dictionnaire de mots valides, est la base des outils automatiques de cassage de substitutions simples : plutôt que de tester manuellement 26 hypothèses, on laisse la machine classer les candidats par plausibilité linguistique.

## 3. Le chiffre de Vigenère et son cassage

Le chiffre de Vigenère applique un décalage de César différent à chaque position, répété selon une clé de longueur *L*. Il résiste à l'analyse fréquentielle directe (la distribution des lettres chiffrées est « aplatie »), mais reste cassable :

### Étape 1 — Trouver la longueur de la clé

- **Méthode de Kasiski** : repérer les répétitions de séquences de lettres identiques dans le texte chiffré ; la distance entre deux occurrences est probablement un multiple de la longueur de clé. Le PGCD des distances observées donne une estimation de *L*. Cette méthode fonctionne d'autant mieux que le texte est long : plus le texte clair contient de répétitions naturelles (mots courants, terminaisons), plus les coïncidences dans le texte chiffré, alignées sur un multiple de *L*, sont nombreuses.
- **Indice de coïncidence** : mesure statistique de la probabilité que deux lettres tirées au hasard dans un texte soient identiques. Un texte en langue naturelle a un indice de coïncidence élevé (≈ 0,0778 en français) ; un texte purement aléatoire a un indice bas (≈ 1/26 ≈ 0,0385). En découpant le texte chiffré en *L* sous-suites (une par position de clé) et en testant plusieurs valeurs de *L*, on retient la valeur pour laquelle l'indice de coïncidence moyen des sous-suites se rapproche de celui de la langue.

L'indice de coïncidence d'un texte de longueur *N*, où *n_i* désigne le nombre d'occurrences de la *i*-ème lettre de l'alphabet, se calcule par :

```
IC = Σ [ n_i × (n_i − 1) ] / [ N × (N − 1) ]
```

Intuitivement, ce calcul estime la probabilité qu'en tirant deux lettres au hasard sans remise dans le texte, elles soient identiques. Un texte chiffré par simple décalage (comme chaque sous-suite d'un Vigenère correctement découpé) conserve exactement la même distribution que le texte clair sous-jacent, à une permutation circulaire près — d'où un IC proche de celui de la langue naturelle dès que *L* est correctement deviné, et un IC proche de l'aléatoire uniforme sinon (les lettres de différentes sous-clés se mélangent).

### Étape 2 — Casser chaque sous-alphabet

Une fois *L* connue, chaque sous-suite est un simple chiffre de César indépendant, cassable par analyse fréquentielle classique (section précédente). Une variante plus robuste consiste, pour chaque sous-suite, à calculer une corrélation croisée entre la distribution observée et la distribution théorique décalée de chaque valeur possible (0 à 25), et à retenir le décalage donnant la meilleure corrélation — c'est l'équivalent statistique, généralisé, du score fréquentiel présenté en section 2.

### Exemple illustratif simplifié (chiffres fictifs, à but pédagogique)

Supposons un texte chiffré dans lequel la séquence de quatre lettres `XKCD` réapparaît trois fois, aux positions 10, 34 et 58 (exemple purement fictif construit pour l'illustration). Les distances entre occurrences sont 34 − 10 = 24 et 58 − 34 = 24 ; le PGCD de ces distances vaut 24, dont les diviseurs (1, 2, 3, 4, 6, 8, 12, 24) constituent les longueurs de clé candidates. En combinant cette information avec le test de l'indice de coïncidence pour chaque diviseur plausible (typiquement entre 3 et 12 pour une clé raisonnable), on retient généralement une valeur unique de *L*, par exemple *L* = 6 dans cet exemple fictif, avant de passer à l'étape 2.

## 4. Autres chiffrements classiques

- **Chiffre de Playfair** (digrammes) : chiffre chaque paire de lettres consécutives selon une grille 5×5 construite à partir d'un mot-clé, ce qui masque partiellement les statistiques mono-lettre. Il résiste donc mieux à l'analyse fréquentielle simple, mais reste vulnérable à l'analyse des digrammes fréquents (certaines paires de lettres, comme « ES », « LE », « DE » en français, restent statistiquement dominantes), combinée à des contraintes structurelles propres à Playfair (pas de digramme composé de deux lettres identiques, par exemple), qui réduisent l'espace des hypothèses.
- **Chiffre de transposition** (permutation des lettres sans substitution) : la distribution de fréquence des lettres est conservée à l'identique (contrairement à la substitution), ce qui trahit immédiatement ce type de chiffrement (un simple calcul de fréquence, comparé au profil de la langue, révèle une correspondance quasi parfaite lettre à lettre) et oriente vers une cryptanalyse par recherche d'anagrammes/permutations, souvent guidée par la longueur du texte (indice sur la taille probable de la grille de transposition) et par des mots probables (*cribs*) recherchés sous forme de permutations locales.
- **Chiffres homophoniques** : pour aplatir la distribution de fréquence, certaines variantes historiques associent à une lettre fréquente (comme le E) plusieurs symboles chiffrés différents utilisés alternativement. Cette technique complique l'analyse fréquentielle simple au niveau des lettres, mais laisse généralement subsister des biais exploitables au niveau des digrammes et de la structure du langage, et reste cassable avec davantage de texte chiffré et de patience.

## 5. Ce que cette cryptanalyse historique enseigne

- Toute redondance statistique du texte clair qui « survit » au chiffrement est exploitable — principe qui reste valable pour évaluer des schémas modernes mal conçus (ex. mode ECB, chapitre TP3).
- Un espace de clé trop petit (chiffre de César : 25 clés) rend l'attaque exhaustive triviale, quelle que soit la qualité apparente de l'algorithme.
- Un espace de clé très grand (substitution mono-alphabétique : 26!) ne suffit pas à garantir la sécurité si une propriété statistique (ici, la fréquence des lettres) fuite malgré le chiffrement — la taille de l'espace de clé est une condition **nécessaire**, jamais **suffisante**.
- La confusion (rendre la relation clé/texte chiffré complexe) et la diffusion (répartir l'influence de chaque bit du clair sur tout le chiffré), concepts formalisés plus tard par Claude Shannon, sont directement motivées par la nécessité de résister à l'analyse fréquentielle : un bon chiffrement moderne doit, par construction, produire un texte chiffré statistiquement indiscernable d'une suite aléatoire.

### Approfondissement : de la cryptanalyse manuelle à la cryptanalyse assistée par ordinateur

Les techniques présentées dans ce chapitre (comptage de fréquences, calcul de l'indice de coïncidence, recherche de répétitions) étaient historiquement réalisées à la main, ce qui limitait la longueur des textes analysables et le nombre d'hypothèses testées. Leur automatisation informatique — même avec des scripts très simples comme ceux présentés ci-dessus — change radicalement l'échelle de l'attaque : un ordinateur personnel actuel peut tester des milliers d'hypothèses de longueur de clé et des millions de sous-hypothèses de décalage en une fraction de seconde. Ce constat annonce un thème central du chapitre 3 : à mesure que la puissance de calcul disponible augmente, la marge de sécurité qu'un concepteur d'algorithme doit intégrer doit croître en conséquence.

## À retenir

- L'analyse fréquentielle casse toute substitution mono-alphabétique, quelle que soit la taille apparente de son espace de clé (26! permutations).
- Le chiffre de Vigenère se casse en deux temps : détermination de la longueur de clé (Kasiski, indice de coïncidence, formule IC = Σ n_i(n_i−1) / N(N−1)), puis cryptanalyse fréquentielle de chaque sous-alphabet.
- Playfair et les chiffres homophoniques compliquent l'analyse fréquentielle simple mais restent vulnérables à des attaques statistiques d'ordre supérieur (digrammes, contraintes structurelles).
- Ces méthodes historiques posent les bases conceptuelles (confusion, diffusion, exploitation de redondance) de la cryptanalyse moderne, et leur automatisation informatique illustre pourquoi la marge de sécurité des algorithmes doit suivre la croissance de la puissance de calcul disponible.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
