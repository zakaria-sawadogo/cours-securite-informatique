# TP2 — Simulation d'un consensus par preuve de travail (2h)

## Objectifs
- Implémenter un mécanisme simplifié de preuve de travail (PoW).
- Observer concrètement l'effet du niveau de difficulté sur le temps de calcul.

## Exercice 1 — Minage simplifié

À partir de la classe `Bloc` du TP1, implémenter une fonction `miner(bloc, difficulte)` qui incrémente le `nonce` du bloc jusqu'à ce que son hash commence par `difficulte` zéros (ex. `difficulte = 4` → hash commençant par `"0000"`). Mesurer le nombre d'essais et le temps nécessaire.

## Exercice 2 — Impact de la difficulté

Miner successivement des blocs avec `difficulte` variant de 1 à 6. Tracer un graphique (temps de calcul en fonction de la difficulté) et commenter la croissance observée (théoriquement exponentielle en base 16, la difficulté étant exprimée en préfixe hexadécimal de zéros).

## Exercice 3 — Ajustement dynamique de la difficulté (simplifié)

Implémenter une règle simple d'ajustement automatique : si le dernier bloc a été miné en moins de 2 secondes, augmenter la difficulté de 1 ; s'il a fallu plus de 5 secondes, la diminuer de 1. Simuler le minage de 10 blocs consécutifs avec cet ajustement et observer la stabilisation du temps de minage.

## Exercice 4 — Simulation d'une attaque des 51 % (conceptuelle)

Sans implémentation complète, répondre par écrit (15 lignes) : en supposant une blockchain à 6 nœuds mineurs de puissance égale, combien de nœuds un attaquant devrait-il contrôler pour réécrire l'historique avec certitude ? Comment ce raisonnement change-t-il si les puissances de calcul sont très inégales entre les nœuds ? En quoi ce mécanisme diffère-t-il fondamentalement d'une preuve d'enjeu du point de vue du coût d'attaque (cf. chapitre 3 du cours) ?

## À rendre

Le script Python complet, le graphique de l'exercice 2, et la réponse écrite de l'exercice 4.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
