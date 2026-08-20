# Chapitre 3 — Fonctions de hachage et authentification de message (MAC/HMAC)

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/03-hachage-mac-hmac.txt){ target=_blank }

## 1. Fonctions de hachage cryptographiques

Une fonction de hachage `H` transforme une entrée de taille arbitraire en une sortie de taille fixe (l'empreinte ou *digest*), avec trois propriétés attendues (détaillées côté attaque au chapitre 4 du module *Cryptanalyse*) :

- résistance à la préimage,
- résistance à la seconde préimage,
- résistance aux collisions.

### 1.1 Reformulation intuitive des trois propriétés

- **Résistance à la préimage** : étant donné une empreinte `h`, il doit être impraticable de retrouver une entrée `m` telle que `H(m) = h`. C'est cette propriété qui permet, par exemple, de stocker une empreinte de mot de passe plutôt que le mot de passe lui-même.
- **Résistance à la seconde préimage** : étant donné une entrée `m1`, il doit être impraticable de trouver une entrée différente `m2` telle que `H(m1) = H(m2)`. Cette propriété protège contre la substitution d'un document par un autre ayant la même empreinte.
- **Résistance aux collisions** : il doit être impraticable de trouver **une paire quelconque** `(m1, m2)` telle que `H(m1) = H(m2)`, sans contrainte sur `m1` au départ. C'est la propriété la plus difficile à garantir, car le paradoxe des anniversaires (module *Cryptanalyse*) réduit la complexité théorique d'une recherche de collision à environ la racine carrée de l'espace de sortie, plutôt que sa totalité.

### 1.2 Panorama des fonctions usuelles

| Fonction | Taille de sortie | Statut |
|---|---|---|
| MD5 | 128 bits | **cassée** (collisions pratiques), à ne plus utiliser pour un usage sécuritaire |
| SHA-1 | 160 bits | **cassée** (collision démontrée en 2017), dépréciée |
| SHA-2 (SHA-256, SHA-384, SHA-512) | 256 à 512 bits | recommandée, aucune attaque pratique connue |
| SHA-3 (Keccak) | 224 à 512 bits | recommandée, construction interne différente de SHA-2 (fonction éponge), diversifie le risque en cas de faille future sur la famille SHA-2 |

### 1.3 Construction de Merkle-Damgård (SHA-2) vs construction en éponge (SHA-3)

SHA-2 repose sur une **construction de Merkle-Damgård** : le message est découpé en blocs successifs, chacun étant combiné à l'état interne via une fonction de compression, l'état final constituant l'empreinte. Cette construction, bien comprise et éprouvée, présente néanmoins une propriété structurelle exploitable dans certains contextes : l'**attaque par extension de longueur** (*length extension attack*), qui permet, connaissant `H(m)` sans connaître `m`, de calculer `H(m || padding || m')` pour un suffixe `m'` choisi par l'attaquant, sans connaître `m` en entier. C'est précisément ce qui rend dangereuse une authentification naïve construite comme `H(clé || message)` (voir section 4).

SHA-3 (Keccak) repose sur une **construction en éponge** (*sponge construction*), structurellement différente : les données sont absorbées puis « pressées » à travers une permutation interne, sans exposer directement un état intermédiaire réutilisable de la même manière. SHA-3 n'est donc pas vulnérable à l'attaque par extension de longueur, ce qui en fait une diversification utile, indépendamment de tout doute sur SHA-2 aujourd'hui.

```bash
# Calcul d'empreintes avec OpenSSL, comparaison de plusieurs fonctions
echo -n "cours de cryptographie" | openssl dgst -sha256
echo -n "cours de cryptographie" | openssl dgst -sha3-256
echo -n "cours de cryptographie" | openssl dgst -md5      # à des fins de comparaison uniquement, jamais en production
```

## 2. Usages des fonctions de hachage

- **Vérification d'intégrité** : comparer l'empreinte d'un fichier téléchargé à une valeur de référence publiée.
- **Structures de données** : arbres de Merkle (utilisés en Blockchain, module suivant), tables de hachage.
- **Brique de construction** : dérivation de clé, génération de nombres pseudo-aléatoires, signatures numériques (chapitre 5), preuve de travail.
- **Hachage de mots de passe** : nécessite des fonctions dédiées et volontairement lentes (bcrypt, scrypt, Argon2), jamais une fonction générique seule (cf. module Cryptanalyse, chapitre 4).

### 2.1 Vérification d'intégrité en pratique

```bash
# Publication d'une empreinte de référence pour un fichier distribué
sha256sum archive.tar.gz > archive.tar.gz.sha256

# Vérification côté utilisateur après téléchargement
sha256sum -c archive.tar.gz.sha256
```

Ce mécanisme ne protège que contre une **altération accidentelle** (erreur de transmission) si l'empreinte de référence est publiée sur le même canal, non authentifié, que le fichier lui-même : un attaquant capable de modifier le fichier sur le serveur de distribution peut tout aussi bien modifier l'empreinte publiée en conséquence. Pour se protéger contre une altération **malveillante**, il faut authentifier l'empreinte elle-même — par exemple via une signature numérique (chapitre 5) ou en la publiant sur un canal distinct et de confiance.

### 2.2 Arbres de Merkle : principe

Un arbre de Merkle organise un grand ensemble de données en une structure arborescente où chaque nœud interne contient le hachage de la concaténation de ses deux enfants, jusqu'à une **racine de Merkle** unique qui résume l'ensemble de la structure. Toute modification, même d'une seule feuille, change la racine. Cette structure permet de vérifier efficacement qu'une donnée particulière appartient bien à un ensemble représenté par une racine connue, sans avoir à transmettre ni vérifier l'ensemble des données : il suffit d'un « chemin de preuve » de taille logarithmique par rapport au nombre total de feuilles. Ce principe est repris intensivement en Blockchain (module suivant) et dans les registres de transparence de certificats (Certificate Transparency, chapitre 5).

## 3. Code d'authentification de message (MAC)

Un MAC garantit à la fois **l'intégrité** d'un message et **l'authenticité** de son origine, pour deux parties partageant une clé secrète : `MAC = Fk(message)`, où seule une partie possédant la clé `k` peut produire ou vérifier un MAC valide.

À la différence d'une signature numérique (chapitre 5), un MAC ne fournit pas la non-répudiation : toute partie capable de vérifier le MAC est également capable d'en produire un (elle possède la même clé).

### 3.1 Vérification : comparaison en temps constant

Vérifier un MAC consiste à recalculer la valeur attendue et à la comparer à celle reçue. Une comparaison naïve, octet par octet, qui s'arrête dès le premier octet différent, laisse fuiter une information exploitable via le temps d'exécution : un attaquant peut, message après message, deviner le MAC correct un octet à la fois en mesurant de minuscules différences de temps de réponse (**attaque temporelle**, ou *timing attack*). Les bibliothèques cryptographiques fournissent pour cette raison des fonctions de comparaison en **temps constant** (par exemple `CRYPTO_memcmp` dans OpenSSL, ou `hmac.compare_digest` en Python), qui doivent systématiquement être utilisées à la place d'une comparaison directe (`==` ou `memcmp`) pour tout secret cryptographique.

## 4. HMAC (Hash-based MAC)

Construction générique permettant de bâtir un MAC à partir de n'importe quelle fonction de hachage :

```
HMAC(k, m) = H( (k' XOR opad) || H( (k' XOR ipad) || m ) )
```

où `k'` est la clé complétée à la taille de bloc de `H`, et `ipad`/`opad` des constantes fixes. Cette double application de `H`, avec la clé combinée de part et d'autre, neutralise spécifiquement l'attaque par extension de longueur qui rend dangereuse une simple concaténation `H(clé || message)` (vue au chapitre 4 du module *Cryptanalyse*).

HMAC-SHA256 est aujourd'hui l'un des mécanismes de MAC les plus utilisés (authentification d'API, jetons JWT signés en HS256, intégrité de sessions).

### 4.1 Pourquoi `H(clé || message)` seul est dangereux

Avec une construction Merkle-Damgård (SHA-2), connaître `H(clé || message)` permet à un attaquant, même sans connaître la clé ni le message, de calculer `H(clé || message || padding || suffixe)` pour un suffixe de son choix : il lui suffit de reprendre l'état interne révélé par l'empreinte connue et de continuer le calcul de hachage à partir de ce point, en profitant du padding déterministe de la construction. Concrètement, cela permettrait à un attaquant de forger un MAC valide pour un message différent du message original, en y ajoutant des données arbitraires à la fin, sans jamais connaître la clé secrète — un exemple documenté de ce type de faille a affecté des API web mal conçues utilisant un simple `H(clé || message)` comme mécanisme d'authentification de requêtes. HMAC évite ce piège précisément parce que la clé intervient à **deux endroits distincts et imbriqués** du calcul (via `ipad` et `opad`), rompant la propriété exploitée par l'attaque par extension de longueur.

```bash
# Calcul d'un HMAC-SHA256 avec OpenSSL
echo -n "message a authentifier" | openssl dgst -sha256 -hmac "cle_secrete_partagee"
```

### 4.2 HMAC et dérivation de clé (HKDF)

HMAC ne sert pas uniquement à authentifier des messages : il constitue également le cœur de **HKDF** (HMAC-based Key Derivation Function), une fonction de dérivation de clé standardisée utilisée notamment dans TLS 1.3 (chapitre 6) pour dériver, à partir d'un secret partagé issu d'un échange Diffie-Hellman, plusieurs clés symétriques indépendantes (clé de chiffrement, clé de MAC, etc.) de façon sûre et reproductible des deux côtés de la communication.

## 5. MAC intégré : GMAC et Poly1305

Les modes AEAD vus au chapitre précédent (AES-GCM, ChaCha20-Poly1305) intègrent directement un mécanisme de MAC (GMAC pour GCM, Poly1305 pour ChaCha20) dans l'opération de chiffrement elle-même, évitant d'avoir à composer manuellement chiffrement et authentification — une composition manuelle mal ordonnée (ex. authentifier avant de chiffrer plutôt que l'inverse) est une source d'erreurs classique.

### 5.1 Poly1305 : un MAC à usage unique par message

Poly1305 est un MAC basé sur l'évaluation d'un polynôme dans un corps fini de grande taille (nombre premier proche de `2^130`), combiné à une clé à usage unique dérivée pour chaque message via ChaCha20. Sa conception, volontairement simple et rapide, offre une sécurité prouvable sous l'hypothèse que la clé à usage unique n'est effectivement jamais réutilisée pour deux messages différents — d'où l'importance, une fois de plus, de l'unicité stricte du nonce dans ChaCha20-Poly1305.

## 6. Ordre chiffrement/authentification (Encrypt-then-MAC)

Lorsqu'un schéma AEAD intégré n'est pas utilisé et qu'il faut composer soi-même chiffrement et MAC, la construction recommandée est **Encrypt-then-MAC** (chiffrer d'abord, puis authentifier le texte chiffré) plutôt que MAC-then-Encrypt ou Encrypt-and-MAC, car elle permet de rejeter un texte chiffré invalide avant même de tenter de le déchiffrer, limitant la surface d'attaques par oracle (padding oracle notamment).

### 6.1 Comparaison des trois constructions

| Construction | Définition | Risque principal |
|---|---|---|
| **Encrypt-and-MAC** | `MAC` calculé sur le clair, transmis à côté du chiffré | le MAC peut fuiter de l'information sur le clair, indépendamment du chiffrement |
| **MAC-then-Encrypt** | `MAC` calculé sur le clair, puis l'ensemble (message + MAC) est chiffré | le déchiffrement doit être tenté avant toute vérification, exposant à des attaques de type padding oracle si un mode par bloc avec padding est utilisé |
| **Encrypt-then-MAC** | le clair est chiffré, puis le MAC est calculé sur le texte chiffré | **recommandé** : le MAC est vérifié avant tout déchiffrement, un texte chiffré invalide est rejeté immédiatement |

Les modes AEAD (section 5) implémentent en réalité une forme optimisée et intégrée d'Encrypt-then-MAC, ce qui explique pourquoi ils sont recommandés par défaut plutôt que de recomposer manuellement ces trois briques séparément.

## 7. Pièges d'implémentation courants

- **Utiliser une fonction de hachage générique (SHA-256) seule pour stocker des mots de passe**, sans sel ni ralentissement volontaire : une fonction de hachage rapide permet à un attaquant de tester des milliards de mots de passe candidats par seconde sur du matériel dédié — il faut une fonction dédiée (bcrypt, scrypt, Argon2).
- **Comparer des MAC ou des empreintes avec une comparaison non constante en temps**, exposant à une attaque temporelle (section 3.1).
- **Construire une authentification maison via `H(clé || message)`** au lieu de HMAC, vulnérable à l'attaque par extension de longueur sur les fonctions de type Merkle-Damgård.
- **Publier une empreinte d'intégrité sur le même canal non authentifié que le fichier associé**, n'apportant alors qu'une protection contre la corruption accidentelle, pas contre une altération malveillante.

## À retenir

- Une fonction de hachage cryptographique doit résister à la préimage, à la seconde préimage et aux collisions ; SHA-2 et SHA-3 sont recommandées, MD5 et SHA-1 sont cassées.
- La construction en éponge de SHA-3 la rend, contrairement à SHA-2, naturellement résistante à l'attaque par extension de longueur.
- Un MAC garantit intégrité et authenticité pour des parties partageant une clé secrète, sans non-répudiation.
- HMAC neutralise spécifiquement l'attaque par extension de longueur inhérente aux fonctions de hachage basées sur Merkle-Damgård, en intégrant la clé à deux endroits du calcul.
- Privilégier un mode AEAD intégré (GCM, Poly1305) plutôt que de composer manuellement chiffrement et MAC ; si nécessaire, appliquer le principe Encrypt-then-MAC.
- Toute comparaison de secret cryptographique (MAC, empreinte, jeton) doit être effectuée en temps constant pour éviter les attaques temporelles.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
