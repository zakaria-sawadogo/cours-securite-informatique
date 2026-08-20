# Chapitre 6 — Sécurité de l'IA : attaques adversariales et IA offensive

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=05-ia-appliquee-securite-informatique/slides/06-securite-de-lia.txt){ target=_blank }

## 1. L'IA comme nouvelle surface d'attaque

À mesure que des systèmes d'IA sont déployés dans des fonctions sensibles (détection de fraude, contrôle d'accès biométrique, filtrage de contenu, systèmes autonomes), ils deviennent eux-mêmes des cibles. Le référentiel **OWASP Machine Learning Security Top 10** et le cadre **MITRE ATLAS** recensent ces menaces spécifiques.

### 1.1 Pourquoi la sécurité de l'IA ne se réduit pas à la sécurité logicielle classique

Un système ML introduit des surfaces d'attaque qui n'ont pas d'équivalent direct dans la sécurité logicielle traditionnelle, ce qui justifie de le traiter comme une discipline à part entière plutôt que comme un simple cas particulier de sécurité applicative :

- **La logique du système n'est pas écrite explicitement par un développeur**, mais apprise à partir de données. Il n'existe donc pas de code source à auditer ligne par ligne pour garantir un comportement correct : la « logique » est distribuée dans des millions ou des milliards de paramètres numériques, ce qui rend l'audit traditionnel (revue de code, analyse statique) largement inapplicable.
- **Les données d'entraînement deviennent elles-mêmes une surface d'attaque** (section 3), alors qu'en sécurité logicielle classique, les données de configuration ne sont généralement pas le vecteur principal de compromission du comportement du programme.
- **Le modèle entraîné peut être considéré comme un actif à protéger en tant que tel** (propriété intellectuelle, avantage concurrentiel), au même titre que le code source d'un logiciel classique, mais avec des vecteurs de vol différents (extraction par requêtes successives, section 4).
- **La frontière entre comportement « normal » et comportement « attaqué »** peut être ténue et dépendre de l'interprétation du contexte (une entrée légèrement modifiée reste-t-elle une entrée légitime « bruitée » ou une attaque délibérée ?), contrairement à une vulnérabilité logicielle classique où l'exploitation produit généralement un comportement clairement anormal (crash, exécution de code arbitraire).

Ces particularités expliquent l'émergence de cadres dédiés comme MITRE ATLAS (*Adversarial Threat Landscape for Artificial-Intelligence Systems*), qui adapte au domaine de l'IA une structuration en tactiques et techniques inspirée du cadre MITRE ATT&CK bien connu en sécurité classique, tout en introduisant des catégories propres au ML (empoisonnement, exfiltration de modèle, évasion) absentes du cadre original.

## 2. Attaques adversariales (evasion attacks)

Principe : modifier légèrement une entrée, de façon souvent imperceptible pour un humain, pour tromper un modèle et lui faire produire une prédiction erronée, sans modifier le comportement réel de l'objet analysé.

- **Sur des images** : ajouter un bruit calculé, invisible à l'œil nu, qui fait classer une image de panneau stop comme un panneau de limitation de vitesse par un modèle de vision.
- **Sur des malwares** : ajouter des octets inoffensifs à un binaire malveillant pour franchir la frontière de décision d'un classifieur, sans changer le comportement malveillant réel (évoqué au chapitre 4).
- **Sur du texte (phishing, spam)** : introduire des variations orthographiques, des caractères invisibles ou des synonymes pour échapper à un filtre NLP tout en restant compréhensible par un humain.

### 2.1 Intuition géométrique : pourquoi une perturbation imperceptible suffit

