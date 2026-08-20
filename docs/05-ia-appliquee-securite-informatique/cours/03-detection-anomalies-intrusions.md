# Chapitre 3 — Détection d'anomalies et d'intrusions par ML

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=05-ia-appliquee-securite-informatique/slides/03-detection-anomalies-intrusions.txt){ target=_blank }

## 1. Détection par signatures vs détection par anomalies

| Approche | Principe | Avantage | Limite |
|---|---|---|---|
| **Signatures** | comparaison à des motifs d'attaques connues | très faible taux de faux positifs sur les menaces connues | ne détecte pas les attaques inédites |
| **Anomalies (ML)** | apprentissage du comportement « normal », alerte sur tout écart significatif | peut détecter des menaces nouvelles (zero-day) | taux de faux positifs souvent plus élevé, nécessite une phase d'apprentissage du « normal » |

En pratique, les IDS/IPS (systèmes de détection/prévention d'intrusion) modernes combinent les deux approches.

### 1.1 Un troisième pilier : la détection par spécification

Une troisième approche, moins souvent mise en avant mais utile à connaître, complète ce panorama : la **détection par spécification (specification-based)**. Elle consiste à définir manuellement les comportements *autorisés* d'un système (par exemple, les commandes qu'un protocole industriel est censé accepter) et à considérer comme suspect tout ce qui s'en écarte. Contrairement à la détection par anomalies, elle ne repose pas sur un apprentissage statistique mais sur une modélisation experte du comportement légitime — elle est donc moins sujette aux faux positifs liés au bruit statistique, mais coûteuse à maintenir et peu adaptable à des systèmes complexes ou évolutifs. Elle reste néanmoins pertinente dans des environnements très contraints (systèmes industriels SCADA, objets connectés à fonction unique) où le comportement légitime est simple à spécifier exhaustivement.

### 1.2 Pourquoi la détection par anomalies séduit malgré son coût en faux positifs

Le principal argument en faveur de la détection par anomalies, malgré ses limites, tient à la nature même des attaques modernes : une part croissante des compromissions n'exploite pas de vulnérabilité logicielle « signable » au sens classique, mais des techniques dites *living-off-the-land* (utilisation d'outils légitimes déjà présents sur le système à des fins malveillantes) ou des identifiants compromis via ingénierie sociale. Dans ces cas, il n'existe littéralement aucune signature statique à détecter : seul un écart de comportement (un compte utilisateur se connectant à une heure inhabituelle, un outil d'administration exécuté depuis un poste qui ne l'utilise jamais) peut révéler l'intrusion. C'est ce constat qui justifie l'investissement dans des approches par anomalies malgré le coût opérationnel plus élevé en gestion des fausses alertes.

## 2. Sources de données pour un IDS basé ML

- **Trafic réseau** : caractéristiques dérivées des flux (NetFlow) : durée de connexion, protocole, nombre d'octets échangés, nombre de paquets, taux d'erreur.
- **Journaux systèmes/applicatifs** : séquences d'événements, fréquence des connexions échouées.
- **Comportement utilisateur (UEBA)** : horaires de connexion habituels, ressources accédées, volume de données téléchargées.

### 2.1 Granularité de l'analyse : paquet, flux ou session

Le choix de la granularité d'analyse conditionne fortement le type de menaces détectables et le coût de calcul associé :

- **Analyse par paquet (deep packet inspection)** : la plus fine, elle permet d'inspecter le contenu même des paquets, mais génère un volume de données considérable et pose des questions de confidentialité (inspection du contenu, parfois chiffré).
- **Analyse par flux (NetFlow/IPFIX)** : agrège les paquets d'une même connexion en un enregistrement résumé (durée, volume, protocole), largement utilisée en pratique car elle réduit fortement le volume de données à traiter tout en conservant l'essentiel du signal utile à la détection.
- **Analyse par session ou par entité** : agrège encore davantage, à l'échelle d'un utilisateur ou d'une machine sur une fenêtre de temps (ex. nombre total de connexions échouées sur une heure), ce qui est la granularité typique de l'UEBA.

