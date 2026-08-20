# Chapitre 2 — Rappels de machine learning

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=05-ia-appliquee-securite-informatique/slides/02-rappels-machine-learning.txt){ target=_blank }

## 1. Apprentissage supervisé, non supervisé, par renforcement

| Type | Principe | Exemple en sécurité |
|---|---|---|
| **Supervisé** | apprend à partir d'exemples étiquetés (entrée, sortie attendue) | classifier un fichier comme malware/sain à partir d'exemples déjà étiquetés |
| **Non supervisé** | découvre une structure dans des données non étiquetées | regrouper des connexions réseau similaires, détecter des anomalies |
| **Par renforcement** | un agent apprend par essai-erreur en maximisant une récompense | automatisation de la réponse à incident (usage encore émergent) |

### 1.1 Pourquoi cette distinction est centrale en sécurité

Le choix entre ces trois paradigmes n'est pas qu'une question académique : il est directement contraint par la disponibilité de données étiquetées, qui pose un problème récurrent en sécurité.

- Pour entraîner un modèle **supervisé** de détection de malwares, il faut disposer d'un grand nombre d'exemples déjà étiquetés « malveillant » ou « sain ». C'est possible grâce à des bases partagées par la communauté (échantillons soumis à des services d'analyse), mais ces exemples sont nécessairement **rétrospectifs** : ils décrivent des menaces déjà connues, jamais les attaques de demain.
- Les approches **non supervisées** (clustering, détection d'anomalies) ne nécessitent pas d'étiquettes : elles apprennent uniquement à partir d'une notion de « normalité » observée dans les données. C'est pourquoi elles sont privilégiées pour détecter des menaces inédites (chapitre 3), au prix d'un taux de fausses alertes généralement plus élevé.
- L'**apprentissage semi-supervisé**, qui combine un petit nombre d'exemples étiquetés avec un grand volume de données non étiquetées, est une piste intermédiaire de plus en plus explorée en sécurité, où l'étiquetage manuel par des experts est coûteux et lent.
- L'apprentissage **par renforcement** reste marginal en production dans la détection elle-même, mais gagne du terrain pour l'automatisation de la réponse à incident (choisir automatiquement une action de remédiation en fonction du contexte) et pour la génération de scénarios d'attaque simulés (« red teaming » automatisé).

## 2. Classification vs régression

- **Classification** : prédire une catégorie (ex. « malveillant » / « bénin »).
- **Régression** : prédire une valeur continue (ex. score de risque entre 0 et 100).

La plupart des cas d'usage en détection de sécurité relèvent de la classification (binaire ou multi-classe).

Il existe cependant des variantes utiles à connaître :

- La **classification multi-classe** ne se contente pas de dire « malveillant/sain » : elle peut viser à identifier la *famille* de malware (ransomware, cheval de Troie, ver, etc.), ce qui aide à prioriser la réponse.
- La **classification multi-label** autorise plusieurs étiquettes simultanées pour une même observation (un même binaire peut à la fois relever d'un cheval de Troie *et* d'un comportement de vol d'identifiants).
- Certains systèmes combinent classification et régression : un modèle de scoring de risque (régression, valeur continue) est ensuite converti en décision binaire (classification) via un **seuil de décision** ajustable — un point essentiel car ce seuil détermine directement le compromis entre précision et rappel (section 5).

## 3. Cycle de vie d'un projet de machine learning

1. **Collecte et préparation des données** : nettoyage, gestion des valeurs manquantes, normalisation.
2. **Ingénierie des caractéristiques (feature engineering)** : transformer des données brutes (paquets réseau, texte) en variables numériques exploitables par un modèle.
3. **Séparation des données** : ensemble d'entraînement, de validation, de test — pour éviter le surapprentissage (*overfitting*).
4. **Entraînement du modèle**.
5. **Évaluation** sur des données jamais vues par le modèle.
6. **Déploiement et supervision continue** (un modèle en production doit être réévalué régulièrement : concept drift).

### 3.1 Illustration avec un pipeline scikit-learn simplifié

L'extrait ci-dessous illustre, de façon pédagogique, l'enchaînement des étapes 2 à 5 sur un jeu de données fictif de connexions réseau caractérisées par quelques variables numériques (durée, nombre d'octets échangés) et une variable catégorielle (protocole). Il ne s'agit pas d'un exemple destiné à être exécuté sur des données réelles, mais d'illustrer la structure typique d'un pipeline :

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

# Jeu de données fictif : chaque ligne = une connexion réseau
# 'label' = 1 si connexion malveillante, 0 sinon (exemple pédagogique)
df = pd.read_csv("connexions_reseau_exemple.csv")

X = df.drop(columns=["label"])
y = df["label"]

# Étape 3 : séparation entraînement / test, en conservant
# la proportion de classes (important sur données déséquilibrées)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Étape 2 : prétraitement des caractéristiques
# - variables numériques : normalisation
# - variable catégorielle (protocole) : encodage one-hot
preprocesseur = ColumnTransformer([
    ("num", StandardScaler(), ["duree", "octets_src", "octets_dst"]),
    ("cat", OneHotEncoder(handle_unknown="ignore"), ["protocole"]),
])

pipeline = Pipeline([
    ("prep", preprocesseur),
    ("modele", RandomForestClassifier(
        n_estimators=200, class_weight="balanced", random_state=42
    )),
])

# Étape 4 : entraînement
pipeline.fit(X_train, y_train)

# Étape 5 : évaluation sur des données jamais vues à l'entraînement
y_pred = pipeline.predict(X_test)
print(classification_report(y_test, y_pred, target_names=["bénin", "malveillant"]))
```

Quelques remarques pédagogiques sur cet extrait :

- L'option `stratify=y` de `train_test_split` garantit que la proportion d'exemples malveillants est similaire dans les ensembles d'entraînement et de test — un point important car, comme discuté en section 5, les classes sont souvent très déséquilibrées en sécurité.
- L'usage d'un `Pipeline` scikit-learn regroupe prétraitement et modèle en un seul objet : cela évite une erreur fréquente en pratique, la **fuite de données (data leakage)**, qui consiste par exemple à calculer la normalisation (moyenne, écart-type) sur l'ensemble complet des données *avant* la séparation train/test, ce qui laisse involontairement filtrer de l'information de l'ensemble de test vers l'entraînement et surestime la performance réelle du modèle.
- Le paramètre `class_weight="balanced"` demande au modèle de pénaliser davantage les erreurs sur la classe minoritaire (malveillant), une réponse simple — mais pas toujours suffisante — au déséquilibre des classes.

### 3.2 Validation croisée : au-delà d'une seule séparation train/test

Une unique séparation entraînement/test peut donner une estimation optimiste ou pessimiste de la performance selon le hasard du découpage, en particulier sur un jeu de données de taille modeste. La **validation croisée à k plis (k-fold cross-validation)** répète l'entraînement et l'évaluation k fois sur des découpages différents, puis moyenne les résultats, donnant une estimation plus robuste :

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(pipeline, X, y, cv=5, scoring="f1")
print(f"F1-score moyen sur 5 plis : {scores.mean():.3f} (écart-type {scores.std():.3f})")
```

En sécurité, une variante appelée **validation croisée temporelle** est souvent préférable à un découpage aléatoire classique : les plis d'entraînement doivent précéder chronologiquement les plis d'évaluation (par exemple via `TimeSeriesSplit` de scikit-learn), afin de simuler fidèlement une situation réelle de déploiement, où le modèle est entraîné sur du trafic passé et évalué sur du trafic futur — un découpage purement aléatoire pourrait sinon entraîner le modèle sur des données « du futur » par rapport à son test, ce qui gonflerait artificiellement la performance estimée.

## 4. Quelques algorithmes classiques

| Algorithme | Type | Caractéristique |
|---|---|---|
| Régression logistique | supervisé, classification | simple, interprétable, bonne base de référence |
| Arbre de décision / Forêt aléatoire (Random Forest) | supervisé, classification/régression | robuste, gère bien les données hétérogènes, relativement interprétable |
| SVM (Support Vector Machine) | supervisé, classification | efficace en haute dimension |
| K-means | non supervisé, clustering | regroupe des observations similaires |
| Isolation Forest | non supervisé, détection d'anomalies | isole les observations atypiques efficacement, utilisé en TP2 |
| Réseaux de neurones / Deep Learning | supervisé ou non supervisé | performant sur données complexes (texte, image, séquences) mais moins interprétable |

### 4.1 Intuition sur quelques algorithmes clés

Comprendre le principe géométrique ou statistique sous-jacent aide à choisir le bon algorithme pour un problème donné, plutôt que de traiter chaque modèle comme une boîte noire interchangeable :

- **Régression logistique** : modélise la probabilité qu'une observation appartienne à la classe positive comme une fonction sigmoïde d'une combinaison linéaire des variables d'entrée. Son atout majeur est l'interprétabilité : le poids associé à chaque variable indique directement, toutes choses égales par ailleurs, son influence sur la prédiction — un avantage précieux pour justifier une alerte auprès d'un analyste SOC.
- **Arbres de décision et forêts aléatoires** : un arbre de décision découpe successivement l'espace des caractéristiques par des règles simples (« si `octets_dst > 5000` alors... »), ce qui le rend très lisible mais sujet au surapprentissage sur un arbre unique. Une **forêt aléatoire** entraîne un grand nombre d'arbres sur des sous-échantillons différents des données et des variables, puis agrège leurs prédictions (vote majoritaire) : elle gagne en robustesse et en performance ce qu'elle perd en lisibilité immédiate, tout en restant plus interprétable qu'un réseau de neurones (on peut par exemple mesurer l'**importance des variables**).
- **SVM** : cherche l'hyperplan séparant les deux classes avec la marge maximale, éventuellement après projection dans un espace de dimension supérieure via une fonction noyau (*kernel*) pour traiter des frontières non linéaires. Efficace lorsque le nombre de variables est grand par rapport au nombre d'observations, mais son entraînement passe mal à l'échelle sur de très gros volumes de données, ce qui limite son usage sur des flux réseau massifs.
- **K-means** : partitionne les observations en *k* groupes en minimisant la distance de chaque point au centre (centroïde) de son groupe. Simple et rapide, mais suppose des groupes de forme sphérique et de taille comparable, et nécessite de choisir *k* à l'avance — une limite importante en détection d'anomalies, où le nombre de comportements distincts n'est pas connu d'avance.
- **Isolation Forest** : contrairement aux approches basées sur une notion de distance ou de densité, elle exploite l'idée qu'une observation atypique est plus facile à « isoler » du reste des données par des séparations aléatoires successives (elle nécessite en moyenne moins de coupures pour être isolée seule dans une branche de l'arbre). Cette propriété en fait un algorithme rapide et bien adapté à la détection d'anomalies en haute dimension, ce qui explique son usage fréquent en sécurité (approfondi au chapitre 3 et pratiqué en TP2).
- **Réseaux de neurones** : composés de couches de neurones artificiels reliés par des poids ajustés lors de l'entraînement (rétropropagation du gradient), ils peuvent approximer des fonctions très complexes et non linéaires. Leur force réside dans le traitement de données non structurées (texte brut, séquences d'appels système, images), mais ils nécessitent généralement plus de données et de puissance de calcul, et leur fonctionnement interne reste largement opaque — un inconvénient dans un contexte où l'explicabilité d'une décision de sécurité peut être requise.

### 4.2 Comment choisir un algorithme en pratique ?

Il n'existe pas d'algorithme universellement supérieur (« no free lunch theorem ») : le choix dépend du contexte. Quelques critères pratiques à considérer, particulièrement pertinents en sécurité :

- **Volume et nature des données disponibles** : peu de données étiquetées → privilégier des modèles simples (régression logistique, arbres) moins sujets au surapprentissage ; beaucoup de données non structurées (texte, séquences) → le deep learning devient compétitif.
- **Besoin d'explicabilité** : un contexte réglementaire ou une nécessité de justifier une alerte auprès d'un analyste pousse vers des modèles interprétables (arbres, régression logistique) ou vers l'usage de techniques d'explicabilité post-hoc (par exemple SHAP) sur des modèles plus complexes.
- **Contraintes de latence** : un IDS traitant du trafic en temps réel ne peut pas se permettre un modèle dont l'inférence est trop lente ; un modèle plus simple, quitte à être légèrement moins performant, peut être préférable en production.
- **Robustesse aux données déséquilibrées** : certains algorithmes (forêts aléatoires, gradient boosting) offrent des mécanismes natifs de pondération des classes plus simples à exploiter que d'autres.

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

### 5.1 Le compromis précision/rappel et le rôle du seuil de décision

La plupart des classifieurs ne produisent pas directement une décision binaire, mais une probabilité (ou un score) qu'une observation appartienne à la classe positive. Cette probabilité est ensuite comparée à un **seuil de décision** (souvent 0,5 par défaut) pour produire l'étiquette finale. Faire varier ce seuil déplace le compromis entre précision et rappel :

- **Abaisser le seuil** (déclencher une alerte plus facilement) augmente généralement le rappel (moins de menaces manquées) mais diminue la précision (davantage de fausses alertes).
- **Relever le seuil** fait l'inverse : moins de fausses alertes, mais un risque accru de laisser passer de véritables menaces.

Le choix du seuil optimal dépend du **coût relatif** des deux types d'erreur dans le contexte opérationnel considéré : dans un SOC déjà saturé d'alertes, un seuil plus élevé peut être préféré pour limiter la fatigue d'alerte, quitte à accepter un rappel légèrement inférieur ; à l'inverse, sur un système critique (infrastructure de santé, contrôle industriel), un rappel maximal peut être priorisé malgré un volume d'alertes plus important à traiter.

Deux outils graphiques aident à visualiser ce compromis sur l'ensemble des seuils possibles, plutôt qu'à un seuil fixé arbitrairement :

- La **courbe ROC** (*Receiver Operating Characteristic*) trace le taux de vrais positifs en fonction du taux de faux positifs pour tous les seuils, et l'**AUC** (aire sous la courbe) résume la capacité globale du modèle à séparer les classes, indépendamment d'un seuil particulier.
- La **courbe précision-rappel** est généralement plus informative que la courbe ROC lorsque les classes sont très déséquilibrées, situation fréquente en sécurité, car elle reste sensible aux performances sur la classe rare (l'attaque).

```python
from sklearn.metrics import precision_recall_curve, roc_auc_score
import numpy as np

# proba : probabilité prédite d'appartenance à la classe "malveillant"
proba = pipeline.predict_proba(X_test)[:, 1]

precisions, rappels, seuils = precision_recall_curve(y_test, proba)
print("AUC ROC :", roc_auc_score(y_test, proba))

# Exemple : choisir le seuil qui maximise le F1-score sur l'ensemble de validation
f1_scores = 2 * (precisions * rappels) / (precisions + rappels + 1e-9)
meilleur_seuil = seuils[np.argmax(f1_scores)]
print("Seuil retenu :", meilleur_seuil)
```

### 5.2 Autres métriques utiles en contexte opérationnel

Au-delà de la précision, du rappel et du F1-score, un praticien de la sécurité gagne à connaître :

- **Le taux de fausses alertes en volume absolu** (et non seulement en proportion) : sur un très grand volume d'événements, même un taux de faux positifs de 0,1 % peut représenter des centaines d'alertes quotidiennes à traiter manuellement — une information que le pourcentage seul ne rend pas visible.
- **Le délai de détection (time to detect)** : au-delà de la seule justesse de la prédiction, le temps écoulé entre le début d'une attaque et sa détection est un critère opérationnel de premier plan, en particulier pour les attaques à progression rapide (ransomware).
- **La matrice de confusion multi-classe**, lorsque le problème dépasse la simple distinction bénin/malveillant (ex. identification de la famille de malware), permettant d'identifier quelles classes sont le plus souvent confondues entre elles.

## 6. Surapprentissage et sous-apprentissage

- **Surapprentissage (overfitting)** : le modèle « apprend par cœur » les données d'entraînement et généralise mal à de nouvelles données.
- **Sous-apprentissage (underfitting)** : le modèle est trop simple pour capturer la structure des données.

La validation croisée (cross-validation) et un ensemble de test totalement indépendant permettent de détecter ces situations avant tout déploiement.

### 6.1 Diagnostiquer et corriger ces situations

En pratique, on distingue le surapprentissage et le sous-apprentissage en comparant la performance du modèle sur l'ensemble d'entraînement et sur l'ensemble de validation :

- Une performance **élevée à l'entraînement mais nettement plus faible en validation** est le symptôme classique du surapprentissage : le modèle a mémorisé des particularités propres à l'échantillon d'entraînement (y compris du bruit) qui ne se généralisent pas.
- Une performance **médiocre à la fois à l'entraînement et en validation** signale un sous-apprentissage : le modèle est trop contraint (trop simple, ou insuffisamment entraîné) pour capturer la structure réelle des données.

Quelques leviers pratiques pour remédier à ces situations, fréquemment combinés :

- Contre le **surapprentissage** : simplifier le modèle (réduire la profondeur d'un arbre, le nombre de couches d'un réseau de neurones), augmenter le volume de données d'entraînement, utiliser une **régularisation** (pénaliser les modèles trop complexes, par exemple via les pénalités L1/L2 en régression logistique), ou appliquer un **arrêt précoce** (*early stopping*) pendant l'entraînement d'un réseau de neurones.
- Contre le **sous-apprentissage** : complexifier le modèle, enrichir l'ingénierie des caractéristiques (chapitres 3 à 5), ou entraîner plus longtemps.

Une bonne pratique consiste à tracer les **courbes d'apprentissage** (performance en fonction de la taille de l'ensemble d'entraînement, ou du nombre d'itérations) pour visualiser directement lequel de ces deux régimes s'applique, plutôt que de deviner uniquement à partir d'un score final.

## À retenir

- Le choix entre supervisé/non supervisé dépend de la disponibilité de données étiquetées, souvent rare en sécurité pour les attaques nouvelles.
- Précision, rappel et F1-score, pas la seule exactitude, sont les métriques pertinentes pour évaluer un modèle de détection en contexte déséquilibré.
- Le seuil de décision permet d'ajuster le compromis précision/rappel selon le coût opérationnel des faux positifs et des faux négatifs dans le contexte visé.
- Le cycle de vie complet (données → features → entraînement → évaluation → supervision continue) doit être maîtrisé, pas seulement l'entraînement du modèle.
- En sécurité, la validation croisée temporelle est souvent préférable à un découpage aléatoire, afin de simuler fidèlement un déploiement réel sur des données futures inconnues du modèle.
- Le surapprentissage et le sous-apprentissage se diagnostiquent en comparant performance à l'entraînement et en validation, et se corrigent par des leviers différents et parfois opposés.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
