# Chapitre 4 — Attaques sur les fonctions de hachage

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/04-attaques-fonctions-hachage.txt){ target=_blank }

## 1. Rappel des propriétés attendues

Une fonction de hachage cryptographique doit garantir : la **résistance à la préimage** (impossible de retrouver un message à partir de son empreinte), la **résistance à la seconde préimage** (impossible de trouver un autre message ayant la même empreinte qu'un message donné), et la **résistance aux collisions** (impossible de trouver deux messages distincts ayant la même empreinte).

## 2. Le paradoxe des anniversaires et l'attaque par collision

Pour une empreinte de *n* bits, trouver une collision par force brute « naïve » coûterait en théorie 2^n essais. Le **paradoxe des anniversaires** montre qu'il suffit en réalité d'environ 2^(n/2) essais pour trouver une collision avec une probabilité significative (par analogie : dans un groupe de 23 personnes, la probabilité que deux partagent un anniversaire dépasse 50 %, bien moins que les 365 attendus intuitivement).

Conséquence directe : une fonction de hachage de *n* bits n'offre qu'une sécurité de *n/2* bits contre les collisions, ce qui explique pourquoi MD5 (128 bits, soit 64 bits de sécurité en collision) est aujourd'hui totalement cassé en pratique, et pourquoi SHA-1 (160 bits) est déconseillé depuis la démonstration d'une collision pratique en 2017 (attaque « SHAttered »).

## 3. Attaques historiques marquantes

- **MD5** : collisions pratiques démontrées dès 2004-2005, exploitées ensuite pour forger de faux certificats numériques (affaire du malware Flame, 2012).
- **SHA-1** : collision pratique démontrée par Google et le CWI en 2017, accélérant la migration généralisée vers SHA-2/SHA-3.
- **SHA-2 (SHA-256, SHA-512) et SHA-3** : à ce jour, aucune attaque pratique ne remet en cause leur sécurité ; ce sont les standards recommandés.

## 4. Attaque par extension de longueur (length extension attack)

Les fonctions basées sur la construction de Merkle-Damgård (MD5, SHA-1, SHA-2) sont vulnérables à l'attaque par extension de longueur : connaissant `H(message)` et la longueur de `message` (mais pas `message` lui-même), un attaquant peut calculer `H(message || padding || extension)` pour une extension de son choix, **sans connaître `message`**.

Conséquence pratique : construire une authentification de message par simple concaténation `H(clé || message)` est dangereux — un attaquant peut forger un MAC valide pour un message étendu sans connaître la clé. La construction **HMAC** (chapitre Cryptographie) a été spécifiquement conçue pour neutraliser cette attaque.

## 5. Hachage de mots de passe : attaques et contre-mesures

| Attaque | Principe | Contre-mesure |
|---|---|---|
| Force brute / dictionnaire | tester des mots de passe candidats et comparer les empreintes | politique de mots de passe robustes, limitation du nombre d'essais |
| Table arc-en-ciel | tables précalculées d'empreintes pour inverser rapidement un hachage non salé | **salage** (valeur aléatoire unique par mot de passe, ajoutée avant hachage) |
| Calcul massivement parallèle (GPU/ASIC) | les fonctions de hachage génériques (SHA-256) sont très rapides à calculer, donc favorables à l'attaquant | fonctions **spécialement conçues pour être lentes** : bcrypt, scrypt, Argon2 (facteur de coût réglable) |

## 6. Méthodologie d'évaluation d'une fonction de hachage

1. Vérifier la taille de sortie (borne théorique de sécurité en collision = taille/2).
2. Vérifier l'absence d'attaques publiées de préimage/collision pratiques.
3. Vérifier le contexte d'usage : une fonction sûre pour l'intégrité de fichiers (SHA-256) ne l'est pas nécessairement pour le hachage de mots de passe sans adaptation (facteur de travail, salage).

## À retenir

- Le paradoxe des anniversaires divise par deux, en bits, la sécurité effective d'une fonction de hachage contre les collisions.
- MD5 et SHA-1 sont cassés en pratique pour les collisions et ne doivent plus être utilisés pour un usage sécuritaire.
- Le hachage de mots de passe exige des fonctions dédiées (bcrypt, scrypt, Argon2), volontairement lentes et salées — jamais une fonction générique seule (SHA-256 brut).