Le compromis est le suivant : plus la granularité est fine, plus l'information disponible est riche, mais plus le volume de données et le coût de calcul augmentent, et plus l'entraînement du modèle nécessite de données pour éviter le surapprentissage sur des détails non pertinents (chapitre 2).

## 3. Ingénierie des caractéristiques pour le trafic réseau

Exemple de caractéristiques dérivées classiques (utilisées dans le jeu de données NSL-KDD, réutilisé en TP1) :

- durée de la connexion ;
- type de protocole (TCP, UDP, ICMP) ;
- service ciblé (HTTP, FTP, SSH…) ;
- nombre d'octets source→destination et destination→source ;
- nombre de tentatives de connexion échouées récentes vers le même hôte ;
- taux de connexions vers un même service dans une fenêtre temporelle glissante.

Le passage de paquets réseau bruts à ce type de caractéristiques agrégées est une étape déterminante : un modèle ne peut être meilleur que les caractéristiques qui lui sont fournies.

### 3.1 Caractéristiques « de contenu » vs caractéristiques « de trafic »

Le jeu NSL-KDD, comme la plupart des jeux de données classiques pour la détection d'intrusion, distingue généralement plusieurs familles de variables, une distinction utile à retenir au-delà du jeu de données lui-même :

- **Caractéristiques de base (basic features)** : issues directement de l'en-tête de la connexion (durée, protocole, service, drapeaux TCP, volume d'octets).
- **Caractéristiques de contenu (content features)** : dérivées du contenu même de la connexion lorsqu'il est accessible, utiles notamment pour repérer des attaques applicatives (ex. nombre de tentatives de connexion échouées dans une session, indicateurs de commandes suspectes).
- **Caractéristiques de trafic sur fenêtre temporelle (traffic features)** : calculées sur une fenêtre glissante de connexions récentes (par exemple les deux dernières secondes), et particulièrement utiles pour détecter des attaques par balayage (scan) ou par déni de service, qui se manifestent par une répétition rapide de connexions similaires plutôt que par une connexion isolée suspecte.

### 3.2 Exemple pédagogique d'extraction de caractéristiques

L'extrait suivant illustre, de façon simplifiée, comment calculer une caractéristique de type « fenêtre glissante » (nombre de connexions échouées vers le même hôte dans les deux dernières secondes) à partir d'un journal de connexions déjà horodaté — un exemple pédagogique destiné à illustrer le principe, non un code de production :

```python
import pandas as pd

# 'log' : DataFrame fictif avec colonnes ['horodatage', 'hote_dest', 'echec']
log = pd.read_csv("journal_connexions_exemple.csv", parse_dates=["horodatage"])
log = log.sort_values("horodatage").set_index("horodatage")

def compte_echecs_fenetre(sous_journal, hote):
    """Nombre de connexions échouées vers `hote` dans la fenêtre courante."""
    return sous_journal[
        (sous_journal["hote_dest"] == hote) & (sous_journal["echec"] == 1)
    ].shape[0]

# Fenêtre glissante de 2 secondes : pour chaque connexion, on compte
# les échecs récents vers le même hôte, une caractéristique classique
# des jeux de type NSL-KDD/CICIDS.
resultats = []
for horodatage, ligne in log.iterrows():
    fenetre = log.loc[horodatage - pd.Timedelta(seconds=2): horodatage]
    resultats.append(compte_echecs_fenetre(fenetre, ligne["hote_dest"]))

log["echecs_recents_meme_hote"] = resultats
```

Ce type de caractéristique dérivée capture une signature comportementale (une rafale de tentatives échouées) invisible dans une connexion isolée examinée seule, ce qui illustre bien pourquoi l'ingénierie des caractéristiques est souvent plus déterminante pour la performance finale que le choix de l'algorithme lui-même.

## 4. Approches non supervisées de détection d'anomalies

