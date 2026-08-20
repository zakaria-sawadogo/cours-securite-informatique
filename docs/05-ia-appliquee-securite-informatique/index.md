# IA appliquée à la sécurité informatique

**Volume horaire :** 16h CM · 8h TP (24h)

## Objectifs pédagogiques

- Comprendre les apports et les limites de l'intelligence artificielle et du machine learning pour la cybersécurité défensive (détection d'intrusions, de malwares, de phishing).
- Mettre en œuvre des modèles de machine learning simples sur des données de sécurité (réseau, logs, texte).
- Comprendre les risques spécifiques à l'IA elle-même en tant que surface d'attaque (attaques adversariales, empoisonnement de données) et les usages offensifs de l'IA.

## Prérequis

Bases de programmation (le module *Algorithmique et Programmation en C* apporte les fondamentaux de logique algorithmique ; les TP de ce module utilisent Python) et notions de statistiques élémentaires.

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction : IA et cybersécurité, panorama](cours/01-introduction-ia-cybersecurite.md) | 2h |
| 2 | [Rappels de machine learning](cours/02-rappels-machine-learning.md) | 3h |
| 3 | [Détection d'anomalies et d'intrusions par ML](cours/03-detection-anomalies-intrusions.md) | 3h |
| 4 | [Détection de malwares par machine learning](cours/04-detection-malwares-ml.md) | 2h |
| 5 | [NLP appliqué à la sécurité (phishing, logs)](cours/05-nlp-securite.md) | 3h |
| 6 | [Sécurité de l'IA : attaques adversariales et IA offensive](cours/06-securite-de-lia.md) | 3h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Classification de trafic réseau (NSL-KDD)](tp/tp1-classification-trafic-reseau.md) | 2h |
| 2 | [Détection d'anomalies non supervisée](tp/tp2-detection-anomalies.md) | 2h |
| 3 | [Détection de phishing par NLP](tp/tp3-detection-phishing-nlp.md) | 2h |
| 4 | [Attaque adversariale sur un classifieur](tp/tp4-attaque-adversariale.md) | 2h |

## Évaluation

- TP notés : 40 %
- Projet (application d'un modèle ML à un jeu de données de sécurité au choix) : 30 %
- Examen final : 30 %

## Environnement technique

Python 3, `scikit-learn`, `pandas`, `numpy`, `matplotlib` ; `PyTorch` ou `TensorFlow` pour les exercices avancés (TP4). Jeux de données publics utilisés : NSL-KDD (intrusions réseau), un corpus public d'e-mails de phishing.

## Bibliographie

- S. Raschka, V. Mirjalili, *Python Machine Learning*.
- I. Goodfellow, Y. Bengio, A. Courville, *Deep Learning*.
- OWASP Machine Learning Security Top 10.
- MITRE ATLAS (Adversarial Threat Landscape for Artificial-Intelligence Systems).

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
