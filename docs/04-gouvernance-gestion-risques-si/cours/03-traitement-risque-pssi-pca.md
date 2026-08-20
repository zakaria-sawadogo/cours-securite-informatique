# Chapitre 3 — Traitement du risque, PSSI et continuité d'activité

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=04-gouvernance-gestion-risques-si/slides/03-traitement-risque-pssi-pca.txt){ target=_blank }

## 1. Les quatre stratégies de traitement du risque

| Stratégie | Principe | Exemple |
|---|---|---|
| **Éviter** | supprimer l'activité ou l'actif générant le risque | ne pas exposer un service inutile sur Internet |
| **Réduire (atténuer)** | mettre en œuvre des mesures pour diminuer probabilité et/ou impact | déployer un pare-feu, chiffrer les données |
| **Transférer (partager)** | déplacer une partie de l'impact vers un tiers | souscrire une cyber-assurance, externaliser à un prestataire avec obligations contractuelles |
| **Accepter** | assumer consciemment le risque, en général si le coût du traitement dépasse l'impact potentiel | accepter un risque résiduel faible après mise en œuvre des mesures principales |

Le choix résulte d'un arbitrage coût/bénéfice, validé par le niveau de gouvernance approprié (la direction pour les risques majeurs).

## 2. La Politique de Sécurité des Systèmes d'Information (PSSI)

La PSSI est le document de référence formalisant les orientations stratégiques et les règles de sécurité applicables à l'ensemble de l'organisation. Elle se décline généralement en plusieurs niveaux :

1. **Politique générale** : principes, engagement de la direction, périmètre (quelques pages, stable dans le temps).
2. **Politiques thématiques** : contrôle d'accès, gestion des mots de passe, usage des équipements mobiles, télétravail, gestion des tiers.
3. **Procédures opérationnelles** : instructions détaillées d'application (gestion des incidents, sauvegardes).

### Structure type d'une politique générale

- Objet et périmètre.
- Engagement de la direction.
- Organisation et responsabilités (rôles définis au chapitre 1).
- Principes directeurs (moindre privilège, défense en profondeur, séparation des tâches).
- Gestion des risques et conformité.
- Sanctions en cas de non-respect.
- Modalités de révision.

Une PSSI efficace est **connue, appliquée et révisée régulièrement** — une politique rédigée puis jamais diffusée ni actualisée est un signe classique d'immaturité relevé en audit organisationnel.

## 3. Plan de Continuité d'Activité (PCA) et Plan de Reprise d'Activité (PRA)

- **PCA** : ensemble des dispositions visant à assurer, en cas de crise majeure, le maintien (éventuellement dégradé) des activités essentielles de l'organisation.
- **PRA** : volet plus spécifiquement technique du PCA, centré sur la reprise des systèmes d'information après un sinistre.

### Notions clés

| Indicateur | Définition |
|---|---|
| **RTO** (Recovery Time Objective) | durée maximale d'interruption acceptable avant reprise du service |
| **RPO** (Recovery Point Objective) | perte de données maximale acceptable, exprimée en temps (ex. RPO de 4h = perte maximale des 4 dernières heures de données) |

Ces deux indicateurs déterminent directement l'architecture de résilience à mettre en œuvre (fréquence de sauvegarde, site de secours actif/passif, réplication temps réel).

### Étapes d'élaboration d'un PCA

1. Analyse d'impact sur l'activité (BIA — Business Impact Analysis) : identifier les processus critiques et leur RTO/RPO cible.
2. Définition des stratégies de continuité (site de secours, télétravail de crise, procédures dégradées).
3. Rédaction des plans opérationnels (qui fait quoi, avec quelles ressources).
4. Tests réguliers (exercices de crise, tests techniques de bascule).
5. Mise à jour continue.

Un PCA non testé est un PCA dont l'efficacité réelle est inconnue : les tests réguliers sont aussi importants que la rédaction du plan lui-même.

## 4. Gestion de crise cyber

Distincte mais complémentaire du PCA/PRA : une cellule de crise dédiée (souvent associant RSSI, DSI, communication, juridique, direction) doit pouvoir être activée rapidement en cas d'incident majeur (ex. rançongiciel), avec des procédures de décision pré-établies (isoler, communiquer, notifier les autorités compétentes le cas échéant).

## À retenir

- Quatre stratégies de traitement du risque : éviter, réduire, transférer, accepter — le choix relève d'un arbitrage validé par la gouvernance.
- La PSSI se décline en plusieurs niveaux, de la politique générale aux procédures opérationnelles, et doit être vivante (diffusée, appliquée, révisée).
- RTO et RPO traduisent en exigences concrètes la criticité des processus métier et dimensionnent le PCA/PRA ; un plan non testé reste théorique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
