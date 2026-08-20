# TP1 — Chiffrement symétrique avec OpenSSL (2h)

## Objectifs
- Manipuler AES en pratique avec OpenSSL en ligne de commande.
- Comparer différents modes opératoires et observer leurs faiblesses/forces.

## Exercice 1 — Chiffrement/déchiffrement AES-256-GCM

```bash
# Génération d'une clé aléatoire de 256 bits
openssl rand -hex 32 > cle.txt

# Chiffrement
openssl enc -aes-256-gcm -in message.txt -out message.enc -K $(cat cle.txt) -iv $(openssl rand -hex 12)

# Déchiffrement (utiliser le même IV que celui généré au chiffrement)
```

Documenter chaque paramètre utilisé et vérifier que le message déchiffré correspond bien à l'original.

## Exercice 2 — Démonstration de la faiblesse d'ECB

Reprendre l'exercice du module *Cryptanalyse* (TP3) : chiffrer une image bitmap simple en mode ECB puis en mode CBC avec OpenSSL ou une bibliothèque de votre langage, comparer visuellement les résultats, et rédiger 5 lignes expliquant pourquoi ECB ne doit jamais être utilisé en production.

## Exercice 3 — Dérivation de clé à partir d'un mot de passe

Utiliser une fonction de dérivation de clé (ex. PBKDF2 via `openssl enc -pbkdf2` ou une bibliothèque comme `cryptography` en Python avec `Scrypt`/`PBKDF2HMAC`) pour dériver une clé de chiffrement à partir d'un mot de passe et d'un sel. Chiffrer un fichier avec la clé dérivée et vérifier le déchiffrement.

## Exercice 4 — Réutilisation de nonce (démonstration guidée)

Sous supervision, chiffrer deux messages différents avec ChaCha20 (ou AES-CTR) en réutilisant volontairement le même couple (clé, nonce). Calculer le XOR des deux textes chiffrés obtenus et observer qu'il correspond exactement au XOR des deux textes clairs, révélant une fuite d'information sans même connaître la clé. Expliquer par écrit pourquoi ceci confirme l'importance de l'unicité du nonce.

## À rendre

Les commandes/scripts utilisés pour chaque exercice, les fichiers de résultat, et les réponses écrites aux exercices 2 et 4.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
