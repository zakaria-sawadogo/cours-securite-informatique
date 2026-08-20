# TP1 — Classification de trafic réseau (NSL-KDD) (2h)

## Objectifs
- Prendre en main `scikit-learn` sur un jeu de données de sécurité réel.
- Entraîner et évaluer un classifieur supervisé de détection d'intrusions.

## Préparation

```bash
pip install scikit-learn pandas numpy matplotlib --break-system-packages
```

Télécharger (ou récupérer via le dossier fourni par l'enseignant) le jeu de données **NSL-KDD** (version corrigée du célèbre KDD Cup 99), qui contient des connexions réseau étiquetées « normal » ou selon un type d'attaque (DoS, probe, R2L, U2R).

## Exercice 1 — Exploration des données

Charger le jeu de données avec `pandas`. Afficher la répartition des classes (normal vs types d'attaques). Identifier le déséquilibre des classes et discuter son impact potentiel sur l'entraînement.

## Exercice 2 — Prétraitement

Encoder les variables catégorielles (protocole, service, flag) en variables numériques (`OneHotEncoder` ou `LabelEncoder`). Normaliser les variables numériques (`StandardScaler`). Séparer les données en ensembles d'entraînement (70 %) et de test (30 %), en conservant la répartition des classes (`stratify`).

## Exercice 3 — Entraînement de deux modèles

Entraîner un modèle de régression logistique et un modèle Random Forest sur la tâche de classification binaire (normal / attaque). Comparer leurs performances.

## Exercice 4 — Évaluation

Pour chaque modèle, calculer la matrice de confusion, la précision, le rappel et le F1-score (pas uniquement l'exactitude — se référer au chapitre 2 du cours). Discuter par écrit : quel modèle privilégier si l'on cherche à minimiser les faux négatifs (attaques non détectées), même au prix de plus de faux positifs ?

## Exercice 5 — Importance des variables (Random Forest)

Afficher les 10 variables les plus importantes selon le modèle Random Forest (`feature_importances_`) et commenter leur cohérence avec les connaissances métier (ex. nombre de connexions échouées, durée de connexion).

## À rendre

Un notebook ou script Python contenant l'ensemble des étapes, les métriques obtenues pour les deux modèles, et les réponses écrites aux questions de discussion.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
