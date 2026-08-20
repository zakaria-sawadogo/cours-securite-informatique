# Chapitre 5 — NLP appliqué à la sécurité (phishing, analyse de logs)

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=05-ia-appliquee-securite-informatique/slides/05-nlp-securite.txt){ target=_blank }

## 1. Le traitement automatique du langage naturel (NLP) en cybersécurité

Le NLP permet d'analyser automatiquement des contenus textuels : e-mails, messages, URL, journaux d'événements, rapports de threat intelligence. En sécurité, ses deux applications les plus répandues sont la détection de phishing et l'analyse assistée de journaux volumineux.

### 1.1 Pourquoi le texte est un signal particulièrement riche mais difficile

Le texte présente une combinaison de propriétés qui le rend à la fois précieux et délicat à exploiter pour la sécurité :

- **Richesse sémantique** : un message de phishing peut être identifié par son intention (créer un sentiment d'urgence, inciter à une action) bien au-delà de simples mots-clés, ce qu'aucune règle statique simple ne peut capturer exhaustivement.
- **Variabilité et créativité linguistique** : contrairement à un flux réseau structuré (chapitre 3) ou à un binaire (chapitre 4), le langage naturel offre une infinité de façons d'exprimer la même intention, ce qui complique la détection par simples règles ou motifs.
- **Manipulation délibérée par l'attaquant** : un attaquant en phishing adapte activement son texte pour éviter les filtres, un phénomène de « jeu du chat et de la souris » qui rapproche ce domaine de la détection de malwares (chapitre 4) plus que d'applications NLP classiques (analyse de sentiment sur des avis clients, par exemple), où le texte analysé n'a pas d'intention adverse.

## 2. Représentation du texte pour le machine learning

Un modèle ML ne manipule pas directement du texte : il faut le transformer en représentation numérique.

| Méthode | Principe | Caractéristique |
|---|---|---|
| **Bag-of-words / TF-IDF** | compte (pondéré) l'occurrence des mots, sans tenir compte de l'ordre | simple, efficace pour des tâches de classification classiques |
| **N-grammes de caractères** | séquences de *n* caractères consécutifs | robuste aux fautes d'orthographe volontaires (technique d'évasion courante en phishing) |
| **Word embeddings** (Word2Vec, GloVe) | représentation vectorielle dense capturant des relations sémantiques entre mots | capture le sens, plus coûteux à entraîner |
| **Modèles de langage pré-entraînés (Transformers, type BERT)** | représentation contextuelle, état de l'art actuel | performance élevée, coût de calcul plus important |

### 2.1 TF-IDF en détail

Le TF-IDF (*Term Frequency – Inverse Document Frequency*) pondère chaque mot d'un document selon deux composantes complémentaires :

- **TF (fréquence du terme)** : plus un mot apparaît fréquemment dans un document donné, plus il est susceptible d'être significatif pour ce document.
- **IDF (fréquence inverse de document)** : un mot apparaissant dans presque tous les documents du corpus (comme les mots grammaticaux très courants) apporte peu d'information discriminante et voit son poids réduit ; à l'inverse, un mot rare mais présent dans peu de documents est jugé plus informatif.

Formellement, pour un terme $t$ et un document $d$ appartenant à un corpus de $N$ documents :

$$\text{TF-IDF}(t, d) = \text{TF}(t, d) \times \log\left(\frac{N}{\text{DF}(t)}\right)$$

où $\text{DF}(t)$ est le nombre de documents du corpus contenant le terme $t$. Cette pondération explique pourquoi TF-IDF reste une base de référence solide en classification de texte malgré son ancienneté : elle capture, avec un coût de calcul minime, l'idée intuitive qu'un mot rare et répété dans un message donné (par exemple, un nom de marque usurpée répété plusieurs fois dans un e-mail de phishing) est plus significatif qu'un mot omniprésent dans tout le corpus.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline

