# Chapitre 2 — Rappels de machine learning

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=05-ia-appliquee-securite-informatique/slides/02-rappels-machine-learning.txt){ target=_blank }

## 1. Apprentissage supervisé, non supervisé, par renforcement

| Type | Principe | Exemple en sécurité |
|---|---|---|
| **Supervisé** | apprend à partir d'exemples étiquetés (entrée, sortie attendue) | classifier un fichier comme malware/sain à partir d'exemples déjà étiquetés |
| **Non supervisé** | découvre une structure dans des données non étiquetées | regrouper des connexions réseau similaires, détecter des anomalies |
| **Par renforcement** | un agent apprend par essai-erreur en maximisant une récompense | automatisation de la réponse à incident (usage encore émergent) |

## 2. Classification vs régression

- **Classification** : prédire une catégorie (ex. « malveillant » / « bénin »).
- **Régression** : prédire une valeur continue (ex. score de risque entre 0 et 100).

La plupart des cas d'usage en détection de sécurité relèvent de la classification (binaire ou multi-classe).

## 3. Cycle de vie d'un projet de machine learning

1. **Collecte et préparation des données** : nettoyage, gestion des valeurs manquantes, normalisation.
2. **Ingénierie des caractéristiques (feature engineering)** : transformer des données brutes (paquets réseau, texte) en variables numériques exploitables par un modèle.
3. **Séparation des données** : ensemble d'entraînement, de validation, de test — pour éviter le surapprentissage (*overfitting*).
4. **Entraînement du modèle**.
5. **Évaluation** sur des données jamais vues par le modèle.
6. **Déploiement et supervision continue** (un modèle en production doit être réévalué régulièrement : concept drift).

## 4. Quelques algorithmes classiques

| Algorithme | Type | Caractéristique |
|---|---|---|
| Régression logistique | supervisé, classification | simple, interprétable, bonne base de référence |
| Arbre de décision / Forêt aléatoire (Random Forest) | supervisé, classification/régression | robuste, gère bien les données hétérogènes, relativement interprétable |
| SVM (Support Vector Machine) | supervisé, classification | efficace en haute dimension |
| K-means | non supervisé, clustering | regroupe des observations similaires |
| Isolation Forest | non supervisé, détection d'anomalies | isole les observations atypiques efficacement, utilisé en TP2 |
| Réseaux de neurones / Deep Learning | supervisé ou non supervisé | performant sur données complexes (texte, image, séquences) mais moins interprétable |

## 5. Métriques d'évaluation, essentielles en sécurité

Pour une classification binaire, la matrice de confusion distingue :

|  | Prédit positif | Prédit négatif |
|---|---|---|
| **Réel positif** | Vrai Positif (VP) | Faux Négatif (FN) |
| **Réel négatif** | Faux Positif (FP) | Vrai Négatif (VN) |

- **Précision** = VP / (VP + FP) : parmi les alertes levées, quelle proportion est réellement une menace ?
- **Rappel (recall)** = VP / (VP + FN) : parmi les menaces réelles, quelle proportion a été détectée ?
- **F1-score** : moyenne harmonique de précision et rappel.
- **Taux de faux positifs** = FP / (FP + VN) : critique en sécurité, car un taux élevé provoque la « fatigue d'alerte » des analystes, qui finissent par ignorer les alertes.

**En sécurité, la simple exactitude globale (accuracy) est souvent trompeuse** : sur un jeu de données très déséquilibré (ex. 0,1 % de trafic réellement malveillant), un modèle qui prédit toujours « bénin » atteint 99,9 % d'exactitude tout en étant totalement inutile. D'où l'importance du rappel et de la précision, en particulier sur la classe minoritaire (l'attaque).

## 6. Surapprentissage et sous-apprentissage

- **Surapprentissage (overfitting)** : le modèle « apprend par cœur » les données d'entraînement et généralise mal à de nouvelles données.
- **Sous-apprentissage (underfitting)** : le modèle est trop simple pour capturer la structure des données.

La validation croisée (cross-validation) et un ensemble de test totalement indépendant permettent de détecter ces situations avant tout déploiement.

## À retenir

- Le choix entre supervisé/non supervisé dépend de la disponibilité de données étiquetées, souvent rare en sécurité pour les attaques nouvelles.
- Précision, rappel et F1-score, pas la seule exactitude, sont les métriques pertinentes pour évaluer un modèle de détection en contexte déséquilibré.
- Le cycle de vie complet (données → features → entraînement → évaluation → supervision continue) doit être maîtrisé, pas seulement l'entraînement du modèle.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
