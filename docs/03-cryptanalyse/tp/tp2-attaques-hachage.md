# TP2 — Attaques par force brute et dictionnaire sur hachages (2h)

## Cadre du TP

Toutes les manipulations portent sur des empreintes générées pour l'exercice, jamais sur des données réelles d'un tiers.

## Objectifs
- Comprendre concrètement l'impact du salage et du choix de la fonction de hachage sur la résistance aux attaques de mots de passe.

## Exercice 1 — Attaque par dictionnaire sur MD5 non salé

L'enseignant fournit une liste d'empreintes MD5 de mots de passe faibles (issus d'une petite liste de mots courants). Écrire un script qui hache chaque mot d'un dictionnaire fourni et compare au hachage cible, jusqu'à trouver une correspondance. Mesurer le temps nécessaire.

## Exercice 2 — Effet du salage

Générer un mot de passe haché avec et sans sel (`SHA-256(motdepasse)` vs `SHA-256(sel || motdepasse)` avec un sel aléatoire par entrée). Expliquer par écrit pourquoi une table précalculée (rainbow table) devient inefficace en présence d'un sel unique par utilisateur.

## Exercice 3 — Comparaison de coût : SHA-256 vs bcrypt

Comparer expérimentalement (avec la bibliothèque `bcrypt` de votre langage) le temps nécessaire pour hacher 1000 mots de passe avec SHA-256 simple contre bcrypt (facteur de coût 12). En déduire l'impact sur la faisabilité d'une attaque par force brute à grande échelle.

## Exercice 4 — Démonstration de l'attaque par extension de longueur (guidée)

Sous supervision de l'enseignant, observer sur un exemple simplifié comment, connaissant `SHA-256(clé || message)` et la longueur de `clé`, il est possible de calculer `SHA-256(clé || message || padding || extension)` sans connaître `clé`, en utilisant un outil dédié (ex. `hashpump`). En déduire pourquoi HMAC (et non une simple concaténation) doit être utilisé pour authentifier un message.

## À rendre

Les scripts des exercices 1 à 3, et une réponse écrite (10 lignes maximum) à la question de l'exercice 4.