# corpus_emails : liste de textes d'e-mails (exemple pédagogique)
# labels : 1 = phishing, 0 = légitime
vectoriseur = TfidfVectorizer(
    lowercase=True,
    ngram_range=(1, 2),   # unigrammes et bigrammes de mots
    max_features=20000,   # limite le vocabulaire aux termes les plus fréquents
    stop_words=None,      # attention : ne pas retirer aveuglément les mots
                           # vides en phishing, certains (« urgent », « cliquez »)
                           # sont eux-mêmes des indicateurs utiles
)

pipeline = make_pipeline(vectoriseur, LogisticRegression(max_iter=1000))
pipeline.fit(corpus_emails, labels)

# Inspection des mots les plus associés à la classe "phishing"
import numpy as np
vocab = vectoriseur.get_feature_names_out()
poids = pipeline.named_steps["logisticregression"].coef_[0]
top_indices = np.argsort(poids)[-15:]
print("Termes les plus associés au phishing :", vocab[top_indices])
```

Une remarque pédagogique importante : dans les tâches NLP « génériques », il est courant de retirer les **mots vides** (*stop words* : « le », « de », « et »…), considérés comme peu informatifs. En détection de phishing, cette pratique doit être appliquée avec prudence : certains mots très fréquents (« urgent », « cliquez », « vérifier », « suspendu ») sont au contraire des indicateurs discriminants forts, et les retirer par un filtre générique de mots vides risquerait de supprimer un signal utile.

### 2.2 N-grammes de caractères : robustesse à l'évasion orthographique

Une limite du bag-of-words au niveau du mot est sa fragilité face à de simples variations orthographiques volontaires — une technique d'évasion très répandue en phishing et en spam (par exemple remplacer « virement » par « v1rement » ou insérer des espaces ou caractères invisibles au milieu d'un mot-clé filtré). Les n-grammes de *caractères* (plutôt que de mots) offrent une robustesse partielle à ce type de manipulation, car une grande partie des sous-séquences de caractères reste inchangée malgré la modification :

```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectoriseur_car = TfidfVectorizer(
    analyzer="char_wb",   # n-grammes de caractères, respectant les limites de mots
    ngram_range=(3, 5),
    max_features=20000,
)
```

Ce type de représentation est également largement utilisé pour la détection d'URL malveillantes (section 4), où l'unité « mot » est souvent moins pertinente que les sous-chaînes de caractères composant le nom de domaine.

### 2.3 Des embeddings statiques aux Transformers : ce que le contexte apporte

Les méthodes de type Word2Vec ou GloVe associent à chaque mot un vecteur **fixe**, indépendant du contexte dans lequel il apparaît : le mot « virus » aura toujours le même vecteur, qu'il apparaisse dans un contexte informatique ou médical. Les modèles de langage pré-entraînés de type Transformer (BERT et ses variantes) produisent au contraire une représentation **contextuelle** : le vecteur associé à un mot dépend des mots qui l'entourent dans la phrase, ce qui permet de désambiguïser le sens et de capturer des relations syntaxiques et sémantiques plus fines (négation, ironie, structure de phrase).

Ce gain de finesse a un coût : l'entraînement (ou même le simple usage en inférence) de ces modèles est nettement plus coûteux en calcul que TF-IDF ou les embeddings statiques, ce qui doit être mis en balance avec les contraintes opérationnelles (latence, volume de messages à traiter en temps réel) d'un système de filtrage de messagerie à grande échelle. En pratique, il est fréquent de partir d'un modèle pré-entraîné sur un vaste corpus généraliste, puis de l'affiner (*fine-tuning*) sur un corpus spécifique à la tâche de détection de phishing — une approche de **transfert d'apprentissage** qui réduit fortement le volume de données étiquetées nécessaire par rapport à un entraînement entièrement from scratch.

## 3. Détection de phishing par e-mail

Caractéristiques exploitées, combinant NLP et méta-données :

- contenu textuel du message (urgence, demande d'action, fautes, incitation au clic) ;
- caractéristiques de l'expéditeur (domaine récemment enregistré, usurpation d'affichage — *display name spoofing*) ;
- caractéristiques des URL contenues (longueur, présence de caractères trompeurs, similarité avec un domaine légitime — *typosquatting*) ;
- en-têtes techniques du message (résultats SPF/DKIM/DMARC, incohérences de routage).

Une approche efficace combine généralement un modèle NLP sur le contenu du message et des règles/caractéristiques dédiées à l'analyse des URL et des en-têtes, plutôt qu'un seul modèle « texte brut ».

### 3.1 Pourquoi une approche multi-signaux surpasse un modèle « texte seul »

Cette combinaison n'est pas un simple choix d'ingénierie : elle répond à une limite structurelle du texte seul comme signal de détection. Un attaquant sophistiqué peut produire un texte irréprochable, sans faute et parfaitement crédible (d'autant plus facilement aujourd'hui avec l'assistance de modèles de langage génératifs, voir section 6), rendant les indicateurs purement textuels insuffisants. À l'inverse, les caractéristiques techniques — authenticité du domaine expéditeur, cohérence des enregistrements SPF/DKIM/DMARC, âge d'enregistrement du nom de domaine — sont beaucoup plus difficiles à falsifier pour un attaquant, car elles dépendent d'une infrastructure qu'il ne contrôle pas entièrement (registres de noms de domaine, autorités de certification, réputation historique).

Un système robuste combine ainsi typiquement plusieurs modèles ou caractéristiques en un **score composite** :

```python
# Exemple pédagogique simplifié d'agrégation de plusieurs signaux
# en un score de risque de phishing (les poids sont illustratifs)

