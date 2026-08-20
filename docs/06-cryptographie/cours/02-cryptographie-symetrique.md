# Chapitre 2 — Cryptographie symétrique : chiffrement par bloc et par flux

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/02-cryptographie-symetrique.txt){ target=_blank }

## 1. Chiffrement par flux (stream cipher)

Combine le texte clair avec une suite pseudo-aléatoire (le « keystream ») générée à partir de la clé, généralement par XOR bit à bit :

```
chiffré[i] = clair[i] XOR keystream[i]
```

Exemple moderne largement utilisé : **ChaCha20**. Un chiffrement par flux exige impérativement de **ne jamais réutiliser le même couple (clé, nonce)** : réutiliser le même keystream sur deux messages différents permet de retrouver le XOR des deux messages clairs, une fuite d'information souvent exploitable (rappel du chapitre Cryptanalyse).

### 1.1 Pourquoi la réutilisation d'un keystream est catastrophique

Si un même keystream `K` est utilisé pour chiffrer deux messages clairs `m1` et `m2` :

```
c1 = m1 XOR K
c2 = m2 XOR K
c1 XOR c2 = m1 XOR m2   (le keystream s'annule)
```

L'attaquant obtient directement le XOR des deux messages clairs sans jamais connaître la clé. Si l'un des deux messages est connu, ou si les deux présentent une structure exploitable (langage naturel, en-têtes de fichier prévisibles), il devient souvent possible de reconstituer une grande partie du contenu des deux messages. C'est exactement le mécanisme illustré, à titre d'exemple pédagogique, par le TP de cryptanalyse sur la réutilisation de flux chiffrant.

### 1.2 ChaCha20 : structure en bref

ChaCha20 génère son keystream à partir d'un état interne de 16 mots de 32 bits (clé de 256 bits, nonce, compteur de bloc), transformé par une série de tours mêlant additions, XOR et rotations (opérations dites « ARX » : Add-Rotate-XOR). Cette conception, purement logicielle et sans table de substitution, le rend particulièrement rapide et résistant aux attaques par canal auxiliaire basées sur le temps d'accès mémoire (contrairement à certaines implémentations non protégées d'AES en logiciel, avant la généralisation du support matériel AES-NI).

## 2. Chiffrement par bloc (block cipher)

Chiffre des blocs de taille fixe (128 bits pour AES) en appliquant plusieurs « tours » de transformations combinant substitution et permutation.

### AES (Advanced Encryption Standard)

Standardisé par le NIST en 2001 (algorithme Rijndael), AES opère sur des blocs de 128 bits avec des clés de 128, 192 ou 256 bits, en 10, 12 ou 14 tours respectivement. Chaque tour comprend classiquement :

1. **SubBytes** : substitution non linéaire octet par octet (boîte-S), assure la confusion.
2. **ShiftRows** : décalage cyclique des lignes de l'état, contribue à la diffusion.
3. **MixColumns** : combinaison linéaire des colonnes (absente au dernier tour), renforce la diffusion.
4. **AddRoundKey** : XOR avec une sous-clé dérivée de la clé principale pour ce tour.

### 2.1 Cadencement de clé (key schedule)

AES ne réutilise pas directement la clé principale à chaque tour : un algorithme de **cadencement de clé** (*key schedule*) dérive, à partir de la clé initiale, une sous-clé distincte pour chaque tour (11 sous-clés pour AES-128, un tour supplémentaire de plus pour AES-192/256). Cette dérivation combine des rotations d'octets, des passages par la boîte-S, et des constantes de tour (*round constants*), afin que les sous-clés successives n'aient pas de relation linéaire simple entre elles — une relation trop simple faciliterait des attaques par cryptanalyse différentielle sur le cadencement lui-même.

### 2.2 AES-NI et implémentation matérielle

Les processeurs modernes (Intel, AMD, ARM) intègrent des instructions dédiées (AES-NI sur x86, instructions Crypto Extension sur ARM) qui exécutent directement les tours AES en matériel. Ceci apporte deux bénéfices simultanés : une exécution nettement plus rapide qu'une implémentation logicielle pure, et une résistance native aux attaques par canal auxiliaire temporel (l'accès aux boîtes-S ne dépend plus d'un tableau en mémoire dont le temps d'accès pourrait fuiter de l'information, comme cela pouvait être le cas pour certaines implémentations logicielles historiques d'AES).

