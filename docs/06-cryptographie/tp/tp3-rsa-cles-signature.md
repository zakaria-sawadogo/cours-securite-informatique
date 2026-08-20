# TP3 — RSA : génération de clés, chiffrement et signature (2h)

## Objectifs
- Générer une paire de clés RSA et manipuler le chiffrement et la signature avec OpenSSL.
- Comprendre l'importance du padding.

## Exercice 1 — Génération de clés

```bash
openssl genrsa -out privee.pem 2048
openssl rsa -in privee.pem -pubout -out publique.pem
```

Examiner (`openssl rsa -in privee.pem -text -noout`) les composants de la clé générée : modulus, exposant public, exposant privé.

## Exercice 2 — Chiffrement et déchiffrement avec padding OAEP

```bash
openssl pkeyutl -encrypt -pubin -inkey publique.pem -pkeyopt rsa_padding_mode:oaep -in message.txt -out message.enc
openssl pkeyutl -decrypt -inkey privee.pem -pkeyopt rsa_padding_mode:oaep -in message.enc -out message.dec
```

Comparer avec un chiffrement sans padding (`rsa_padding_mode:none`, à des fins strictement pédagogiques) et observer les limites pratiques (taille de message, déterminisme).

## Exercice 3 — Signature et vérification (RSA-PSS)

```bash
openssl dgst -sha256 -sign privee.pem -sigopt rsa_padding_mode:pss -out signature.bin document.txt
openssl dgst -sha256 -verify publique.pem -sigopt rsa_padding_mode:pss -signature signature.bin document.txt
```

Modifier un octet de `document.txt` et vérifier que la signature échoue à se vérifier.

## Exercice 4 — Modules partagés (lien avec le module Cryptanalyse)

Reprendre l'exercice 2 du TP4 de *Cryptanalyse* (facteur premier commun entre deux modules RSA) si non déjà traité, ou, à défaut, en rédiger une synthèse de 10 lignes expliquant le lien entre une mauvaise génération de nombres premiers ici (exercice 1) et cette classe d'attaque.

## Exercice 5 — Comparaison RSA / Ed25519

Générer une paire de clés Ed25519 (`openssl genpkey -algorithm ed25519`) et comparer le temps de génération de clé et de signature avec RSA 2048 bits. Commenter les résultats en lien avec le chapitre 5 du cours.

## À rendre

Les commandes utilisées, les fichiers de clés/signatures générés, et les réponses écrites aux exercices 4 et 5.