- **Clustering (K-means, DBSCAN)** : les points isolés ou dans de petits clusters sont considérés suspects.
- **Isolation Forest** : construit des arbres aléatoires ; une observation atypique nécessite en moyenne moins de séparations pour être isolée, ce qui produit un score d'anomalie.
- **Autoencodeurs (réseaux de neurones)** : le modèle apprend à reconstruire les données normales ; une erreur de reconstruction élevée sur une nouvelle observation signale une anomalie potentielle.

### 4.1 Exemple pédagogique : Isolation Forest avec scikit-learn

```python
from sklearn.ensemble import IsolationForest
from sklearn.preprocessing import StandardScaler

# X : matrice de caractéristiques numériques (durée, octets, taux d'erreur, ...)
# déjà extraites selon les principes de la section 3
scaler = StandardScaler()
X_norm = scaler.fit_transform(X)

# contamination : proportion attendue d'anomalies dans les données
# (à ajuster selon le contexte ; ici une valeur pédagogique arbitraire)
detecteur = IsolationForest(
    n_estimators=200, contamination=0.02, random_state=42
)
detecteur.fit(X_norm)

# decision_function : plus le score est faible/négatif, plus
# l'observation est considérée comme anormale
scores_anomalie = detecteur.decision_function(X_norm)
predictions = detecteur.predict(X_norm)  # -1 = anomalie, 1 = normal
```

Le paramètre `contamination` mérite une attention particulière : il fixe a priori la proportion attendue d'anomalies dans les données et influence directement le seuil de décision appliqué au score. En contexte réel, cette proportion est rarement connue avec précision — un choix trop faible sous-détecte les attaques, un choix trop élevé génère un excès de fausses alertes. Une pratique courante consiste à traiter le score continu produit par `decision_function` comme un signal de priorisation pour l'analyste plutôt qu'à figer une décision binaire automatique, notamment lors des premiers déploiements d'un tel système.

### 4.2 Autoencodeurs : intuition et limites

Un autoencodeur est un réseau de neurones entraîné à reproduire en sortie ce qu'il reçoit en entrée, en passant par une couche intermédiaire de dimension réduite (le « goulot d'étranglement »). Entraîné uniquement sur du trafic normal, il apprend une représentation compacte des régularités de ce trafic ; face à une observation qui s'écarte de ces régularités (une attaque), il reconstruit moins bien l'entrée, produisant une **erreur de reconstruction** plus élevée, utilisée comme score d'anomalie.

Cette approche présente un intérêt particulier pour des données de haute dimension ou fortement non linéaires (ex. séquences temporelles de trafic), là où K-means ou Isolation Forest peuvent être moins efficaces. Elle comporte cependant des limites propres :

- Elle nécessite un volume de données d'entraînement nettement plus important que les approches classiques pour converger correctement.
- Le choix de l'architecture (taille du goulot d'étranglement, profondeur du réseau) est déterminant et nécessite une phase d'expérimentation ; un goulot trop large permet au modèle de « tricher » en apprenant presque une fonction identité, réduisant sa capacité de détection.
- Comme tout réseau de neurones profond, elle hérite des enjeux d'explicabilité et de vulnérabilité aux attaques adversariales évoqués au chapitre 6.

## 5. Approches supervisées

Lorsque des données étiquetées (trafic normal vs attaques connues, ex. jeux NSL-KDD ou CICIDS) sont disponibles, une classification supervisée (Random Forest, SVM, réseaux de neurones) obtient généralement de meilleures performances qu'une approche purement non supervisée, mais reste limitée aux catégories d'attaques représentées dans les données d'entraînement.

### 5.1 Approches hybrides

En pratique, une architecture fréquemment rencontrée combine les deux paradigmes plutôt que de choisir l'un contre l'autre :

1. Un module **non supervisé** (clustering ou Isolation Forest) filtre en amont un large volume de trafic pour isoler les observations statistiquement atypiques, réduisant considérablement le volume à analyser plus finement.
2. Un module **supervisé**, entraîné sur des catégories d'attaques connues, classifie plus précisément les observations retenues (type d'attaque probable), aidant à la priorisation par l'analyste.

