# Chapitre 3 — Cryptanalyse des chiffrements symétriques modernes

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/03-cryptanalyse-symetrique-moderne.txt){ target=_blank }

## 1. Attaque par force brute (exhaustive key search)

Pour un chiffrement par bloc à clé de *n* bits, l'attaque par force brute nécessite en moyenne 2^(n-1) essais (on s'attend statistiquement à trouver la bonne clé à mi-parcours de l'espace des 2^n clés possibles, en testant chaque hypothèse à l'aide d'un couple clair/chiffré connu). C'est pourquoi la taille de clé est le premier paramètre de sécurité : DES (56 bits) est aujourd'hui cassable par force brute en quelques heures avec du matériel spécialisé, ce qui l'a rendu obsolète au profit d'AES (128, 192 ou 256 bits), hors de portée de la force brute avec la technologie actuelle et prévisible.

### Zoom : structure de DES et controverse historique des boîtes-S

DES (*Data Encryption Standard*), standardisé en 1977, repose sur un **réseau de Feistel** à 16 tours : à chaque tour, la moitié droite du bloc est transformée par une fonction dépendant d'une sous-clé de tour puis combinée par XOR à la moitié gauche, les deux moitiés étant ensuite échangées. Le cœur non linéaire de cette fonction repose sur huit **boîtes-S** (S-boxes), tables de substitution fixes qui introduisent la confusion nécessaire à la sécurité.

DES, dérivé de l'algorithme Lucifer d'IBM, a vu ses boîtes-S modifiées par la NSA avant standardisation, ce qui a suscité une controverse durable sur d'éventuelles portes dérobées. L'historique a montré, avec la publication académique de la cryptanalyse différentielle par Biham et Shamir à la fin des années 1980, que ces modifications renforçaient en réalité la résistance de DES à cette attaque — laissant penser que la NSA connaissait déjà cette technique en interne avant sa découverte publique. Cet épisode illustre un principe de gouvernance important en cryptographie : la confiance dans un algorithme se construit par un examen public ouvert, pas par la confiance dans une autorité de conception, aussi compétente soit-elle.

## 2. Cryptanalyse différentielle

Introduite publiquement par Biham et Shamir (fin des années 1980) contre DES. Principe :

1. Choisir une paire de textes clairs présentant une différence fixe (souvent un XOR précis), notée Δ.
2. Chiffrer les deux textes et observer la différence entre les chiffrés correspondants.
3. Certaines différences de sortie apparaissent avec une probabilité plus élevée que si le chiffrement était parfaitement aléatoire : ce biais statistique permet de déduire de l'information sur des bits de la clé (ou de sous-clés de tour).
4. Répéter sur de nombreuses paires pour affiner la déduction.

DES s'est révélé partiellement vulnérable à cette technique mais sa conception (notamment le choix des boîtes-S) avait déjà anticipé et limité cette attaque — la NSA connaissait apparemment la cryptanalyse différentielle avant sa publication académique.

### Illustration pédagogique simplifiée d'un biais différentiel

Considérons un chiffrement par bloc jouet totalement fictif, à but strictement illustratif, opérant sur des blocs de 4 bits avec une seule boîte-S non linéaire. Une **caractéristique différentielle** décrit la probabilité qu'une différence d'entrée Δ_in produise une différence de sortie Δ_out donnée, après passage par la boîte-S :

```python
# Boîte-S jouet fictive (4 bits -> 4 bits), à but strictement pédagogique
SBOX_JOUET = [0xE, 0x4, 0xD, 0x1, 0x2, 0xF, 0xB, 0x8,
              0x3, 0xA, 0x6, 0xC, 0x5, 0x9, 0x0, 0x7]

def table_differentielle(sbox):
    """Construit la table de distribution des différences (DDT) :
    pour chaque différence d'entrée, compte combien de paires
    d'entrées produisent chaque différence de sortie."""
    taille = len(sbox)
    table = [[0] * taille for _ in range(taille)]
    for x in range(taille):
        for delta_in in range(taille):
            x2 = x ^ delta_in
            delta_out = sbox[x] ^ sbox[x2]
            table[delta_in][delta_out] += 1
    return table

# Une caractéristique différentielle "intéressante" pour un cryptanalyste
# est une paire (delta_in, delta_out) dont le comptage dans la table
# dépasse significativement la moyenne attendue par hasard (taille / 2^n).
```

