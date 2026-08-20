# TP2 — Détection d'anomalies non supervisée (2h)

## Objectifs
- Mettre en œuvre une détection d'anomalies sans étiquettes, situation réaliste pour des menaces inédites.
- Comparer Isolation Forest et une approche par clustering.

## Exercice 1 — Isolation Forest

Sur le même jeu de données NSL-KDD (TP1), en **ignorant les étiquettes** lors de l'entraînement, entraîner un modèle `IsolationForest` de `scikit-learn` sur les caractéristiques numériques du trafic. Utiliser les étiquettes réelles **uniquement a posteriori**, pour évaluer si les connexions marquées comme anomalies par le modèle correspondent effectivement à des attaques.

## Exercice 2 — Réglage du taux de contamination

Faire varier le paramètre `contamination` (proportion attendue d'anomalies) et observer son impact sur le rappel et la précision (par rapport aux vraies étiquettes, à des fins d'évaluation uniquement). Tracer un graphique précision/rappel en fonction de ce paramètre.

## Exercice 3 — Comparaison avec K-means

Appliquer un clustering K-means (avec un nombre de clusters à choisir et justifier, ex. via la méthode du coude) sur les mêmes données. Considérer comme anomalies les points les plus éloignés du centre de leur cluster. Comparer les performances obtenues à celles de l'Isolation Forest.

## Exercice 4 — Cas d'usage sans étiquettes disponibles

Rédiger une réponse courte (10 lignes) : dans un contexte réel où l'on ne dispose d'aucune étiquette d'attaque connue (nouveau réseau, menace zero-day potentielle), comment évaluer la pertinence des anomalies détectées avant de les transformer en alertes opérationnelles pour un analyste SOC ?

## À rendre

Le notebook/script avec les deux approches, le graphique de l'exercice 2, et la réponse de l'exercice 4.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
