# Chapitre 1 — Introduction à la cryptanalyse : histoire et classification des attaques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/01-introduction-cryptanalyse.txt){ target=_blank }

## 1. Définition

La cryptanalyse est l'étude des méthodes permettant de retrouver, en tout ou partie, l'information protégée par un système cryptographique (message clair, clé) sans disposer légitimement de la clé de déchiffrement. Elle est le pendant offensif de la cryptographie, et les deux disciplines progressent ensemble : un algorithme n'est considéré fiable qu'après avoir résisté longtemps à un examen cryptanalytique public et intensif (principe de Kerckhoffs, vu au module *Cryptographie*).

## 2. Repères historiques

- **Antiquité à Renaissance** : chiffrement de César, chiffres de substitution ; cassés par analyse fréquentielle dès le IXe siècle par le savant arabe Al-Kindi.
- **XVIe siècle** : chiffre de Vigenère, longtemps considéré incassable (« le chiffre indéchiffrable »), cassé au XIXe siècle par Kasiski et Babbage.
- **Seconde Guerre mondiale** : cryptanalyse de la machine Enigma par les équipes polonaises puis britanniques (Bletchley Park, Alan Turing) — un tournant pour la cryptanalyse moderne et la naissance de l'informatique.
- **Ère moderne** : cryptanalyse différentielle et linéaire contre les chiffrements par blocs (DES), attaques mathématiques contre RSA (factorisation), attaques par canaux auxiliaires.

## 3. Classification des attaques selon les informations disponibles

| Type d'attaque | Information disponible à l'attaquant |
|---|---|
| **Ciphertext-only** (texte chiffré seul) | uniquement des messages chiffrés |
| **Known-plaintext** (texte clair connu) | un ou plusieurs couples (clair, chiffré) |
| **Chosen-plaintext** (texte clair choisi) | l'attaquant peut faire chiffrer des textes de son choix |
| **Chosen-ciphertext** (texte chiffré choisi) | l'attaquant peut faire déchiffrer des textes chiffrés de son choix (hors le message cible) |

Plus l'attaquant dispose de capacités (de ciphertext-only à chosen-ciphertext), plus le modèle est puissant, et plus un algorithme résistant à un modèle fort est considéré robuste.

## 4. Classification des attaques selon la méthode

- **Attaques statistiques** : exploitation des propriétés statistiques du texte clair (fréquence des lettres, des mots) ou du chiffrement.
- **Attaques par force brute (exhaustive search)** : test systématique de toutes les clés possibles.
- **Attaques structurelles/mathématiques** : exploitation d'une faiblesse mathématique de l'algorithme (factorisation, logarithme discret, biais différentiels ou linéaires).
- **Attaques par canaux auxiliaires (side-channel)** : exploitation d'informations physiques fuitées par l'implémentation (temps d'exécution, consommation électrique) plutôt que de l'algorithme lui-même.
- **Attaques par ingénierie sociale ou implémentation** : exploitation d'erreurs d'implémentation (générateur aléatoire faible, réutilisation de clé, mode opératoire mal choisi) plutôt que de l'algorithme mathématique — en pratique, la source la plus fréquente de compromission de systèmes cryptographiques réels.

## 5. Notion de sécurité et de coût d'une attaque

Un système est dit sûr non pas parce qu'il est mathématiquement incassable dans l'absolu, mais parce que le coût de la meilleure attaque connue (temps de calcul, mémoire, nombre de messages requis) dépasse très largement ce qu'un attaquant réaliste peut mobiliser. La sécurité est donc **relative et évolutive** : un algorithme sûr aujourd'hui peut devenir vulnérable avec de nouvelles techniques d'attaque ou une puissance de calcul accrue (ex. menace future de l'informatique quantique sur RSA, abordée au chapitre 5).

## 6. Méthodologie générale d'une cryptanalyse

1. Comprendre précisément l'algorithme et son mode d'utilisation (mode opératoire, gestion des clés).
2. Identifier le modèle d'attaque réaliste (quelles informations l'attaquant peut-il obtenir ?).
3. Rechercher des biais statistiques, structurels ou d'implémentation exploitables.
4. Évaluer le coût de l'attaque (complexité temporelle, données nécessaires) et le comparer à un budget attaquant réaliste.
5. Le cas échéant, implémenter une preuve de concept sur un exemple contrôlé.

## À retenir

- La cryptanalyse et la cryptographie évoluent en miroir : un algorithme n'est validé que par une résistance prouvée à la cryptanalyse publique.
- Le modèle d'attaque (ciphertext-only à chosen-ciphertext) détermine la puissance réaliste de l'attaquant.
- En pratique, la majorité des compromissions viennent d'erreurs d'implémentation, pas de failles mathématiques dans l'algorithme lui-même.
