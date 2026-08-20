# TP4 — Cryptanalyse de RSA mal implémenté (2h)

## Objectifs
- Manipuler concrètement les faiblesses pratiques de RSA vues en cours (modules partagés, petits paramètres).
- Comprendre pourquoi ces attaques ne remettent pas en cause RSA correctement implémenté.

## Exercice 1 — Factorisation d'une clé RSA volontairement faible

L'enseignant fournit un module *n* RSA généré avec des nombres premiers *p* et *q* volontairement petits (à des fins pédagogiques uniquement, taille de l'ordre de 30 à 40 bits, jamais représentative d'un usage réel). Écrire un script qui factorise *n* par essai de division ou méthode de Pollard rho, retrouve *p* et *q*, reconstruit la clé privée et déchiffre un message fourni.

## Exercice 2 — Modules partagés (facteur premier commun)

L'enseignant fournit deux modules RSA *n₁* et *n₂* générés par erreur avec un facteur premier commun. Calculer `pgcd(n₁, n₂)` pour retrouver instantanément ce facteur commun, puis en déduire la factorisation complète des deux modules et déchiffrer les messages associés. Expliquer par écrit pourquoi cette situation peut se produire en pratique (générateur aléatoire de faible entropie lors de la génération des clés, notamment sur des équipements embarqués).

## Exercice 3 — Exposant public faible sans padding

L'enseignant fournit un chiffré RSA obtenu avec *e* = 3, sans padding, sur un message court. Retrouver le message en calculant la racine cubique entière du texte chiffré (le message étant plus petit que le module, aucune réduction modulaire n'a lieu). Expliquer pourquoi l'ajout d'un padding correct (OAEP) empêche cette attaque.

## Exercice 4 — Synthèse comparative

Rédiger un tableau comparatif (fourni en gabarit) listant, pour chacune des trois attaques précédentes : la cause racine (paramètre faible, absence de padding, aléa insuffisant), et la contre-mesure correspondante côté implémentation (taille de clé recommandée ≥ 2048 bits, génération avec un CSPRNG robuste, usage systématique d'OAEP/PSS).

## À rendre

Les trois scripts (exercices 1 à 3) et le tableau de synthèse (exercice 4).

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
