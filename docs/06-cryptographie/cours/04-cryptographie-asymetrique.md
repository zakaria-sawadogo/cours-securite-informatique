# Chapitre 4 — Cryptographie asymétrique : RSA, Diffie-Hellman, ECC

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/04-cryptographie-asymetrique.txt){ target=_blank }

## 1. Principe général

Chaque partie dispose d'une paire de clés mathématiquement liées : une **clé publique**, diffusable librement, et une **clé privée**, gardée strictement secrète. Deux usages principaux : chiffrer avec la clé publique du destinataire (lui seul peut déchiffrer avec sa clé privée), ou signer avec sa propre clé privée (tout le monde peut vérifier avec la clé publique correspondante — chapitre 5).

Cette asymétrie fondamentale repose sur la notion de **fonction à sens unique avec trappe** (*trapdoor one-way function*) : une fonction facile à calculer dans un sens, difficile à inverser sans une information secrète particulière (la trappe), mais facile à inverser avec cette information. Multiplier deux grands nombres premiers est facile ; retrouver les deux facteurs à partir du produit est difficile — sauf à connaître déjà l'un des deux facteurs (RSA). Calculer `g^a mod p` est facile ; retrouver `a` à partir de `g^a mod p` est difficile (Diffie-Hellman, ECC).

## 2. Rappels d'arithmétique modulaire nécessaires

- **Congruence modulo n** : `a ≡ b (mod n)` si `a` et `b` ont le même reste dans la division par `n`.
- **Exponentiation modulaire** : `a^e mod n`, calculable efficacement par l'algorithme d'exponentiation rapide (« square and multiply »), sans jamais calculer `a^e` en entier.
- **Petit théorème de Fermat / théorème d'Euler** : fondement de la construction de RSA.
- **Inverse modulaire** : `d` tel que `e × d ≡ 1 (mod φ(n))`, calculable par l'algorithme d'Euclide étendu — c'est ainsi que la clé privée RSA est dérivée de la clé publique et des facteurs premiers.

### 2.1 Exponentiation rapide : illustration simplifiée

Calculer `a^13 mod n` naïvement demanderait 12 multiplications successives. L'algorithme « square and multiply » exploite l'écriture binaire de l'exposant (`13 = 1101` en binaire) pour ne réaliser qu'un nombre de multiplications proportionnel au nombre de bits de l'exposant :

```
13 en binaire : 1101
a^13 = a^8 × a^4 × a^1
     = ((a^2)^2)^2 × (a^2)^2 × a
```

Ce principe, appliqué à des exposants de plusieurs centaines de chiffres, est ce qui rend RSA praticable en temps raisonnable malgré la taille considérable des nombres manipulés.

### 2.2 Un exemple arithmétique jouet (à but pédagogique uniquement, tailles non sécurisées)

À titre purement illustratif — ces tailles de nombres sont bien trop petites pour un usage réel, elles serviraient à un adversaire disposant d'un simple crayon :

```
p = 61, q = 53
n = p × q = 3233
φ(n) = (p-1)(q-1) = 60 × 52 = 3120
e = 17            (choisi premier avec 3120)
d = 2753          (inverse modulaire de 17 modulo 3120)

Chiffrement d'un message m = 65 :
c = 65^17 mod 3233 = 2790

Déchiffrement :
m = 2790^2753 mod 3233 = 65
```

Cet exemple (classique dans la littérature pédagogique sur RSA) illustre la mécanique exacte de l'algorithme sur des nombres suffisamment petits pour être vérifiables, mais il ne doit jamais laisser penser que des tailles de cet ordre seraient utilisables en pratique : la sécurité de RSA repose sur l'usage de nombres premiers `p` et `q` de plusieurs centaines de chiffres.

## 3. RSA : construction