Cette architecture en cascade tire parti de la capacité des approches non supervisées à couvrir des menaces inconnues, tout en bénéficiant de la précision et de la richesse d'information (type d'attaque, niveau de confiance) qu'apporte un modèle supervisé sur les catégories déjà documentées.

## 6. Défis pratiques du déploiement en production

- **Déséquilibre des classes** : les attaques représentent une infime fraction du trafic total ; des techniques de rééquilibrage (sur-échantillonnage, sous-échantillonnage, SMOTE) ou des métriques adaptées (chapitre 2) sont nécessaires.
- **Dérive conceptuelle (concept drift)** : le trafic « normal » évolue dans le temps (nouveaux usages, nouvelles applications), un modèle doit être réentraîné périodiquement.
- **Volumétrie et latence** : un IDS doit souvent traiter des flux en temps réel, ce qui contraint le choix et la complexité du modèle.
- **Intégration avec le SOC** : les alertes générées doivent être exploitables par des analystes humains (contexte, explicabilité, priorisation).

### 6.1 SMOTE et ses limites

La technique **SMOTE** (*Synthetic Minority Over-sampling Technique*), fréquemment citée pour traiter le déséquilibre des classes, génère de nouveaux exemples synthétiques de la classe minoritaire en interpolant entre des exemples existants proches dans l'espace des caractéristiques, plutôt que de simplement dupliquer les exemples réels (sur-échantillonnage naïf). Bien que largement utilisée, elle présente des limites à connaître avant application :

- Elle peut générer des exemples synthétiques peu réalistes si la classe minoritaire est elle-même composée de sous-groupes très différents (par exemple, plusieurs types d'attaques très distincts regroupés sous une même étiquette « malveillant »), en créant des points intermédiaires qui ne correspondent à aucun comportement d'attaque réel.
- Elle doit impérativement être appliquée **après** la séparation entraînement/test (et recalculée à chaque pli en validation croisée), sous peine de fuite de données : générer des exemples synthétiques avant le découpage risque de placer des points très proches à la fois dans l'ensemble d'entraînement et de test, biaisant l'évaluation.
- Elle ne résout pas le problème de fond si les caractéristiques disponibles ne permettent tout simplement pas de distinguer statistiquement les deux classes.

### 6.2 Explicabilité opérationnelle : au-delà du score

Un défi pratique souvent sous-estimé est la nécessité de fournir à l'analyste SOC davantage qu'un simple score numérique. Une alerte accompagnée des caractéristiques ayant le plus contribué à la décision (par exemple via l'importance des variables d'une forêt aléatoire, ou des techniques post-hoc comme SHAP pour des modèles plus complexes) permet à l'analyste de valider rapidement la pertinence de l'alerte, d'écarter un faux positif évident, ou de comprendre le vecteur d'attaque probable sans devoir ré-analyser manuellement l'ensemble du trafic concerné. Cette dimension d'**explicabilité opérationnelle** est aussi importante pour l'adoption réelle d'un système par les équipes de sécurité que la performance brute du modèle.

## À retenir

- Les systèmes de détection combinent en général signatures (précis mais limités aux menaces connues) et ML par anomalies (couvre potentiellement les menaces nouvelles, au prix de plus de faux positifs).
- La détection par anomalies est particulièrement pertinente contre les attaques ne laissant pas de signature statique exploitable (living-off-the-land, identifiants compromis).
- La qualité des caractéristiques extraites du trafic brut conditionne fortement la performance du modèle ; la granularité d'analyse (paquet, flux, session) est un choix structurant.
- Les architectures hybrides, combinant filtrage non supervisé et classification supervisée en cascade, tirent parti des forces respectives des deux paradigmes.
- Le déséquilibre des classes et la dérive conceptuelle sont deux défis pratiques majeurs, souvent sous-estimés lors du passage du prototype à la production ; des techniques comme SMOTE aident mais ne remplacent pas des caractéristiques de qualité.
- L'explicabilité des alertes auprès des analystes est un facteur clé d'adoption opérationnelle d'un IDS basé ML.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
