# Chapitre 2 — Cryptographie symétrique : chiffrement par bloc et par flux

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/02-cryptographie-symetrique.txt){ target=_blank }

## 1. Chiffrement par flux (stream cipher)

Combine le texte clair avec une suite pseudo-aléatoire (le « keystream ») générée à partir de la clé, généralement par XOR bit à bit :

```
chiffré[i] = clair[i] XOR keystream[i]
```

Exemple moderne largement utilisé : **ChaCha20**. Un chiffrement par flux exige impérativement de **ne jamais réutiliser le même couple (clé, nonce)** : réutiliser le même keystream sur deux messages différents permet de retrouver le XOR des deux messages clairs, une fuite d'information souvent exploitable (rappel du chapitre Cryptanalyse).

## 2. Chiffrement par bloc (block cipher)

Chiffre des blocs de taille fixe (128 bits pour AES) en appliquant plusieurs « tours » de transformations combinant substitution et permutation.

### AES (Advanced Encryption Standard)

Standardisé par le NIST en 2001 (algorithme Rijndael), AES opère sur des blocs de 128 bits avec des clés de 128, 192 ou 256 bits, en 10, 12 ou 14 tours respectivement. Chaque tour comprend classiquement :

1. **SubBytes** : substitution non linéaire octet par octet (boîte-S), assure la confusion.
2. **ShiftRows** : décalage cyclique des lignes de l'état, contribue à la diffusion.
3. **MixColumns** : combinaison linéaire des colonnes (absente au dernier tour), renforce la diffusion.
4. **AddRoundKey** : XOR avec une sous-clé dérivée de la clé principale pour ce tour.

## 3. Modes opératoires : pourquoi ils sont indispensables

Un chiffrement par bloc ne traite que des blocs de taille fixe : un **mode opératoire** définit comment enchaîner le chiffrement de plusieurs blocs pour traiter un message de longueur arbitraire.

| Mode | Principe | Points clés |
|---|---|---|
| **ECB** (Electronic Codebook) | chaque bloc chiffré indépendamment | **à proscrire** : des blocs clairs identiques donnent des blocs chiffrés identiques (motifs visibles, cf. TP3 du module Cryptanalyse) |
| **CBC** (Cipher Block Chaining) | chaque bloc clair est XORé avec le bloc chiffré précédent avant chiffrement ; nécessite un IV aléatoire | vulnérable aux attaques de padding oracle si le padding n'est pas vérifié de façon sûre |
| **CTR** (Counter) | transforme un chiffrement par bloc en chiffrement par flux, en chiffrant un compteur incrémental | parallélisable, mais un nonce réutilisé est aussi dangereux qu'en chiffrement par flux |
| **GCM** (Galois/Counter Mode) | mode CTR combiné à une authentification intégrée (AEAD) | **recommandé par défaut** : fournit confidentialité ET intégrité/authenticité en une seule primitive |

## 4. AEAD : chiffrement authentifié

Un mode **AEAD** (Authenticated Encryption with Associated Data), comme AES-GCM ou ChaCha20-Poly1305, combine chiffrement et code d'authentification en une seule opération, produisant un **tag d'authentification** vérifié au déchiffrement. Si le texte chiffré a été modifié (accidentellement ou par un attaquant), le déchiffrement échoue explicitement plutôt que de produire silencieusement un texte clair corrompu ou manipulé.

**Recommandation pratique actuelle** : privilégier systématiquement un mode AEAD (AES-GCM, ChaCha20-Poly1305) plutôt que de composer soi-même un mode CBC avec un HMAC séparé — une composition manuelle est une source fréquente d'erreurs d'implémentation.

## 5. Gestion de la clé et du vecteur d'initialisation (IV)/nonce

- La **clé** doit être générée par un générateur aléatoire cryptographiquement sûr (CSPRNG) et jamais codée en dur dans le code source.
- L'**IV/nonce** doit être unique pour chaque chiffrement avec une même clé (aléatoire pour CBC, souvent un compteur pour CTR/GCM) — sa réutilisation est l'une des erreurs d'implémentation les plus dommageables en pratique.

## 6. Dérivation de clé à partir d'un mot de passe

Un mot de passe humain n'a pas l'entropie suffisante pour servir directement de clé de chiffrement : on utilise une **fonction de dérivation de clé (KDF)** dédiée (PBKDF2, scrypt, Argon2), volontairement coûteuse en calcul, pour ralentir les attaques par force brute (rappel du chapitre Cryptanalyse sur le hachage de mots de passe).

## À retenir

- AES est le standard de chiffrement par bloc de référence ; sa sécurité dépend fortement du mode opératoire choisi.
- ECB est à proscrire ; les modes AEAD (GCM) sont recommandés par défaut car ils apportent confidentialité et intégrité simultanément.
- La réutilisation d'un nonce/IV avec la même clé est l'une des erreurs d'implémentation les plus graves en cryptographie symétrique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
