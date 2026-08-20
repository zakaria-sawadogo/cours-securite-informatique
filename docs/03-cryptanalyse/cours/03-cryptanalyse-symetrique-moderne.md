# Chapitre 3 — Cryptanalyse des chiffrements symétriques modernes

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/03-cryptanalyse-symetrique-moderne.txt){ target=_blank }

## 1. Attaque par force brute (exhaustive key search)

Pour un chiffrement par bloc à clé de *n* bits, l'attaque par force brute nécessite en moyenne 2^(n-1) essais. C'est pourquoi la taille de clé est le premier paramètre de sécurité : DES (56 bits) est aujourd'hui cassable par force brute en quelques heures avec du matériel spécialisé, ce qui l'a rendu obsolète au profit d'AES (128, 192 ou 256 bits), hors de portée de la force brute avec la technologie actuelle et prévisible.

## 2. Cryptanalyse différentielle

Introduite publiquement par Biham et Shamir (fin des années 1980) contre DES. Principe :

1. Choisir une paire de textes clairs présentant une différence fixe (souvent un XOR précis).
2. Chiffrer les deux textes et observer la différence entre les chiffrés correspondants.
3. Certaines différences de sortie apparaissent avec une probabilité plus élevée que si le chiffrement était parfaitement aléatoire : ce biais statistique permet de déduire de l'information sur des bits de la clé (ou de sous-clés de tour).
4. Répéter sur de nombreuses paires pour affiner la déduction.

DES s'est révélé partiellement vulnérable à cette technique mais sa conception (notamment le choix des boîtes-S) avait déjà anticipé et limité cette attaque — la NSA connaissait apparemment la cryptanalyse différentielle avant sa publication académique.

## 3. Cryptanalyse linéaire

Introduite par Matsui (1993). Principe : rechercher des approximations linéaires (combinaisons XOR de bits d'entrée, de sortie et de clé) vraies avec une probabilité significativement différente de 1/2. Une approximation suffisamment biaisée, combinée à un grand nombre de couples clair/chiffré connus, permet de retrouver des bits de clé par vote statistique (lemme du pilage/*piling-up lemma*).

## 4. Pourquoi AES résiste-t-il mieux ?

AES a été conçu (concours public NIST, choix de Rijndael en 2001) en intégrant explicitement des critères de résistance à la cryptanalyse différentielle et linéaire dès sa conception (structure des boîtes-S optimisée, nombre de tours dimensionné avec marge de sécurité). Aucune attaque académique connue à ce jour ne casse AES pleine puissance plus rapidement qu'une recherche exhaustive proche de l'optimum théorique.

## 5. Attaques structurelles sur les modes opératoires

Un algorithme de bloc robuste (AES) peut néanmoins être compromis par un **mode opératoire** mal choisi ou mal utilisé :

| Mode | Faiblesse exploitable |
|---|---|
| **ECB** (Electronic Codebook) | des blocs de clair identiques produisent des blocs chiffrés identiques → motifs visibles (voir TP3) |
| **CBC sans authentification** | vulnérable aux attaques par *padding oracle* si un serveur révèle si le padding est valide (voir TP3) |
| **Réutilisation de nonce/IV** (CTR, GCM) | annule la sécurité du chiffrement en flux résultant, permet de retrouver le XOR de deux messages clairs |

Ces attaques ne cassent pas l'algorithme de bloc lui-même : elles exploitent un mauvais usage, ce qui souligne l'importance, en cryptographie appliquée, de suivre des schémas éprouvés (AES-GCM, ChaCha20-Poly1305) plutôt que de composer soi-même bloc + mode.

## 6. Attaques par compromis temps-mémoire

Les **tables arc-en-ciel (rainbow tables)** illustrent un compromis entre temps de calcul et espace de stockage pour inverser une fonction à sens unique (utilisées historiquement contre des hachages de mots de passe non salés — approfondi au chapitre 4).

## À retenir

- La taille de clé fixe la borne théorique de résistance à la force brute ; DES est aujourd'hui cassable, AES ne l'est pas avec la technologie actuelle.
- Cryptanalyse différentielle et linéaire exploitent des biais statistiques internes à la structure de l'algorithme et ont directement influencé la conception d'AES.
- La grande majorité des compromissions pratiques de chiffrements symétriques modernes viennent d'un mauvais mode opératoire ou d'une mauvaise gestion des nonces/IV, pas d'une faille dans l'algorithme de bloc lui-même.