```bash
# Vérifier la disponibilité d'AES-NI sur un système Linux
grep -o aes /proc/cpuinfo | head -n1

# Chiffrement/déchiffrement d'un fichier avec AES-256 en mode GCM via OpenSSL
openssl enc -aes-256-gcm -pbkdf2 -salt -in message.txt -out message.enc
openssl enc -aes-256-gcm -pbkdf2 -salt -d -in message.enc -out message_dechiffre.txt
```

## 3. Modes opératoires : pourquoi ils sont indispensables

Un chiffrement par bloc ne traite que des blocs de taille fixe : un **mode opératoire** définit comment enchaîner le chiffrement de plusieurs blocs pour traiter un message de longueur arbitraire.

| Mode | Principe | Points clés |
|---|---|---|
| **ECB** (Electronic Codebook) | chaque bloc chiffré indépendamment | **à proscrire** : des blocs clairs identiques donnent des blocs chiffrés identiques (motifs visibles, cf. TP3 du module Cryptanalyse) |
| **CBC** (Cipher Block Chaining) | chaque bloc clair est XORé avec le bloc chiffré précédent avant chiffrement ; nécessite un IV aléatoire | vulnérable aux attaques de padding oracle si le padding n'est pas vérifié de façon sûre |
| **CTR** (Counter) | transforme un chiffrement par bloc en chiffrement par flux, en chiffrant un compteur incrémental | parallélisable, mais un nonce réutilisé est aussi dangereux qu'en chiffrement par flux |
| **GCM** (Galois/Counter Mode) | mode CTR combiné à une authentification intégrée (AEAD) | **recommandé par défaut** : fournit confidentialité ET intégrité/authenticité en une seule primitive |

### 3.1 Le cas ECB en détail

En mode ECB, chaque bloc de 128 bits identique du texte clair produit, avec la même clé, exactement le même bloc chiffré. Sur des données présentant une forte structure répétitive (une image bitmap non compressée, par exemple), les motifs du clair restent visibles dans le chiffré malgré le chiffrement — un exemple pédagogique classique montre une image chiffrée en ECB où le contour du logo d'origine reste parfaitement reconnaissable. C'est une démonstration directe et intuitive du fait qu'un mode opératoire mal choisi peut annuler une grande partie de la sécurité apportée par l'algorithme de bloc sous-jacent, aussi robuste soit-il.

### 3.2 CBC et le padding

Comme AES traite des blocs de 128 bits (16 octets), un message dont la longueur n'est pas un multiple exact de 16 octets doit être complété (« bourrage », *padding* — par exemple selon le schéma PKCS#7) avant chiffrement en CBC. Au déchiffrement, le padding est retiré et sa validité vérifiée. Si le système signale de façon distinguable (message d'erreur différent, ou même simplement un temps de réponse différent) qu'un padding est invalide, un attaquant peut exploiter cette information pour déchiffrer progressivement un message intercepté sans jamais connaître la clé — c'est l'attaque dite du **padding oracle**, une des raisons majeures pour lesquelles CBC est aujourd'hui déconseillé au profit des modes AEAD.

### 3.3 CTR : parallélisation

