# Cryptanalyse

**Volume horaire :** 16h CM · 8h TP (24h)

## Objectifs pédagogiques

- Comprendre les principes et l'histoire de la cryptanalyse, de la substitution simple aux attaques sur les systèmes modernes.
- Analyser les faiblesses des chiffrements classiques par analyse fréquentielle.
- Connaître les grandes familles d'attaques contre les chiffrements symétriques, les fonctions de hachage et les systèmes asymétriques.
- Comprendre les attaques par canaux auxiliaires et leur implication pratique en sécurité des systèmes.

## Prérequis

Le cours *Cryptographie* pose les bases nécessaires (chiffrement symétrique/asymétrique, fonctions de hachage) ; il est recommandé de suivre les deux modules en parallèle ou de suivre *Cryptographie* en premier. Des notions de probabilités élémentaires et d'arithmétique modulaire sont utiles.

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction à la cryptanalyse : histoire et classification des attaques](cours/01-introduction-cryptanalyse.md) | 2h |
| 2 | [Cryptanalyse des chiffrements classiques](cours/02-cryptanalyse-chiffrements-classiques.md) | 3h |
| 3 | [Cryptanalyse des chiffrements symétriques modernes](cours/03-cryptanalyse-symetrique-moderne.md) | 3h |
| 4 | [Attaques sur les fonctions de hachage](cours/04-attaques-fonctions-hachage.md) | 2h |
| 5 | [Cryptanalyse des systèmes asymétriques](cours/05-cryptanalyse-asymetrique.md) | 3h |
| 6 | [Attaques par canaux auxiliaires](cours/06-attaques-canaux-auxiliaires.md) | 3h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Cryptanalyse par analyse fréquentielle](tp/tp1-analyse-frequentielle.md) | 2h |
| 2 | [Attaques par force brute et dictionnaire sur hachages](tp/tp2-attaques-hachage.md) | 2h |
| 3 | [Attaque sur le mode ECB et padding oracle](tp/tp3-ecb-padding-oracle.md) | 2h |
| 4 | [Cryptanalyse de RSA mal implémenté](tp/tp4-cryptanalyse-rsa.md) | 2h |

## Évaluation

- TP notés : 40 %
- Exposé sur une attaque cryptographique historique ou récente (au choix) : 20 %
- Examen final : 40 %

## Avertissement éthique

Les techniques enseignées dans ce module s'exercent exclusivement sur des exemples pédagogiques (messages, systèmes ou clés fournis pour l'exercice). Leur usage contre un système réel sans autorisation explicite est illégal et hors du cadre de ce cours.

## Bibliographie

- D. Kahn, *The Codebreakers*.
- A. Menezes, P. van Oorschot, S. Vanstone, *Handbook of Applied Cryptography* (disponible librement en ligne).
- J. Katz, Y. Lindell, *Introduction to Modern Cryptography*.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
