# Chapitre 3 — Fonctions de hachage et authentification de message (MAC/HMAC)

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/03-hachage-mac-hmac.txt){ target=_blank }

## 1. Fonctions de hachage cryptographiques

Une fonction de hachage `H` transforme une entrée de taille arbitraire en une sortie de taille fixe (l'empreinte ou *digest*), avec trois propriétés attendues (détaillées côté attaque au chapitre 4 du module *Cryptanalyse*) :

- résistance à la préimage,
- résistance à la seconde préimage,
- résistance aux collisions.

### Panorama des fonctions usuelles

| Fonction | Taille de sortie | Statut |
|---|---|---|
| MD5 | 128 bits | **cassée** (collisions pratiques), à ne plus utiliser pour un usage sécuritaire |
| SHA-1 | 160 bits | **cassée** (collision démontrée en 2017), dépréciée |
| SHA-2 (SHA-256, SHA-384, SHA-512) | 256 à 512 bits | recommandée, aucune attaque pratique connue |
| SHA-3 (Keccak) | 224 à 512 bits | recommandée, construction interne différente de SHA-2 (fonction éponge), diversifie le risque en cas de faille future sur la famille SHA-2 |

## 2. Usages des fonctions de hachage

- **Vérification d'intégrité** : comparer l'empreinte d'un fichier téléchargé à une valeur de référence publiée.
- **Structures de données** : arbres de Merkle (utilisés en Blockchain, module suivant), tables de hachage.
- **Brique de construction** : dérivation de clé, génération de nombres pseudo-aléatoires, signatures numériques (chapitre 5), preuve de travail.
- **Hachage de mots de passe** : nécessite des fonctions dédiées et volontairement lentes (bcrypt, scrypt, Argon2), jamais une fonction générique seule (cf. module Cryptanalyse, chapitre 4).

## 3. Code d'authentification de message (MAC)

Un MAC garantit à la fois **l'intégrité** d'un message et **l'authenticité** de son origine, pour deux parties partageant une clé secrète : `MAC = Fk(message)`, où seule une partie possédant la clé `k` peut produire ou vérifier un MAC valide.

À la différence d'une signature numérique (chapitre 5), un MAC ne fournit pas la non-répudiation : toute partie capable de vérifier le MAC est également capable d'en produire un (elle possède la même clé).

## 4. HMAC (Hash-based MAC)

Construction générique permettant de bâtir un MAC à partir de n'importe quelle fonction de hachage :

```
HMAC(k, m) = H( (k' XOR opad) || H( (k' XOR ipad) || m ) )
```

où `k'` est la clé complétée à la taille de bloc de `H`, et `ipad`/`opad` des constantes fixes. Cette double application de `H`, avec la clé combinée de part et d'autre, neutralise spécifiquement l'attaque par extension de longueur qui rend dangereuse une simple concaténation `H(clé || message)` (vue au chapitre 4 du module *Cryptanalyse*).

HMAC-SHA256 est aujourd'hui l'un des mécanismes de MAC les plus utilisés (authentification d'API, jetons JWT signés en HS256, intégrité de sessions).

## 5. MAC intégré : GMAC et Poly1305

Les modes AEAD vus au chapitre précédent (AES-GCM, ChaCha20-Poly1305) intègrent directement un mécanisme de MAC (GMAC pour GCM, Poly1305 pour ChaCha20) dans l'opération de chiffrement elle-même, évitant d'avoir à composer manuellement chiffrement et authentification — une composition manuelle mal ordonnée (ex. authentifier avant de chiffrer plutôt que l'inverse) est une source d'erreurs classique.

## 6. Ordre chiffrement/authentification (Encrypt-then-MAC)

Lorsqu'un schéma AEAD intégré n'est pas utilisé et qu'il faut composer soi-même chiffrement et MAC, la construction recommandée est **Encrypt-then-MAC** (chiffrer d'abord, puis authentifier le texte chiffré) plutôt que MAC-then-Encrypt ou Encrypt-and-MAC, car elle permet de rejeter un texte chiffré invalide avant même de tenter de le déchiffrer, limitant la surface d'attaques par oracle (padding oracle notamment).

## À retenir

- Un MAC garantit intégrité et authenticité pour des parties partageant une clé secrète, sans non-répudiation.
- HMAC neutralise spécifiquement l'attaque par extension de longueur inhérente aux fonctions de hachage basées sur Merkle-Damgård.
- Privilégier un mode AEAD intégré (GCM, Poly1305) plutôt que de composer manuellement chiffrement et MAC ; si nécessaire, appliquer le principe Encrypt-then-MAC.