def score_risque_phishing(email):
    score_texte = modele_nlp.predict_proba([email.corps])[0][1]        # 0 à 1
    score_url = modele_url.predict_proba([email.urls_extraites])[0][1]  # 0 à 1
    domaine_recent = 1.0 if email.age_domaine_jours < 30 else 0.0
    auth_echouee = 1.0 if not email.spf_dkim_dmarc_ok else 0.0

    score_final = (
        0.4 * score_texte
        + 0.3 * score_url
        + 0.15 * domaine_recent
        + 0.15 * auth_echouee
    )
    return score_final
```

Cet exemple, volontairement simplifié (les poids réels seraient déterminés par apprentissage plutôt que fixés arbitrairement, par exemple via un modèle de stacking combinant les scores des sous-modèles comme nouvelles caractéristiques d'un modèle final), illustre le principe de défense en profondeur appliqué au filtrage anti-phishing : aucun signal isolé n'est fiable à lui seul, mais leur combinaison réduit fortement les angles morts de chaque approche prise individuellement.

## 4. Détection d'URL malveillantes

Approche fréquente : classification supervisée à partir de caractéristiques lexicales de l'URL elle-même (longueur, nombre de sous-domaines, présence de caractères spéciaux, entropie), sans nécessairement visiter la page cible — utile pour un filtrage rapide en amont.

### 4.1 Catégories de caractéristiques d'URL

Au-delà des caractéristiques purement lexicales mentionnées, on distingue généralement plusieurs familles de caractéristiques exploitées en détection d'URL, chacune capturant un aspect différent du risque :

- **Caractéristiques lexicales** : longueur totale de l'URL, nombre de sous-domaines, présence de caractères inhabituels (`@`, tirets multiples), usage d'une adresse IP brute à la place d'un nom de domaine, entropie de la chaîne de caractères.
- **Caractéristiques de réputation du domaine** : ancienneté d'enregistrement (les domaines de phishing sont très souvent enregistrés peu avant la campagne d'attaque), historique de la réputation du domaine, présence dans des listes de réputation connues.
- **Caractéristiques de similarité (typosquatting)** : distance d'édition (par exemple la distance de Levenshtein) entre le domaine observé et des domaines légitimes connus et fréquemment ciblés, permettant de repérer des variantes comme un « o » remplacé par un « 0 », ou l'insertion d'un caractère supplémentaire.
- **Caractéristiques de contenu de la page** (lorsque la page est effectivement visitée, ce qui n'est pas toujours souhaitable pour des raisons de sécurité et de latence) : présence d'un formulaire de connexion imitant une marque connue, ressemblance visuelle avec un site légitime.

```python
import numpy as np

