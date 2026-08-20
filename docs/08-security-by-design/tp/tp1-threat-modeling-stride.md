# TP1 — Threat modeling STRIDE sur une application (2h)

## Cas d'étude

L'enseignant fournit la description fonctionnelle d'une application (« GesFormation », plateforme de gestion d'inscriptions à des formations en ligne) : utilisateurs (étudiants, formateurs, administrateurs), authentification, paiement en ligne, stockage de documents (justificatifs), notifications par e-mail.

## Objectifs
- Construire un diagramme de flux de données et identifier les frontières de confiance.
- Appliquer STRIDE de façon systématique.

## Exercice 1 — Diagramme de flux de données (DFD)

Construire (sur papier, ou avec un outil de diagramme simple) le DFD de « GesFormation » : entités externes (utilisateur, prestataire de paiement), processus (serveur applicatif, service de notification), magasins de données (base de données utilisateurs, stockage de documents), flux entre ces éléments. Identifier explicitement au moins 3 frontières de confiance.

## Exercice 2 — Application de STRIDE

Pour chacune des 3 frontières de confiance identifiées, examiner systématiquement les 6 catégories STRIDE et identifier au moins une menace plausible par catégorie pertinente (toutes les catégories ne s'appliquent pas nécessairement à chaque frontière — le justifier).

## Exercice 3 — Priorisation

Pour les 6 menaces jugées les plus critiques, évaluer sommairement leur sévérité (échelle qualitative simple, réutilisant le format vu en *Gouvernance et gestion des risques SI*) et les classer par priorité.

## Exercice 4 — Contre-mesures

Pour chacune des 6 menaces priorisées, proposer une contre-mesure concrète, en la reliant explicitement à un principe de conception sécurisée (chapitre 3 du cours) ou à un mécanisme technique déjà vu dans le programme (ex. chiffrement, HMAC, authentification forte).

## À rendre

Le DFD annoté, le tableau STRIDE complété, et le tableau de contre-mesures priorisées.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