Un modèle de classification définit implicitement une **frontière de décision** dans l'espace des caractéristiques d'entrée, séparant les régions associées à chaque classe prédite. Une observation légitime se situe généralement loin de cette frontière, dans une région où le modèle est « confiant ». Une attaque adversariale cherche la perturbation de **plus petite amplitude possible** (au sens d'une distance donnée, par exemple la norme euclidienne pour une image) qui déplace le point représentant l'entrée juste de l'autre côté de la frontière de décision.

Ce qui rend ces attaques particulièrement redoutables sur des modèles de deep learning tient à une propriété géométrique associée à la haute dimension de l'espace d'entrée (des millions de pixels pour une image, par exemple) : dans un espace de très grande dimension, il existe généralement une direction de perturbation qui déplace fortement la prédiction du modèle tout en modifiant très peu chaque composante individuelle de l'entrée (chaque pixel), ce qui rend la perturbation totale imperceptible pour un observateur humain tout en étant efficace du point de vue du modèle. Cette observation, documentée dès les premiers travaux sur le sujet (par exemple la méthode *Fast Gradient Sign Method*, qui exploite directement le gradient du modèle pour calculer une perturbation efficace en une seule étape), a durablement structuré la recherche en robustesse adversariale du deep learning.

### Modèles d'attaque

| Modèle | Connaissance de l'attaquant |
|---|---|
| **Boîte blanche (white-box)** | accès complet au modèle (architecture, paramètres) — permet de calculer des perturbations optimales |
| **Boîte noire (black-box)** | accès uniquement aux prédictions du modèle (requêtes successives) — attaques par substitution (entraîner un modèle de substitution pour approximer les frontières de décision) |

### 2.2 Transférabilité : pourquoi le boîte noire n'est pas rassurant

Une propriété empirique importante, souvent sous-estimée, réduit fortement l'intérêt pratique de la distinction boîte blanche/boîte noire du point de vue défensif : la **transférabilité des exemples adversariaux**. Un exemple adversarial conçu pour tromper un modèle donné a souvent de bonnes chances de tromper également un *autre* modèle entraîné sur une tâche similaire, même avec une architecture différente. Cette propriété permet à un attaquant en situation de boîte noire de contourner l'absence d'accès direct au modèle cible : il entraîne un **modèle de substitution** sur des données similaires (ou simplement à partir des prédictions obtenues en interrogeant le modèle cible), conçoit une attaque en boîte blanche contre ce modèle de substitution, puis applique la perturbation obtenue au modèle cible réel, en espérant qu'elle transfère avec succès.

Cette propriété a une implication défensive directe : « cacher » les détails d'un modèle (une approche parfois appelée *security through obscurity*) offre une protection bien plus limitée qu'il n'y paraît intuitivement contre les attaques adversariales, et ne doit jamais constituer la seule ligne de défense.

## 3. Empoisonnement de données (data poisoning)

Attaque visant la **phase d'entraînement** plutôt que l'inférence : un attaquant capable d'injecter des données malveillantes dans le jeu d'entraînement (ex. un système qui se ré-entraîne en continu sur des données utilisateurs) peut dégrader la performance générale du modèle ou y insérer une **porte dérobée (backdoor)** : le modèle se comporte normalement sauf en présence d'un déclencheur spécifique connu de l'attaquant, qui provoque alors une prédiction choisie.

### 3.1 Empoisonnement « en aveugle » vs empoisonnement ciblé

Il est utile de distinguer deux familles d'objectifs pour un attaquant menant une attaque d'empoisonnement, car elles appellent des contre-mesures différentes :

- **Empoisonnement de disponibilité (availability poisoning)** : l'attaquant vise simplement à dégrader la performance générale du modèle (le rendre globalement moins fiable), par exemple en injectant un volume significatif de données mal étiquetées ou bruitées. Cet objectif est relativement facile à détecter *a posteriori*, car il produit une dégradation mesurable et globale de la performance sur un jeu de validation propre.
- **Empoisonnement ciblé avec porte dérobée (backdoor / targeted poisoning)** : l'attaquant vise à faire échouer le modèle uniquement sur des entrées spécifiques (contenant un déclencheur particulier), tout en préservant une performance normale, voire excellente, sur toutes les autres entrées. Ce type d'attaque est nettement plus difficile à détecter, car le modèle « empoisonné » réussit la quasi-totalité des tests d'évaluation standards — seul un test ciblé sur des entrées contenant le déclencheur révélerait le problème, ce qui suppose de savoir *a priori* qu'un tel déclencheur pourrait exister.

Un exemple pédagogique illustrant l'intuition d'une porte dérobée : un système de détection de spam ré-entraîné périodiquement sur les messages que les utilisateurs signalent ou ne signalent pas pourrait être « empoisonné » si un attaquant parvient, à grande échelle, à faire en sorte que des messages contenant un mot-clé anodin spécifique soient systématiquement traités comme légitimes par un grand nombre de comptes compromis ou complices — le modèle apprendrait alors, sans intention explicite de ses concepteurs, à associer ce mot-clé à un comportement « non spam », que l'attaquant pourrait ensuite exploiter dans ses propres campagnes.

### 3.2 Facteurs de risque et contre-mesures pratiques

Le risque d'empoisonnement dépend fortement du **mode d'entraînement** du système :

- Un modèle entraîné **une fois hors ligne**, sur un jeu de données figé et audité, est nettement moins exposé qu'un système en **apprentissage continu (online learning)**, qui intègre en permanence de nouvelles données provenant potentiellement d'utilisateurs non fiables.
- Le degré de contrôle sur la **provenance** des données d'entraînement (données internes auditées vs données collectées automatiquement sur des sources ouvertes ou semi-ouvertes) est un facteur de risque déterminant.

Les contre-mesures pratiques incluent la validation statistique des nouvelles données avant intégration (détection d'anomalies sur les données elles-mêmes, avant même d'entraîner le modèle dessus), la traçabilité des sources (savoir d'où provient chaque exemple d'entraînement), la limitation du poids d'une source unique dans le jeu de données global, et des tests de robustesse ciblés recherchant explicitement des comportements de porte dérobée avant tout déploiement en production.

## 4. Extraction et inversion de modèle

- **Extraction de modèle (model extraction)** : reconstruire une approximation d'un modèle propriétaire à partir de requêtes successives à son API, permettant de le voler ou de préparer une attaque adversariale en boîte blanche sur la copie.
- **Inversion de modèle / attaque par appartenance (membership inference)** : déterminer si une donnée spécifique a été utilisée dans l'entraînement d'un modèle, ou reconstruire partiellement des données d'entraînement sensibles — un enjeu direct de confidentialité (RGPD) lorsque le modèle a été entraîné sur des données personnelles.

### 4.1 Pourquoi l'attaque par appartenance fonctionne : le lien avec le surapprentissage

L'attaque par appartenance (*membership inference*) exploite une observation liée directement à une notion vue au chapitre 2 : un modèle qui a **surappris** (overfitting) se comporte différemment sur les données qu'il a vues à l'entraînement par rapport à des données nouvelles — typiquement, il produit des prédictions avec une confiance plus élevée (ou une erreur plus faible) sur les exemples d'entraînement mémorisés que sur des exemples jamais vus, même de même nature. Un attaquant disposant d'un accès aux scores de confiance produits par le modèle (et non seulement à la décision finale) peut exploiter cet écart statistique pour deviner, avec une fiabilité supérieure au hasard, si une donnée spécifique a fait partie de l'ensemble d'entraînement.

Cette observation a une conséquence défensive concrète : les techniques de régularisation qui limitent le surapprentissage (évoquées au chapitre 2 comme bonnes pratiques de généralisation) réduisent également, en tant qu'effet secondaire bénéfique, la vulnérabilité du modèle à ce type d'attaque de confidentialité — un des rares cas où bonne pratique de performance et bonne pratique de sécurité coïncident directement.

### 4.2 Confidentialité différentielle : une garantie mathématique plutôt qu'une simple bonne pratique

Contrairement à des mesures ad hoc, la **confidentialité différentielle (differential privacy)** offre une garantie mathématique formelle : elle consiste à ajouter un bruit calibré, soit aux données d'entraînement, soit au processus d'apprentissage lui-même (par exemple aux gradients calculés à chaque étape d'entraînement d'un réseau de neurones), de sorte que la présence ou l'absence d'un individu donné dans le jeu de données n'affecte que marginalement, de façon mathématiquement bornée, les prédictions du modèle final. Cette technique constitue une défense de fond contre l'inversion de modèle et l'attaque par appartenance, au prix d'un compromis à gérer avec soin : plus la garantie de confidentialité exigée est forte (plus le bruit ajouté est important), plus la performance prédictive du modèle tend à se dégrader — un arbitrage explicite entre confidentialité et utilité qui doit être calibré selon la sensibilité réelle des données concernées.

## 5. IA offensive : usage par les attaquants

- Génération automatisée de contenus de phishing personnalisés et sans fautes, à grande échelle.
- Assistance à l'écriture de code malveillant ou à l'automatisation de tâches de reconnaissance.
- Génération de deepfakes (voix, image, vidéo) utilisés dans des fraudes ciblées (ex. usurpation de la voix d'un dirigeant pour autoriser un virement frauduleux).
- Automatisation partielle de certaines phases de reconnaissance ou de rédaction de rapports par des attaquants.

### 5.1 Ce que l'IA générative change réellement dans la chaîne d'attaque

Il est utile de préciser en quoi l'IA générative modifie qualitativement (et pas seulement quantitativement) certaines étapes classiques d'une chaîne d'attaque (*kill chain*), plutôt que de se contenter d'un constat général :

- **Passage à l'échelle de la personnalisation** : historiquement, un phishing très personnalisé (*spear phishing*) exigeait un travail de reconnaissance manuel coûteux en temps, limitant ce niveau de sophistication à des cibles à forte valeur. L'automatisation partielle de cette reconnaissance et de la rédaction du message par des modèles génératifs abaisse ce coût, rendant potentiellement viable un phishing hautement personnalisé à une échelle beaucoup plus large.
- **Réduction de la barrière linguistique et technique** : un attaquant peu familier d'une langue cible, ou peu expérimenté techniquement, peut s'appuyer sur des outils génératifs pour produire un texte convaincant dans une langue qu'il ne maîtrise pas, ou pour obtenir de l'assistance sur des tâches de programmation qu'il ne saurait pas réaliser seul — abaissant ainsi, potentiellement, le niveau de compétence minimal requis pour certaines formes d'attaque.
- **Deepfakes audio/vidéo** : la baisse du coût et de la complexité technique nécessaires pour produire un contenu synthétique crédible (voix imitée, visage substitué) élargit le champ des attaques d'ingénierie sociale au-delà du texte écrit, vers des canaux (appel téléphonique, visioconférence) traditionnellement considérés comme plus fiables pour une vérification d'identité informelle.

Ce constat ne signifie pas que l'IA générative crée des catégories d'attaques entièrement nouvelles (le phishing, l'ingénierie sociale et l'usurpation d'identité préexistent largement à ces outils), mais qu'elle en modifie l'échelle, le coût et l'accessibilité — une nuance importante pour calibrer une réponse défensive proportionnée plutôt que de céder à un discours excessivement alarmiste ou, à l'inverse, complaisant.

## 6. Contre-mesures et bonnes pratiques

| Menace | Contre-mesure |
|---|---|
| Attaques adversariales | entraînement adversarial (inclure des exemples adversariaux dans l'entraînement), détection de perturbations anormales, limitation du taux de requêtes à une API de modèle |
| Empoisonnement de données | validation et traçabilité des sources de données d'entraînement, détection d'anomalies dans les données avant réentraînement |
| Extraction/inversion de modèle | limitation et surveillance des requêtes API, techniques de confidentialité différentielle |
| IA offensive (phishing, deepfakes) | sensibilisation renforcée, vérification hors bande des demandes sensibles (procédures « callback » avant tout virement), défenses multi-signaux ne reposant pas uniquement sur des indices textuels |

### 6.1 L'entraînement adversarial : principe et limite fondamentale

L'**entraînement adversarial** (*adversarial training*), cité dans le tableau ci-dessus comme contre-mesure aux attaques par évasion, consiste à générer explicitement des exemples adversariaux pendant la phase d'entraînement et à les inclure dans le jeu de données d'apprentissage, de sorte que le modèle apprenne à correctement classer également ces versions perturbées. Cette technique améliore mesurablement la robustesse du modèle face aux types de perturbations utilisés pendant l'entraînement, mais comporte une limite conceptuelle importante à connaître : elle n'offre en général aucune garantie de robustesse face à des types de perturbations *non anticipés* lors de l'entraînement — un attaquant qui découvre une nouvelle méthode d'attaque non représentée dans le jeu d'entraînement adversarial peut potentiellement continuer à tromper le modèle. Elle s'apparente ainsi davantage à un durcissement empirique qu'à une preuve formelle de robustesse, ce qui justifie de la combiner systématiquement avec d'autres mesures (surveillance en production, limitation des requêtes, défense en profondeur) plutôt que de la considérer comme une solution définitive.

### 6.2 Vers une gouvernance de la sécurité de l'IA en entreprise

Au-delà des contre-mesures techniques ponctuelles, une organisation déployant des systèmes d'IA en production gagne à structurer une démarche de gouvernance dédiée, comparable dans son esprit à un programme de sécurité applicative classique mais adaptée aux spécificités du ML :

- **Inventaire des modèles en production** : savoir précisément quels modèles sont déployés, sur quelles données ils ont été entraînés, et quelles décisions ils influencent — un préalable simple mais souvent négligé.
- **Évaluation de robustesse avant mise en production**, incluant des tests adversariaux ciblés proportionnés à la criticité du système (un modèle de recommandation de contenu n'appelle pas le même niveau d'exigence qu'un système de contrôle d'accès biométrique).
- **Plan de réponse à incident spécifique à l'IA**, anticipant les scénarios propres à ce domaine (détection d'une dérive de performance suspecte, procédure de retrait rapide d'un modèle compromis, communication en cas de fuite de données d'entraînement).
- **Formation continue des équipes**, tant côté développement (bonnes pratiques de robustesse et de confidentialité) que côté utilisateurs finaux (sensibilisation aux nouvelles formes d'ingénierie sociale assistées par IA).

Ce dernier point relie directement ce chapitre à l'ensemble du module : la sécurité de l'IA n'est pas un sujet isolé réservé aux data scientists, mais un prolongement naturel des pratiques de sécurité organisationnelle et technique déjà enseignées dans les autres modules du programme.

## À retenir

- Les systèmes d'IA sont vulnérables à des classes d'attaques spécifiques (adversariales, empoisonnement, extraction) distinctes des vulnérabilités logicielles classiques, notamment parce que leur logique est apprise à partir de données plutôt qu'écrite explicitement.
- Les attaques adversariales exploitent des propriétés géométriques des modèles en haute dimension ; la transférabilité des exemples adversariaux entre modèles limite fortement l'intérêt défensif de la seule opacité (« security through obscurity »).
- L'empoisonnement de données peut viser une dégradation générale (facilement détectable) ou installer une porte dérobée ciblée (beaucoup plus difficile à détecter), avec un risque directement lié au mode d'entraînement (hors ligne vs apprentissage continu).
- Les attaques d'inversion de modèle exploitent le lien entre surapprentissage et fuite d'information ; la confidentialité différentielle offre une garantie mathématique, au prix d'un compromis explicite avec la performance.
- L'IA offensive ne crée pas nécessairement de nouvelles catégories d'attaques, mais en réduit le coût et augmente l'échelle et le réalisme (phishing personnalisé, deepfakes), ce qui impose des contre-mesures organisationnelles autant que techniques.
- L'entraînement adversarial améliore la robustesse empirique mais ne garantit pas une protection contre des attaques non anticipées ; une véritable gouvernance de la sécurité de l'IA en entreprise dépasse les seules contre-mesures techniques ponctuelles.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