def distance_levenshtein(a: str, b: str) -> int:
    """Distance d'édition simple entre deux chaînes (implémentation pédagogique)."""
    m, n = len(a), len(b)
    d = np.zeros((m + 1, n + 1), dtype=int)
    d[:, 0] = np.arange(m + 1)
    d[0, :] = np.arange(n + 1)
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            cout = 0 if a[i - 1] == b[j - 1] else 1
            d[i, j] = min(d[i - 1, j] + 1, d[i, j - 1] + 1, d[i - 1, j - 1] + cout)
    return d[m, n]

# Exemple : repérer un domaine potentiellement usurpateur
domaines_legitimes = ["banque-exemple.com", "monservice.org"]
domaine_observe = "banque-exempl3.com"

distances = [distance_levenshtein(domaine_observe, d) for d in domaines_legitimes]
print("Distances aux domaines légitimes connus :", distances)
```

Une distance d'édition faible (par exemple 1 ou 2) entre un domaine observé et un domaine légitime largement connu, combinée à une faible ancienneté d'enregistrement, constitue un indicateur de typosquatting particulièrement fiable, bien plus robuste qu'une analyse purement lexicale de l'URL prise isolément.

## 5. Analyse de journaux (log analysis) assistée par NLP

Les journaux systèmes et applicatifs peuvent être traités comme du texte semi-structuré :

- **Regroupement de motifs (log clustering / log parsing)** : regrouper des lignes de log similaires en « templates » (ex. `outils comme Drain`), pour réduire des millions de lignes à quelques centaines de motifs analysables.
- **Détection d'anomalies séquentielles** : modéliser la séquence normale d'événements (via un modèle de séquence) et détecter les déviations, potentiellement révélatrices d'un comportement d'attaque (ex. séquence anormale de commandes après une compromission).
- **Résumé automatique** : synthétiser de grands volumes d'alertes pour un analyste SOC, en s'appuyant de plus en plus sur des modèles de langage génératifs (avec vigilance sur la fiabilité de leurs sorties, cf. chapitre 6).

### 5.1 Principe du parsing de logs (illustration simplifiée)

Une ligne de log brute comme `Connexion échouée pour l'utilisateur alice depuis 10.0.0.5` et une autre `Connexion échouée pour l'utilisateur bob depuis 10.0.0.9` partagent un même **template** sous-jacent (`Connexion échouée pour l'utilisateur <VAR> depuis <VAR>`), les parties variables correspondant aux paramètres spécifiques de chaque événement. Les outils de parsing de logs (dont l'algorithme Drain, largement cité dans la littérature) automatisent l'extraction de ces templates à partir d'un flux brut de journaux, permettant de transformer un volume considérable de lignes en un nombre restreint de motifs, sur lesquels des techniques classiques (comptage de fréquence, détection d'anomalies sur la distribution des templates au cours du temps) deviennent applicables :

```python
import re

def extraction_template_simplifiee(ligne_log: str) -> str:
    """
    Illustration très simplifiée du principe de templating :
    remplace les nombres et adresses IP par un marqueur générique.
    (Une implémentation réelle, type Drain, est nettement plus robuste
    et s'appuie sur un arbre de recherche par préfixes.)
    """
    ligne = re.sub(r"\b\d{1,3}(\.\d{1,3}){3}\b", "<IP>", ligne_log)
    ligne = re.sub(r"\b\d+\b", "<NUM>", ligne)
    return ligne

exemples = [
    "Connexion échouée pour l'utilisateur alice depuis 10.0.0.5",
    "Connexion échouée pour l'utilisateur bob depuis 10.0.0.9",
]
for ligne in exemples:
    print(extraction_template_simplifiee(ligne))
```

Une fois les journaux réduits à une séquence de templates, on peut appliquer les techniques de détection d'anomalies séquentielles vues au chapitre 3 (modélisation de la séquence normale d'événements, alerte sur les déviations), ou simplement surveiller l'apparition d'un template rare ou inédit — une piste simple mais souvent efficace pour détecter des événements systèmes inhabituels sans construire un modèle complexe.