Dans un chiffrement à plusieurs tours, l'attaquant enchaîne des caractéristiques différentielles tour après tour pour construire une **caractéristique différentielle globale** dont la probabilité, bien que faible, reste significativement supérieure à celle attendue pour un chiffrement parfaitement aléatoire (2^(-n) pour un bloc de *n* bits). En chiffrant un grand nombre de paires respectant la différence d'entrée attendue et en comptant les occurrences de la différence de sortie prédite, les sous-clés cohérentes avec cette observation se distinguent statistiquement des sous-clés incorrectes — c'est le principe du **comptage de clés candidates** au dernier tour, répété jusqu'à isoler la bonne sous-clé.

La résistance d'un algorithme à la cryptanalyse différentielle dépend directement de la qualité de ses boîtes-S (une boîte-S "idéale" minimise le plus grand coefficient de sa table de distribution des différences) et du nombre de tours : plus il y a de tours, plus la probabilité de la caractéristique globale s'effondre, jusqu'à devenir inexploitable en pratique.

## 3. Cryptanalyse linéaire

Introduite par Matsui (1993). Principe : rechercher des approximations linéaires (combinaisons XOR de bits d'entrée, de sortie et de clé) vraies avec une probabilité significativement différente de 1/2. Une approximation suffisamment biaisée, combinée à un grand nombre de couples clair/chiffré connus, permet de retrouver des bits de clé par vote statistique (lemme du pilage/*piling-up lemma*).

### Le lemme du pilage (piling-up lemma)

Si l'on combine *k* approximations linéaires indépendantes, chacune vraie avec une probabilité (1/2 + ε_i) où ε_i est un petit biais, la probabilité que la combinaison (XOR) de toutes les approximations soit vraie s'exprime par :

```
P = 1/2 + 2^(k-1) × Π ε_i
```

Ce lemme montre que le biais global décroît très rapidement (de façon multiplicative) lorsque l'on enchaîne plusieurs approximations à travers les tours successifs d'un chiffrement — d'où l'importance, pour un concepteur d'algorithme, de garantir qu'aucune boîte-S ne présente d'approximation linéaire trop biaisée, et de multiplier le nombre de tours pour que le biais cumulé devienne négligeable. Du point de vue de l'attaquant, le nombre de couples clair/chiffré nécessaires pour exploiter un biais ε croît en 1/ε², ce qui borne directement la faisabilité pratique de l'attaque : un biais minuscule exige un volume de données irréaliste à collecter.

## 4. Pourquoi AES résiste-t-il mieux ?

AES a été conçu (concours public NIST, choix de Rijndael en 2001) en intégrant explicitement des critères de résistance à la cryptanalyse différentielle et linéaire dès sa conception (structure des boîtes-S optimisée, nombre de tours dimensionné avec marge de sécurité). Aucune attaque académique connue à ce jour ne casse AES pleine puissance plus rapidement qu'une recherche exhaustive proche de l'optimum théorique.

### Zoom : structure SPN d'AES

Contrairement à DES (réseau de Feistel), AES est un **réseau de substitution-permutation (SPN)** : chaque tour applique une substitution non linéaire (une unique boîte-S appliquée octet par octet, construite à partir de l'inverse dans un corps fini GF(2⁸), choisie précisément pour minimiser les biais différentiels et linéaires), suivie d'étapes de diffusion linéaire (décalage de lignes, mélange de colonnes) qui propagent rapidement l'influence de chaque octet sur l'ensemble du bloc. Le nombre de tours dépend de la taille de clé : 10 tours pour AES-128, 12 pour AES-192, 14 pour AES-256, un choix qui intègre une marge de sécurité au-delà du nombre de tours strictement nécessaire pour contrer les meilleures cryptanalyses connues au moment de la conception. C'est cette combinaison de boîtes-S mathématiquement bien comprises et de diffusion rapide (dite propriété de « diffusion maximale » sur quatre tours) qui explique la résistance durable d'AES.

## 5. Attaques structurelles sur les modes opératoires

Un algorithme de bloc robuste (AES) peut néanmoins être compromis par un **mode opératoire** mal choisi ou mal utilisé :

| Mode | Faiblesse exploitable |
|---|---|
| **ECB** (Electronic Codebook) | des blocs de clair identiques produisent des blocs chiffrés identiques → motifs visibles (voir TP3) |
| **CBC sans authentification** | vulnérable aux attaques par *padding oracle* si un serveur révèle si le padding est valide (voir TP3 et chapitre 6) |
| **Réutilisation de nonce/IV** (CTR, GCM) | annule la sécurité du chiffrement en flux résultant, permet de retrouver le XOR de deux messages clairs ; pour GCM, la réutilisation de nonce compromet en plus l'intégrité (falsification de messages) |
| **IV prévisible en CBC** | si le vecteur d'initialisation est prévisible par l'attaquant (et non aléatoire), certaines attaques par distinction deviennent possibles sur des messages choisis |

Ces attaques ne cassent pas l'algorithme de bloc lui-même : elles exploitent un mauvais usage, ce qui souligne l'importance, en cryptographie appliquée, de suivre des schémas éprouvés (AES-GCM, ChaCha20-Poly1305) plutôt que de composer soi-même bloc + mode.

### Illustration : pourquoi ECB fuite des motifs

```python
# Démonstration pédagogique (ne pas utiliser ECB en production) :
# chiffrer des blocs de clair identiques en ECB produit des blocs
# chiffrés identiques, ce qui trahit la structure du message d'origine
# (ex. zones uniformes d'une image, champs répétés d'un formulaire).

def chiffrement_ecb_jouet(blocs_clairs, chiffrer_bloc, cle):
    """Simulation conceptuelle : chaque bloc est chiffré indépendamment,
    sans dépendance au bloc précédent (contrairement à CBC/CTR)."""
    return [chiffrer_bloc(bloc, cle) for bloc in blocs_clairs]

# Si blocs_clairs contient deux blocs identiques (ex. b0 == b3),
# alors, quelle que soit la robustesse de chiffrer_bloc (même AES),
# les blocs chiffrés correspondants seront eux aussi identiques :
# c'est une fuite structurelle du MODE, pas de l'algorithme de bloc.
```

## 6. Attaques par compromis temps-mémoire

Les **tables arc-en-ciel (rainbow tables)** illustrent un compromis entre temps de calcul et espace de stockage pour inverser une fonction à sens unique (utilisées historiquement contre des hachages de mots de passe non salés — approfondi au chapitre 4). Le principe général du compromis temps-mémoire, formalisé par Martin Hellman dès 1980, consiste à précalculer une structure de données (chaînes de réduction/hachage successives) qui permet, lors de l'attaque, de retrouver une préimage en un temps très inférieur à une recherche exhaustive brute, au prix d'un espace de stockage important calculé une seule fois puis réutilisable contre toute cible partageant le même espace de clé (par exemple tous les mots de passe non salés hachés avec le même algorithme).

### Approfondissement : pourquoi le salage neutralise ce compromis

L'ajout d'un sel aléatoire unique avant hachage (détaillé au chapitre 4) multiplie l'espace effectif à précalculer par le nombre de sels possibles, rendant une table arc-en-ciel générique inexploitable : l'attaquant devrait précalculer une table distincte pour chaque valeur de sel, ce qui annule l'intérêt économique du précalcul. C'est un exemple typique où une contre-mesure très simple (quelques octets aléatoires) neutralise une classe entière d'attaques par compromis temps-mémoire, sans changer l'algorithme de hachage lui-même.

## À retenir

- La taille de clé fixe la borne théorique de résistance à la force brute (2^(n-1) essais en moyenne) ; DES est aujourd'hui cassable, AES ne l'est pas avec la technologie actuelle.
- Cryptanalyse différentielle (biais de propagation de différences via la table de distribution des différences) et linéaire (biais d'approximations XOR, lemme du pilage) exploitent des biais statistiques internes à la structure de l'algorithme et ont directement influencé la conception des boîtes-S et du nombre de tours d'AES.
- La structure SPN d'AES (boîtes-S issues de GF(2⁸), diffusion rapide, marge de tours) explique sa résistance durable, alors que la structure de Feistel de DES, combinée à une clé trop courte, l'a rendu obsolète.
- La grande majorité des compromissions pratiques de chiffrements symétriques modernes viennent d'un mauvais mode opératoire (ECB, IV/nonce réutilisé, CBC non authentifié) ou d'une mauvaise gestion des paramètres, pas d'une faille dans l'algorithme de bloc lui-même.
- Les compromis temps-mémoire (tables arc-en-ciel) illustrent qu'un précalcul ciblé peut rendre une attaque exhaustive plus rapide qu'attendu, mais restent neutralisés par des contre-mesures simples comme le salage.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