Le mode CTR chiffre indépendamment un compteur (dérivé du nonce et d'un index de bloc croissant) pour produire un keystream, XORé ensuite avec le clair — exactement comme un chiffrement par flux construit à partir d'un chiffrement par bloc. Chaque bloc étant indépendant des autres (contrairement à CBC), le chiffrement et le déchiffrement peuvent être parallélisés sur plusieurs cœurs de calcul, un avantage important pour le débit sur de gros volumes de données.

## 4. AEAD : chiffrement authentifié

Un mode **AEAD** (Authenticated Encryption with Associated Data), comme AES-GCM ou ChaCha20-Poly1305, combine chiffrement et code d'authentification en une seule opération, produisant un **tag d'authentification** vérifié au déchiffrement. Si le texte chiffré a été modifié (accidentellement ou par un attaquant), le déchiffrement échoue explicitement plutôt que de produire silencieusement un texte clair corrompu ou manipulé.

**Recommandation pratique actuelle** : privilégier systématiquement un mode AEAD (AES-GCM, ChaCha20-Poly1305) plutôt que de composer soi-même un mode CBC avec un HMAC séparé — une composition manuelle est une source fréquente d'erreurs d'implémentation.

### 4.1 Données associées authentifiées (AAD)

Un mode AEAD permet en plus d'authentifier des **données associées** (*Associated Data*) qui ne sont pas chiffrées mais dont l'intégrité doit être garantie conjointement au texte chiffré — par exemple un en-tête de paquet réseau en clair (adresse, numéro de séquence) qui doit rester lisible en transit mais ne doit pas pouvoir être altéré sans que la vérification échoue. C'est le rôle du paramètre `AAD` dans l'API GCM : il est intégré au calcul du tag d'authentification mais n'affecte pas le texte chiffré lui-même.

### 4.2 Structure interne de GCM (aperçu)

GCM combine un chiffrement en mode CTR (pour la confidentialité) avec une fonction d'authentification universelle appelée **GHASH**, construite sur une multiplication dans le corps fini `GF(2^128)`. Le tag d'authentification résulte de ce calcul combinant texte chiffré, données associées et une sous-clé dérivée de la clé de chiffrement. Un point de vigilance technique important : la sécurité de GCM se dégrade fortement si le même couple (clé, nonce) est réutilisé, car cela permet potentiellement à un attaquant de retrouver la sous-clé d'authentification et de forger des tags valides pour d'autres messages — encore une illustration de la criticité de l'unicité du nonce (section 5).

```bash
# Illustration : chiffrement authentifié avec ChaCha20-Poly1305 (AEAD) via OpenSSL
openssl enc -chacha20-poly1305 -pbkdf2 -salt -in message.txt -out message.enc
```

## 5. Gestion de la clé et du vecteur d'initialisation (IV)/nonce

- La **clé** doit être générée par un générateur aléatoire cryptographiquement sûr (CSPRNG) et jamais codée en dur dans le code source.
- L'**IV/nonce** doit être unique pour chaque chiffrement avec une même clé (aléatoire pour CBC, souvent un compteur pour CTR/GCM) — sa réutilisation est l'une des erreurs d'implémentation les plus dommageables en pratique.

### 5.1 Nonce aléatoire vs nonce séquentiel

Deux stratégies courantes pour garantir l'unicité d'un nonce :

- **Nonce aléatoire** (généré par un CSPRNG à chaque chiffrement) : simple à mettre en œuvre côté client sans état persistant, mais expose à un risque de collision si l'espace de nonces est trop petit par rapport au nombre de messages chiffrés (paradoxe des anniversaires, rappel du module *Cryptanalyse*) — pour GCM, la taille de nonce standard de 96 bits est conçue pour limiter ce risque tant qu'un volume raisonnable de messages est chiffré sous une même clé.
- **Nonce séquentiel** (compteur incrémental, souvent utilisé côté serveur avec état) : garantit l'unicité de façon déterministe tant que l'état du compteur est correctement persisté et jamais remis à zéro par erreur (par exemple après un redémarrage mal géré) — une remise à zéro accidentelle du compteur avec réutilisation de la même clé reproduit exactement le scénario dangereux décrit en section 1.1.

### 5.2 Rotation des clés

Aucune clé ne doit être utilisée indéfiniment : au-delà d'un certain volume de données chiffrées sous une même clé, la probabilité de collision de nonce ou d'exposition accrue à une éventuelle faiblesse cryptanalytique augmente. Les architectures robustes prévoient une politique de **rotation périodique des clés** (génération d'une nouvelle clé, réencodage ou transition progressive), combinée à un mécanisme d'identification de version de clé (souvent un identifiant de clé, ou *key ID*, associé aux données chiffrées) permettant de savoir avec quelle clé déchiffrer des données plus anciennes.

## 6. Dérivation de clé à partir d'un mot de passe

Un mot de passe humain n'a pas l'entropie suffisante pour servir directement de clé de chiffrement : on utilise une **fonction de dérivation de clé (KDF)** dédiée (PBKDF2, scrypt, Argon2), volontairement coûteuse en calcul, pour ralentir les attaques par force brute (rappel du chapitre Cryptanalyse sur le hachage de mots de passe).

### 6.1 Paramètres d'une KDF moderne

Une KDF moderne comme Argon2 expose plusieurs paramètres de coût, qu'il faut choisir en fonction du contexte (matériel disponible, contrainte de latence acceptable) :

- **Coût en temps** (nombre d'itérations internes) ;
- **Coût en mémoire** (quantité de mémoire vive requise pour le calcul, ce qui pénalise spécifiquement les attaques massivement parallélisées sur GPU/ASIC, moins efficaces sur des calculs gourmands en mémoire) ;
- **Parallélisme** (nombre de threads pouvant être utilisés).

```bash
# Dérivation d'une clé de chiffrement à partir d'un mot de passe (illustration OpenSSL, PBKDF2)
openssl enc -aes-256-gcm -pbkdf2 -iter 600000 -salt -in message.txt -out message.enc
```

Le **sel** (valeur aléatoire propre à chaque dérivation) est indispensable : sans lui, deux utilisateurs partageant le même mot de passe produiraient la même clé dérivée, et un attaquant pourrait précalculer une table de correspondances mot de passe → clé (attaque par table arc-en-ciel, détaillée au module *Cryptanalyse*).

## 7. Pièges d'implémentation courants

- **Réutiliser un IV/nonce** avec la même clé, en mode CBC comme en mode CTR/GCM — l'erreur la plus fréquente et la plus dommageable.
- **Choisir ECB par défaut** parce que c'est le mode le plus simple à utiliser dans certaines API historiques, sans en connaître les conséquences.
- **Ignorer l'échec de vérification du tag AEAD** au déchiffrement (par exemple continuer à traiter des données malgré une exception d'authentification non gérée) — cela annule totalement la protection d'intégrité apportée par l'AEAD.
- **Stocker la clé et les données chiffrées au même endroit** sans séparation des privilèges d'accès, réduisant le chiffrement à une protection purement cosmétique en cas de compromission du système de stockage.

## À retenir

- AES est le standard de chiffrement par bloc de référence ; sa sécurité dépend fortement du mode opératoire choisi, pas seulement de l'algorithme lui-même.
- ECB est à proscrire ; les modes AEAD (GCM, ChaCha20-Poly1305) sont recommandés par défaut car ils apportent confidentialité et intégrité simultanément, avec authentification des données associées (AAD).
- La réutilisation d'un nonce/IV avec la même clé est l'une des erreurs d'implémentation les plus graves en cryptographie symétrique, qu'il s'agisse d'un chiffrement par flux ou d'un mode CTR/GCM.
- Une KDF (Argon2, scrypt, PBKDF2) est indispensable pour dériver une clé à partir d'un mot de passe ; le coût mémoire d'Argon2 le rend particulièrement résistant aux attaques matérielles massivement parallèles.
- La gestion opérationnelle des clés (génération sûre, rotation, séparation du stockage) est aussi déterminante pour la sécurité globale que le choix de l'algorithme.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