### 5.2 Prudence sur l'usage de modèles génératifs pour le résumé d'alertes

L'usage croissant de modèles de langage génératifs pour synthétiser de grands volumes d'alertes à destination d'un analyste SOC (par exemple, produire un résumé en langage naturel d'une série d'événements corrélés) offre un gain de productivité réel, mais introduit un risque spécifique : ces modèles peuvent produire des affirmations plausibles mais factuellement incorrectes (« hallucinations »), en particulier lorsqu'ils sont sollicités pour interpréter des événements ambigus ou incomplets. Un résumé généré automatiquement doit donc rester **vérifiable** (idéalement accompagné des événements bruts sources) et ne jamais se substituer entièrement au jugement de l'analyste pour une décision de sécurité critique — ce point est approfondi au chapitre 6 sous l'angle plus large de la fiabilité des systèmes d'IA générative.

## 6. Limites spécifiques au NLP en sécurité

- Les attaquants adaptent activement leur texte pour éviter la détection (fautes volontaires, homoglyphes, texte caché dans des images) — un jeu du chat et de la souris comparable à celui vu pour les malwares.
- Un modèle NLP entraîné sur une langue ou un contexte culturel donné généralise mal à d'autres langues ou contextes.
- Le contenu généré automatiquement par des modèles de langage (utilisé de façon offensive par des attaquants pour rédiger des messages de phishing très convaincants et sans fautes, y compris en plusieurs langues) réduit l'efficacité des indicateurs textuels classiques (fautes d'orthographe, formulations maladroites) longtemps utilisés pour repérer le phishing.

### 6.1 Les homoglyphes : une évasion visuelle difficile à détecter par le texte seul

Les attaques par **homoglyphes** exploitent la ressemblance visuelle entre caractères provenant de jeux de caractères différents (par exemple, la lettre latine « a » et une lettre cyrillique visuellement quasi identique) pour construire des noms de domaine ou des mots qui semblent identiques à l'œil humain mais diffèrent au niveau des caractères Unicode réellement utilisés. Un filtre NLP opérant uniquement sur la représentation textuelle brute peut être trompé si les deux caractères sont traités comme complètement distincts par le modèle de représentation, alors qu'un humain ne perçoit aucune différence. Des contre-mesures existent (normalisation Unicode, détection de scripts mixtes dans un même nom de domaine — un domaine légitime mélange rarement plusieurs alphabets), mais illustrent bien que la robustesse d'un système NLP de sécurité doit être pensée au niveau de l'encodage des caractères, pas seulement au niveau du sens des mots.

## À retenir

- Le NLP transforme du texte en représentation numérique exploitable par un modèle (TF-IDF, n-grammes de caractères, embeddings, Transformers), avec un compromis performance/coût de calcul et de robustesse à l'évasion.
- TF-IDF reste une base de référence solide et peu coûteuse ; les n-grammes de caractères apportent une robustesse particulière aux variations orthographiques volontaires, fréquentes en phishing.
- La détection de phishing combine efficacement analyse du contenu textuel et caractéristiques techniques (URL, en-têtes, authentification du domaine, ancienneté du nom de domaine), car le texte seul devient insuffisant face à des messages sans fautes générés ou peaufinés par IA.
- L'analyse de journaux par NLP passe souvent par une étape de regroupement en templates (log parsing), qui réduit drastiquement le volume à traiter avant d'appliquer des techniques de détection d'anomalies séquentielles.
- L'essor des modèles de langage génératifs facilite la production de contenus de phishing convaincants, ce qui affaiblit les indicateurs textuels traditionnels et pousse vers des défenses combinant plusieurs signaux ; ces mêmes modèles, utilisés côté défense pour résumer des alertes, doivent être employés avec prudence en raison du risque d'hallucination.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