1. Choisir deux grands nombres premiers distincts `p` et `q` ; calculer `n = p × q`.
2. Calculer `φ(n) = (p-1)(q-1)` (indicatrice d'Euler).
3. Choisir un exposant public `e` premier avec `φ(n)` (souvent 65537).
4. Calculer l'exposant privé `d`, inverse modulaire de `e` modulo `φ(n)`.
5. Clé publique : `(n, e)`. Clé privée : `(n, d)`.

**Chiffrement** : `c = m^e mod n`. **Déchiffrement** : `m = c^d mod n`.

La sécurité repose sur la difficulté de retrouver `d` (ou de factoriser `n`) sans connaître `p` et `q` (rappel : chapitre 5 du module *Cryptanalyse*).

### 3.1 Pourquoi 65537 ?

L'exposant public 65537 (soit `2^16 + 1`) est un choix très répandu car il combine deux avantages : son écriture binaire (`10000000000000001`) ne comporte que deux bits à 1, ce qui rend l'exponentiation modulaire `m^e mod n` particulièrement rapide au chiffrement ou à la vérification de signature (peu de multiplications nécessaires), tout en étant suffisamment grand pour éviter certaines attaques structurelles connues sur RSA lorsque l'exposant public est trop petit (comme `e = 3` combiné à un padding insuffisant).

### Padding : une nécessité, pas une option

RSA « brut » (sans padding) est déterministe et vulnérable à plusieurs attaques structurelles (cf. module Cryptanalyse). Les standards actuels imposent un schéma de padding : **OAEP** pour le chiffrement, **PSS** pour la signature (chapitre 5). Utiliser RSA sans padding correct est une erreur d'implémentation grave, quelle que soit la taille de clé.

### 3.2 OAEP en bref

**OAEP** (Optimal Asymmetric Encryption Padding) introduit de l'aléa dans le message avant chiffrement RSA, de sorte que chiffrer deux fois le même message clair produise deux textes chiffrés différents (propriété dite de **sécurité sémantique**, absente de RSA brut qui est déterministe). OAEP combine le message avec une valeur aléatoire à travers une construction en deux tours utilisant des fonctions de hachage, rendant également le schéma résistant à des attaques par texte chiffré choisi lorsqu'il est correctement implémenté.

```bash
# Génération d'une paire de clés RSA (2048 bits) et chiffrement avec padding OAEP via OpenSSL
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out rsa_prive.pem
openssl pkey -in rsa_prive.pem -pubout -out rsa_publique.pem

openssl pkeyutl -encrypt -pubin -inkey rsa_publique.pem \
    -pkeyopt rsa_padding_mode:oaep -in message.txt -out message.enc

openssl pkeyutl -decrypt -inkey rsa_prive.pem \
    -pkeyopt rsa_padding_mode:oaep -in message.enc -out message_dechiffre.txt
```

## 4. Échange de clé Diffie-Hellman

Permet à deux parties d'établir un secret partagé sur un canal public, sans jamais transmettre ce secret directement :

1. Paramètres publics : un nombre premier `p` et un générateur `g`.
2. Alice choisit un secret `a`, calcule et envoie `A = g^a mod p`.
3. Bob choisit un secret `b`, calcule et envoie `B = g^b mod p`.
4. Alice calcule `B^a mod p`, Bob calcule `A^b mod p` : les deux obtiennent la même valeur `g^(ab) mod p`, le secret partagé.

Un observateur du canal voit `p`, `g`, `A` et `B`, mais retrouver `a` ou `b` (ou directement `g^(ab)`) requiert de résoudre le problème du logarithme discret, réputé difficile pour des paramètres suffisamment grands.

**Limite importante** : Diffie-Hellman « pur » ne fournit aucune authentification — il est vulnérable à une attaque de l'homme du milieu (*man-in-the-middle*) si les parties ne vérifient pas par ailleurs l'identité de leur interlocuteur (rôle des certificats et signatures, chapitre 5).

### 4.1 Exemple arithmétique jouet

```
Paramètres publics : p = 23, g = 5

Alice choisit a = 6  → A = 5^6 mod 23 = 8      (envoyé à Bob)
Bob choisit   b = 15 → B = 5^15 mod 23 = 19    (envoyé à Alice)

Alice calcule : B^a mod p = 19^6 mod 23 = 2
Bob calcule    : A^b mod p = 8^15 mod 23 = 2

Secret partagé obtenu par les deux parties : 2
```

Comme pour RSA, ces tailles sont purement pédagogiques : les déploiements réels utilisent des groupes de plusieurs centaines à plusieurs milliers de bits pour le Diffie-Hellman classique, ou des courbes elliptiques normalisées pour ECDH (section 5).

### 4.2 Diffie-Hellman éphémère (DHE / ECDHE)

Dans une variante dite **éphémère**, chaque partie génère une nouvelle paire `(a, A)` ou `(b, B)` pour chaque session, puis la détruit une fois le secret partagé calculé, au lieu de réutiliser des paramètres fixes sur le long terme. C'est ce mécanisme qui fournit la propriété de **confidentialité persistante** (*Perfect Forward Secrecy*), détaillée au chapitre 6 : même si la clé privée long terme d'un serveur est compromise ultérieurement, les secrets de session éphémères passés, déjà détruits, ne peuvent pas être reconstitués rétroactivement.

## 5. Cryptographie sur courbes elliptiques (ECC)

Remplace le groupe multiplicatif modulo `p` par le groupe des points d'une courbe elliptique définie sur un corps fini. Le problème équivalent au logarithme discret (ECDLP) y est considéré plus difficile à taille de clé égale, permettant des clés bien plus courtes pour un niveau de sécurité équivalent (rappel chapitre 5 du module *Cryptanalyse* : ECC 256 bits ≈ RSA 3072 bits). Courbes largement utilisées aujourd'hui : P-256 (NIST), Curve25519 (utilisée par X25519 pour l'échange de clé et Ed25519 pour la signature, chapitre 5).

### 5.1 Intuition géométrique

Une courbe elliptique est définie par une équation de la forme `y² = x³ + ax + b`, et l'on définit une opération d'« addition » entre deux points de la courbe, géométriquement construite en traçant une droite passant par les deux points et en réfléchissant son troisième point d'intersection avec la courbe par rapport à l'axe des abscisses. Cette opération d'addition dote l'ensemble des points de la courbe (plus un point « à l'infini » servant d'élément neutre) d'une structure de groupe mathématique. La « multiplication scalaire » `k × P` (additionner un point `P` à lui-même `k` fois, calculée efficacement par une méthode analogue au « square and multiply ») joue, dans ce groupe, le rôle que joue l'exponentiation modulaire dans Diffie-Hellman classique — d'où le nom d'**ECDH** (Elliptic Curve Diffie-Hellman) pour l'échange de clé sur courbe elliptique, qui suit exactement le même schéma que la section 4, en remplaçant `g^a mod p` par `a × G` (`G` étant un point générateur public fixé par la courbe).

### 5.2 Pourquoi des clés plus courtes suffisent

La meilleure attaque connue contre le logarithme discret classique (modulo `p`) exploite des algorithmes sous-exponentiels (comme le crible algébrique généralisé, utilisé aussi pour factoriser un module RSA), ce qui oblige à utiliser des paramètres numériquement très grands pour atteindre un niveau de sécurité donné. Sur une courbe elliptique bien choisie, en revanche, la meilleure attaque générique connue (méthode « rho » de Pollard adaptée) reste pleinement exponentielle en fonction de la taille de la clé, sans raccourci sous-exponentiel connu — d'où des tailles de clé nettement plus courtes pour un niveau de sécurité équivalent, comme rappelé dans le tableau du chapitre 1.

```bash
# Génération d'une paire de clés ECC (courbe P-256) avec OpenSSL
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out ec_prive.pem
openssl pkey -in ec_prive.pem -pubout -out ec_publique.pem

# Génération d'une paire de clés X25519 (échange de clé moderne)
openssl genpkey -algorithm X25519 -out x25519_prive.pem
```

### 5.3 Un piège classique : le choix de la courbe

Toutes les courbes elliptiques ne se valent pas. Certaines courbes historiques présentent des propriétés qui facilitent des attaques (courbes dites « anormales », ou choix de paramètres mal justifiés publiquement). C'est pourquoi il est recommandé de n'utiliser que des courbes standardisées et largement examinées par la communauté (P-256/P-384 du NIST, Curve25519/Curve448), plutôt que des paramètres de courbe choisis sans justification publique claire — un rappel direct du principe de Kerckhoffs (chapitre 1) appliqué au choix des paramètres, pas seulement de l'algorithme.

## 6. Chiffrement hybride

En pratique, la cryptographie asymétrique n'est presque jamais utilisée pour chiffrer directement de gros volumes de données (trop lente) : elle sert à établir ou transmettre une **clé de session**, ensuite utilisée avec un chiffrement symétrique rapide (AES-GCM). C'est le principe exact mis en œuvre par TLS (chapitre 6).

### 6.1 Deux façons d'obtenir la clé de session

- **Transport de clé** (schéma historique, ex. RSA) : l'émetteur génère lui-même une clé symétrique aléatoire, puis la chiffre avec la clé publique RSA du destinataire. Inconvénient majeur : la compromission ultérieure de la clé privée RSA permet de déchiffrer rétroactivement toutes les sessions passées enregistrées, car la même clé privée sert à toutes les sessions.
- **Accord de clé** (schéma recommandé aujourd'hui, ex. ECDHE) : les deux parties calculent conjointement un secret partagé via un échange Diffie-Hellman éphémère, sans jamais transmettre la clé de session elle-même sur le canal. Combiné à l'usage de paramètres éphémères (section 4.2), ce schéma apporte la confidentialité persistante, absente du transport de clé RSA classique — l'une des raisons pour lesquelles TLS 1.3 a supprimé le transport de clé RSA (chapitre 6).

## À retenir

- RSA repose sur la difficulté de factorisation ; Diffie-Hellman et ECC reposent sur la difficulté du logarithme discret (classique ou sur courbe elliptique).
- Le padding (OAEP/PSS) n'est pas optionnel : RSA brut est déterministe et vulnérable à des attaques structurelles connues ; OAEP apporte l'aléa nécessaire à la sécurité sémantique.
- Diffie-Hellman pur n'authentifie pas les parties ; il doit être combiné à un mécanisme d'authentification pour résister à une attaque de l'homme du milieu.
- La variante éphémère de Diffie-Hellman (DHE/ECDHE) apporte la confidentialité persistante, absente du transport de clé RSA classique.
- ECC atteint un niveau de sécurité équivalent à RSA avec des clés beaucoup plus courtes, grâce à l'absence de raccourci sous-exponentiel connu contre le logarithme discret sur courbe elliptique bien choisie.
- Le choix des paramètres (courbe elliptique, groupe Diffie-Hellman) doit toujours s'appuyer sur des standards publiquement examinés, jamais sur des paramètres non justifiés.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
