# Cryptographie

**Volume horaire :** 16h CM · 8h TP (24h)

## Objectifs pédagogiques

- Comprendre les principes fondamentaux de la cryptographie moderne (confusion, diffusion, principe de Kerckhoffs).
- Maîtriser les primitives symétriques (chiffrement par bloc/flux, modes opératoires) et asymétriques (RSA, Diffie-Hellman, ECC).
- Comprendre les fonctions de hachage, les MAC/HMAC et les signatures numériques.
- Comprendre le fonctionnement d'une infrastructure à clés publiques (PKI) et du protocole TLS.
- Utiliser des bibliothèques cryptographiques standards de façon sûre, sans réimplémenter de primitives sensibles.

## Prérequis

Des notions d'arithmétique modulaire facilitent la compréhension du chapitre 4 ; ce module est complémentaire du cours *Cryptanalyse*, qui étudie les attaques contre les mécanismes présentés ici.

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction et principes fondamentaux](cours/01-introduction-principes-fondamentaux.md) | 2h |
| 2 | [Cryptographie symétrique : chiffrement par bloc et par flux](cours/02-cryptographie-symetrique.md) | 3h |
| 3 | [Fonctions de hachage et authentification de message (MAC/HMAC)](cours/03-hachage-mac-hmac.md) | 2h |
| 4 | [Cryptographie asymétrique : RSA, Diffie-Hellman, ECC](cours/04-cryptographie-asymetrique.md) | 3h |
| 5 | [Signatures numériques et infrastructures à clés publiques (PKI)](cours/05-signatures-pki.md) | 3h |
| 6 | [Protocoles appliqués : TLS et cryptographie post-quantique](cours/06-protocoles-appliques-tls-pqc.md) | 3h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Chiffrement symétrique avec OpenSSL](tp/tp1-chiffrement-symetrique-openssl.md) | 2h |
| 2 | [Hachage, HMAC et intégrité de fichiers](tp/tp2-hachage-hmac-integrite.md) | 2h |
| 3 | [RSA : génération de clés, chiffrement et signature](tp/tp3-rsa-cles-signature.md) | 2h |
| 4 | [Analyse d'une négociation TLS et des certificats](tp/tp4-analyse-tls-certificats.md) | 2h |

## Évaluation

- TP notés : 40 %
- Contrôle continu (exercices d'arithmétique modulaire et de manipulation de primitives) : 20 %
- Examen final : 40 %

## Bibliographie

- J. Katz, Y. Lindell, *Introduction to Modern Cryptography*.
- A. Menezes, P. van Oorschot, S. Vanstone, *Handbook of Applied Cryptography*.
- Documentation OpenSSL — [openssl.org/docs](https://www.openssl.org/docs/).
- NIST — publications FIPS 197 (AES), FIPS 180-4 (SHA-2), FIPS 186-5 (signatures numériques).
