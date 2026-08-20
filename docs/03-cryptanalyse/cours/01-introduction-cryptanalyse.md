# Chapitre 1 — Introduction à la cryptanalyse : histoire et classification des attaques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/01-introduction-cryptanalyse.txt){ target=_blank }

## 1. Définition

La cryptanalyse est l'étude des méthodes permettant de retrouver, en tout ou partie, l'information protégée par un système cryptographique (message clair, clé) sans disposer légitimement de la clé de déchiffrement. Elle est le pendant offensif de la cryptographie, et les deux disciplines progressent ensemble : un algorithme n'est considéré fiable qu'après avoir résisté longtemps à un examen cryptanalytique public et intensif (principe de Kerckhoffs, vu au module *Cryptographie*).

Il est important de distinguer plusieurs objectifs possibles d'une cryptanalyse, du plus fort au plus faible :

- **Rupture totale (total break)** : retrouver la clé secrète elle-même. C'est l'objectif le plus ambitieux, mais aussi le plus rarement atteint contre un algorithme moderne bien conçu.
- **Déduction globale** : retrouver un algorithme fonctionnellement équivalent à l'algorithme légitime, sans connaître la clé exacte.
- **Déduction locale (ou déduction d'information)** : retrouver de l'information partielle sur un clair ou une clé — par exemple quelques bits de clé, ou la capacité de distinguer un chiffré d'une suite aléatoire (attaque par distinction). C'est le type de résultat le plus fréquent en recherche académique : une cryptanalyse « théorique » qui réduit la sécurité d'un algorithme de 2^n à 2^(n-10), par exemple, ne casse pas le système en pratique mais constitue un signal d'alerte scientifique important.
- **Distinction (distinguishing attack)** : montrer qu'il existe un moyen de différencier la sortie de l'algorithme d'une sortie parfaitement aléatoire, sans nécessairement retrouver la clé. C'est souvent la première étape qui ouvre la voie à des attaques plus poussées.

Cette gradation explique pourquoi on lit parfois qu'un algorithme est « cassé » alors qu'il reste, en pratique, totalement sûr pour un usage réel : le terme recouvre des niveaux de gravité très différents, et seule une lecture précise du résultat (complexité, modèle d'attaque, objectif atteint) permet d'en évaluer l'impact réel.

## 2. Repères historiques

- **Antiquité à Renaissance** : chiffrement de César, chiffres de substitution ; cassés par analyse fréquentielle dès le IXe siècle par le savant arabe Al-Kindi, dans son traité *Sur le déchiffrement des messages cryptographiques*, considéré comme le premier texte connu de cryptanalyse systématique.
- **XVIe siècle** : chiffre de Vigenère, longtemps considéré incassable (« le chiffre indéchiffrable »), cassé au XIXe siècle par Kasiski et Babbage.
- **1917, télégramme Zimmermann** : un télégramme diplomatique allemand chiffré, intercepté et déchiffré par les cryptanalystes britanniques de la « Room 40 », révèle une proposition d'alliance avec le Mexique contre les États-Unis ; sa divulgation contribue à l'entrée en guerre des États-Unis. Cet épisode illustre très tôt l'impact géopolitique direct que peut avoir une cryptanalyse réussie.
- **Seconde Guerre mondiale** : cryptanalyse de la machine Enigma. Les travaux précurseurs du bureau du chiffre polonais (Marian Rejewski, Jerzy Różycki, Henryk Zygalski), qui exploitent dès les années 1930 des faiblesses de procédure et développent la « bombe » électromécanique polonaise, sont transmis aux Alliés en 1939 et poursuivis à Bletchley Park par les équipes britanniques (Alan Turing, Gordon Welchman notamment), qui perfectionnent la Bombe de Turing pour automatiser la recherche de réglages. Ce travail est considéré comme un tournant fondateur à la fois pour la cryptanalyse moderne et pour la naissance de l'informatique (Turing y développe des concepts qui nourriront sa réflexion sur le calcul).
- **Ère moderne** : cryptanalyse différentielle et linéaire contre les chiffrements par blocs (DES), attaques mathématiques contre RSA (factorisation), attaques par canaux auxiliaires exploitant l'implémentation physique plutôt que l'algorithme.

### Zoom : pourquoi Enigma était-elle vulnérable malgré un espace de clés énorme ?

Enigma combinait des rotors interchangeables, un réflecteur et un tableau de connexions (*plugboard*), offrant un espace de réglages théoriquement gigantesque. Le système n'a pourtant pas été cassé par force brute, mais par l'exploitation de **faiblesses structurelles et procédurales** :

- le réflecteur garantissait qu'aucune lettre ne pouvait être chiffrée en elle-même — une contrainte qui, combinée à des mots probables (*cribs*, comme des formules météo répétitives ou des salutations standardisées), réduisait drastiquement l'espace de recherche ;
- des habitudes opérationnelles prévisibles des opérateurs (réglages réutilisés, messages stéréotypés) fournissaient des points d'appui réguliers aux cryptanalystes.

Cet exemple illustre un principe qui traverse tout ce cours : **la robustesse théorique d'un système ne suffit pas si sa mise en œuvre introduit des régularités exploitables.**

## 3. Classification des attaques selon les informations disponibles

| Type d'attaque | Information disponible à l'attaquant |
|---|---|
| **Ciphertext-only** (texte chiffré seul) | uniquement des messages chiffrés |
| **Known-plaintext** (texte clair connu) | un ou plusieurs couples (clair, chiffré) |
| **Chosen-plaintext** (texte clair choisi) | l'attaquant peut faire chiffrer des textes de son choix |
| **Chosen-ciphertext** (texte chiffré choisi) | l'attaquant peut faire déchiffrer des textes chiffrés de son choix (hors le message cible) |

Plus l'attaquant dispose de capacités (de ciphertext-only à chosen-ciphertext), plus le modèle est puissant, et plus un algorithme résistant à un modèle fort est considéré robuste. On distingue aussi des variantes utiles en pratique :

- **Adaptive chosen-plaintext/ciphertext** : l'attaquant choisit ses requêtes successives en fonction des réponses déjà obtenues, ce qui est souvent bien plus puissant qu'un ensemble de requêtes fixé à l'avance (c'est le modèle réaliste des attaques par *oracle*, comme l'attaque par *padding oracle* étudiée au chapitre 6).
- **Related-key attack** : l'attaquant obtient des chiffrés produits sous plusieurs clés mathématiquement liées entre elles (par exemple différant d'un XOR connu). Ce modèle, moins réaliste dans l'absolu, s'avère pertinent contre certains protocoles où la dérivation de clé est mal conçue.

En pratique, le modèle *chosen-plaintext* (CPA) est devenu le standard minimal exigé pour un chiffrement moderne (sécurité IND-CPA), et le modèle *chosen-ciphertext* (CCA, en particulier CCA2 adaptatif) est requis pour les schémas de chiffrement à clé publique utilisés en pratique.

## 4. Classification des attaques selon la méthode

- **Attaques statistiques** : exploitation des propriétés statistiques du texte clair (fréquence des lettres, des mots) ou du chiffrement.
- **Attaques par force brute (exhaustive search)** : test systématique de toutes les clés possibles. Sa complexité croît exponentiellement avec la taille de la clé, ce qui en fait la référence de comparaison : toute attaque « intéressante » d'un point de vue cryptanalytique doit être **plus rapide** que la force brute pour être considérée comme une avancée.
- **Attaques structurelles/mathématiques** : exploitation d'une faiblesse mathématique de l'algorithme (factorisation, logarithme discret, biais différentiels ou linéaires).
- **Attaques par canaux auxiliaires (side-channel)** : exploitation d'informations physiques fuitées par l'implémentation (temps d'exécution, consommation électrique) plutôt que de l'algorithme lui-même.
- **Attaques par ingénierie sociale ou implémentation** : exploitation d'erreurs d'implémentation (générateur aléatoire faible, réutilisation de clé, mode opératoire mal choisi) plutôt que de l'algorithme mathématique — en pratique, la source la plus fréquente de compromission de systèmes cryptographiques réels.

### Complexité : quelques ordres de grandeur pour situer une attaque

Un réflexe essentiel en cryptanalyse est de savoir situer un nombre d'opérations sur une échelle réaliste. Le tableau suivant (purement illustratif, à but pédagogique) aide à interpréter des complexités exprimées en 2^n :

| Complexité | Ordre de grandeur | Interprétation qualitative |
|---|---|---|
| 2^20 ≈ 10^6 | environ un million | trivial, instantané sur un ordinateur personnel |
| 2^40 ≈ 10^12 | mille milliards | rapide sur une machine actuelle |
| 2^56 ≈ 7×10^16 | espace de clé de DES | atteignable avec du matériel dédié (illustre pourquoi DES est obsolète) |
| 2^80 | — | jugé insuffisant depuis longtemps comme marge de sécurité à long terme |
| 2^128 | espace de clé d'AES-128 | hors de portée de toute force brute envisageable avec la technologie actuelle et prévisible |

Ce tableau ne donne aucune estimation de temps ou de coût matériel précis (qui dépendent de paramètres technologiques évolutifs), mais illustre la **croissance exponentielle** qui fonde la sécurité par force brute : chaque bit de clé supplémentaire double le travail de l'attaquant.

## 5. Notion de sécurité et de coût d'une attaque

Un système est dit sûr non pas parce qu'il est mathématiquement incassable dans l'absolu, mais parce que le coût de la meilleure attaque connue (temps de calcul, mémoire, nombre de messages requis) dépasse très largement ce qu'un attaquant réaliste peut mobiliser. La sécurité est donc **relative et évolutive** : un algorithme sûr aujourd'hui peut devenir vulnérable avec de nouvelles techniques d'attaque ou une puissance de calcul accrue (ex. menace future de l'informatique quantique sur RSA, abordée au chapitre 5).

### Sécurité inconditionnelle contre sécurité calculatoire

Claude Shannon a formalisé en 1949 la notion de **sécurité parfaite (inconditionnelle)** : un système offre une sécurité parfaite si le texte chiffré ne révèle strictement aucune information sur le clair, quelle que soit la puissance de calcul de l'attaquant (c'est le cas du masque jetable — *one-time pad* — utilisé correctement). En pratique, cette sécurité inconditionnelle est rarement utilisable, car elle exige une clé aussi longue que le message, utilisée une seule fois, ce qui pose des problèmes de distribution insurmontables à grande échelle.

