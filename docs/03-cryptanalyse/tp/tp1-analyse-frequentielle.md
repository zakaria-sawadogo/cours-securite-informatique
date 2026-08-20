# TP1 — Cryptanalyse par analyse fréquentielle (2h)

## Objectifs
- Casser un chiffre de substitution mono-alphabétique et un chiffre de Vigenère.
- Implémenter les outils statistiques (fréquences, indice de coïncidence) en Python.

## Exercice 1 — Analyse fréquentielle d'un chiffre de César

L'enseignant fournit un texte chiffré par César. Écrire un script (Python recommandé) qui teste les 26 décalages possibles et affiche celui produisant le texte le plus proche du français (score basé sur la fréquence des lettres, ou détection de mots courants).

## Exercice 2 — Cassage d'une substitution mono-alphabétique

À partir d'un texte chiffré fourni (substitution quelconque), calculer la fréquence de chaque lettre, la comparer au profil du français, et proposer une première hypothèse de correspondance. Affiner manuellement par itérations successives jusqu'à obtenir un texte clair cohérent.

## Exercice 3 — Indice de coïncidence

Implémenter une fonction `indice_coincidence(texte)` calculant l'IC d'un texte. Vérifier sur un texte clair en français (IC proche de 0,078) et sur un texte chiffré par Vigenère avec une clé longue (IC proche de 0,038).

## Exercice 4 — Cassage du chiffre de Vigenère

À partir d'un texte chiffré par Vigenère fourni :
1. Estimer la longueur de clé par la méthode de Kasiski (repérage de séquences répétées) **et** par l'indice de coïncidence moyen des sous-suites, pour plusieurs longueurs candidates.
2. Une fois la longueur *L* déterminée, casser chaque sous-alphabet par analyse fréquentielle (réutiliser le script de l'exercice 1).
3. Reconstituer la clé et déchiffrer le message complet.

## À rendre

Les scripts Python des exercices 1, 3 et 4, ainsi que le texte clair et la clé retrouvés à l'exercice 4.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