La quasi-totalité des systèmes cryptographiques modernes (AES, RSA, ECC) reposent donc sur une **sécurité calculatoire** : ils ne sont pas prouvés incassables, mais on estime que les casser demande des ressources de calcul irréalistes avec les meilleurs algorithmes connus à ce jour. C'est une distinction fondamentale à garder en tête tout au long de ce cours : la cryptanalyse ne cherche pas à démontrer l'impossibilité d'une attaque, mais à faire progresser (ou à borner) le coût de la meilleure attaque connue.

## 6. Méthodologie générale d'une cryptanalyse

1. Comprendre précisément l'algorithme et son mode d'utilisation (mode opératoire, gestion des clés).
2. Identifier le modèle d'attaque réaliste (quelles informations l'attaquant peut-il obtenir ?).
3. Rechercher des biais statistiques, structurels ou d'implémentation exploitables.
4. Évaluer le coût de l'attaque (complexité temporelle, données nécessaires) et le comparer à un budget attaquant réaliste.
5. Le cas échéant, implémenter une preuve de concept sur un exemple contrôlé.

### Illustration pédagogique : une attaque exhaustive triviale en Python

L'exemple suivant, volontairement simplifié, illustre les étapes 1, 2 et 5 de la méthodologie sur le chiffre de César (espace de clés minuscule : 26 valeurs). Il ne s'agit évidemment pas d'un algorithme réaliste, mais d'un support pour raisonner sur le **coût** d'une attaque exhaustive avant de l'appliquer, en apprentissage, à des espaces plus grands (chapitre 3).

```python
# Attaque exhaustive pédagogique contre le chiffre de César
# (à but strictement didactique : espace de clés = 26)

ALPHABET = "abcdefghijklmnopqrstuvwxyz"

def dechiffrer_cesar(texte_chiffre, decalage):
    resultat = []
    for c in texte_chiffre:
        if c.lower() in ALPHABET:
            idx = (ALPHABET.index(c.lower()) - decalage) % 26
            resultat.append(ALPHABET[idx])
        else:
            resultat.append(c)
    return "".join(resultat)

def attaque_exhaustive(texte_chiffre):
    """Étape 5 de la méthodologie : preuve de concept.
    On énumère les 26 clés possibles (étape 2 : modèle ciphertext-only,
    espace de clé trivial) et on affiche chaque hypothèse pour
    évaluation humaine (étape 4 : coût quasi nul ici)."""
    for cle in range(26):
        print(f"clé={cle:2d} -> {dechiffrer_cesar(texte_chiffre, cle)}")

# Exemple fictif : "khoor" est le chiffré de "hello" avec un décalage de 3
attaque_exhaustive("khoor")
```

Ce programme énumère les 26 hypothèses de clé, à charge pour l'analyste (ou un score de vraisemblance linguistique automatisé, voir chapitre 2) de reconnaître la bonne hypothèse. Le point pédagogique central est que **le coût de l'étape 5 dépend directement de la taille de l'espace de clé identifié à l'étape 2** : ici, 26 essais ; contre AES-128, 2^128 essais, ce qui change radicalement la faisabilité pratique de l'approche.

## À retenir

- La cryptanalyse et la cryptographie évoluent en miroir : un algorithme n'est validé que par une résistance prouvée à la cryptanalyse publique.
- Le modèle d'attaque (ciphertext-only à chosen-ciphertext, avec ou sans adaptativité) détermine la puissance réaliste de l'attaquant, et donc la portée réelle d'un résultat de cryptanalyse.
- Une cryptanalyse peut viser des objectifs de gravité très différente (distinction, déduction partielle, rupture totale) : un algorithme « cassé » académiquement n'est pas nécessairement dangereux en pratique.
- La sécurité cryptographique moderne est presque toujours **calculatoire**, non **inconditionnelle** : elle repose sur le coût prohibitif de la meilleure attaque connue, une notion relative et évolutive dans le temps.
- L'histoire (Enigma en particulier) montre qu'un espace de clés immense ne protège pas contre des faiblesses structurelles ou des habitudes opérationnelles prévisibles.
- En pratique, la majorité des compromissions viennent d'erreurs d'implémentation, pas de failles mathématiques dans l'algorithme lui-même.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
