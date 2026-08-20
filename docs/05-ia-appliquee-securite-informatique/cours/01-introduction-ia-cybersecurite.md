# Chapitre 1 — Introduction : IA et cybersécurité, panorama

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=05-ia-appliquee-securite-informatique/slides/01-introduction-ia-cybersecurite.txt){ target=_blank }

## 1. Pourquoi l'IA en cybersécurité ?

Les volumes de données à surveiller (journaux, flux réseau, alertes) ont dépassé depuis longtemps la capacité d'analyse manuelle humaine. L'IA et le machine learning (ML) permettent d'automatiser la détection de schémas (patterns) suspects, de traiter des volumes massifs en temps quasi réel, et de détecter des menaces nouvelles (« zero-day ») que des règles de détection statiques (signatures) ne couvrent pas encore.

Trois facteurs expliquent cette montée en puissance de l'IA dans les outils de sécurité :

- **L'explosion du volume de données** : un centre opérationnel de sécurité (SOC) de taille moyenne peut recevoir des milliers, voire des millions d'événements par jour (connexions réseau, authentifications, alertes d'antivirus). Aucune équipe humaine ne peut examiner ce volume ligne par ligne.
- **La disponibilité croissante de puissance de calcul et de données** : l'entraînement de modèles complexes (réseaux de neurones profonds notamment) est devenu accessible grâce aux progrès matériels (GPU) et à la disponibilité de jeux de données publics ou semi-publics (trafic réseau capturé, corpus de malwares, corpus de phishing).
- **L'évolution des attaquants eux-mêmes** : les techniques d'attaque se diversifient et s'automatisent, ce qui pousse la défense à automatiser également ses capacités de détection et de réponse.

Il est toutefois essentiel de garder à l'esprit que l'IA en sécurité est un **outil d'aide à la décision et d'automatisation**, pas une boîte magique. Elle vient compléter des dispositifs existants (pare-feux, IDS/IPS par signatures, SIEM, procédures organisationnelles), et sa valeur dépend directement de la qualité de son intégration dans un processus de sécurité plus large.

### 1.1 Bref rappel : IA, machine learning, deep learning

Ces trois termes sont souvent utilisés de façon interchangeable dans les discours commerciaux, ce qui entretient une confusion qu'il convient de dissiper dès l'introduction du module :

- **Intelligence artificielle (IA)** : discipline englobante visant à faire réaliser par une machine des tâches nécessitant normalement une forme d'intelligence (raisonnement, perception, apprentissage). Elle inclut aussi des approches non statistiques (systèmes experts à base de règles, par exemple).
- **Machine learning (ML)** : sous-ensemble de l'IA où le système apprend automatiquement des régularités à partir de données, plutôt que d'exécuter des règles écrites explicitement par un humain. C'est le cœur de ce module.
- **Deep learning (apprentissage profond)** : sous-ensemble du ML reposant sur des réseaux de neurones à plusieurs couches (« profonds »), particulièrement performants sur des données non structurées (image, texte, séquences) mais plus coûteux en données et en calcul, et généralement moins interprétables.

En cybersécurité, on trouve les trois : des règles expertes (signatures, règles YARA, règles de corrélation SIEM), du ML « classique » (Random Forest, régression logistique) et du deep learning (réseaux de neurones pour l'analyse de séquences d'appels système ou de texte). Le chapitre 2 détaille les différences entre ces familles d'algorithmes.

## 2. Deux faces d'une même discipline

- **IA défensive** : détection d'intrusions, de malwares, de phishing, priorisation d'alertes, automatisation de la réponse à incident (SOAR).
- **IA comme surface d'attaque** : les systèmes d'IA eux-mêmes deviennent des cibles (empoisonnement de données d'entraînement, attaques adversariales, extraction de modèle).
- **IA offensive** : usage de l'IA par les attaquants (génération de contenus de phishing convaincants, automatisation de la reconnaissance, génération de code malveillant).

Ce module couvre les trois dimensions, avec un accent particulier sur la défense (chapitres 3 à 5) puis sur la sécurité de l'IA elle-même (chapitre 6).

Il est utile de visualiser ces trois dimensions comme les sommets d'un même triangle, en interaction permanente : une amélioration de la détection ML (dimension défensive) pousse les attaquants à développer des techniques d'évasion visant spécifiquement les modèles (dimension « IA comme surface d'attaque »), tandis que l'IA offensive leur permet d'automatiser et de personnaliser ces contournements à grande échelle. Comprendre une seule de ces dimensions sans les deux autres donne une vision incomplète, voire dangereuse, du sujet — un analyste qui déploie un détecteur ML sans connaître les techniques d'évasion (chapitre 6) surestimera la robustesse de son dispositif.

## 3. Cas d'usage défensifs courants

| Cas d'usage | Type de tâche ML | Exemple |
|---|---|---|
| Détection d'intrusions réseau | classification supervisée / détection d'anomalies | identifier un trafic réseau anormal |
| Détection de malwares | classification supervisée | classer un fichier binaire comme sain/malveillant |
| Détection de phishing | classification de texte (NLP) | analyser le contenu d'un e-mail ou d'une URL |
| Analyse comportementale des utilisateurs (UEBA) | détection d'anomalies | repérer un compte utilisateur au comportement inhabituel |
| Priorisation d'alertes SOC | classification / scoring | réduire la fatigue d'alerte des analystes |

### 3.1 Où l'IA s'intègre-t-elle concrètement dans une architecture de sécurité ?

Il est important de ne pas se représenter l'IA comme un produit unique, mais comme une brique intégrée à des outils existants :

- **Au niveau du réseau** : les IDS/IPS de nouvelle génération intègrent des modules de détection d'anomalies en complément des signatures classiques (chapitre 3).
- **Au niveau du poste de travail** : les EDR (*Endpoint Detection and Response*) embarquent des classifieurs ML entraînés sur des caractéristiques statiques et comportementales des exécutables (chapitre 4).
- **Au niveau de la messagerie** : les passerelles anti-phishing combinent des modèles NLP et des règles techniques sur les URL et en-têtes (chapitre 5).
- **Au niveau du SOC** : les plateformes SIEM/SOAR utilisent le ML pour corréler les alertes, réduire le bruit, et parfois pour suggérer des playbooks de réponse automatisée.

Ainsi, un même diplômé du programme de cybersécurité amené à travailler comme analyste SOC, pentester, ou architecte sécurité rencontrera très probablement des composants ML dans les outils qu'il opère, même sans les avoir lui-même développés — d'où l'importance de comprendre leur fonctionnement et leurs limites, indépendamment de la spécialisation professionnelle visée.

## 4. Limites et illusions à éviter

- **Le ML n'élimine pas le besoin d'expertise humaine** : un modèle mal conçu ou mal évalué produit des faux positifs (fatigue d'alerte) ou des faux négatifs (menaces non détectées) coûteux.
- **Les données d'entraînement conditionnent tout** : un modèle entraîné sur un contexte donné (réseau, période) généralise mal à un contexte différent (concept drift).
- **Explicabilité** : un modèle « boîte noire » (ex. réseau de neurones profond) est difficile à auditer et à justifier auprès d'un analyste ou d'un régulateur — un critère important dans le choix du modèle selon le contexte d'usage.
- **L'IA n'est pas une solution miracle contre un attaquant déterminé** : un attaquant conscient qu'un système de détection utilise du ML peut chercher spécifiquement à le contourner (chapitre 6).

### 4.1 Le mythe du modèle « universel »

Une idée reçue fréquente consiste à croire qu'un modèle ML performant, une fois entraîné, fonctionnera durablement et dans tout contexte. En pratique, un modèle de détection déployé dans un environnement de production est soumis à plusieurs formes d'usure :

- **La dérive conceptuelle (concept drift)** évoquée plus haut : les usages légitimes évoluent (nouvelles applications, nouveaux protocoles), ce qui rapproche progressivement le trafic « normal » de ce que le modèle avait appris à considérer comme suspect, dégradant le rappel ou augmentant le taux de faux positifs.
- **L'adaptation active de l'attaquant** : contrairement à un phénomène purement statistique (comme la météo), l'« environnement » en sécurité comprend des acteurs qui réagissent délibérément aux défenses mises en place — un comportement qu'on ne retrouve pas dans la plupart des applications « classiques » du ML (recommandation, vision par ordinateur sur des images naturelles, etc.).
- **Le changement d'échelle** : un modèle validé sur un petit jeu de données pilote peut se comporter très différemment une fois déployé à l'échelle d'un réseau d'entreprise complet, avec une diversité de trafic bien plus grande.

Ce constat justifie une pratique centrale du domaine : la **supervision continue** des modèles en production (réévaluation périodique, réentraînement, suivi de métriques de dérive), abordée plus en détail au chapitre 2.

### 4.2 Coût, gouvernance et responsabilité

Au-delà des aspects strictement techniques, l'adoption de l'IA en sécurité soulève des questions de gouvernance qu'un futur professionnel doit savoir anticiper :

- **Qui est responsable en cas d'erreur du modèle ?** (un faux négatif ayant laissé passer une attaque, un faux positif ayant bloqué une activité légitime critique).
- **Comment documenter et auditer** les décisions d'un modèle, en particulier dans des secteurs réglementés (finance, santé) ?
- **Quel est le coût réel** (données, calcul, expertise, maintenance) d'un système ML par rapport à une approche par règles plus simple, et ce coût est-il justifié par le gain de détection obtenu ?

Ces questions n'ont pas de réponse unique : elles dépendent du contexte organisationnel et seront reprises, sous l'angle technique de la sécurité de l'IA elle-même, au chapitre 6.

## 5. Articulation avec les autres modules du programme

Ce module s'appuie sur les notions de trafic réseau et de vulnérabilités vues en *Audit organisation et technique*, et prépare une lecture critique des outils de sécurité modernes (EDR, XDR, SIEM) qui intègrent aujourd'hui massivement des composants de ML.

Concrètement, les compétences développées dans les modules précédents restent indispensables : comprendre un protocole réseau (pour interpréter les caractéristiques extraites d'un flux), connaître les techniques d'attaque classiques (pour évaluer si un modèle de détection couvre réellement les menaces pertinentes), et maîtriser les principes d'audit (pour évaluer la fiabilité et les limites d'un système de détection avant de s'y fier en production). L'IA appliquée à la sécurité n'est donc pas un domaine cloisonné, mais un prolongement naturel des compétences déjà acquises, appliqué à un nouvel outil.

## À retenir

- L'IA en cybersécurité recouvre trois dimensions : défense, sécurité de l'IA elle-même, et usage offensif par les attaquants.
- Le ML complète mais ne remplace pas l'expertise humaine et les approches par signatures/règles.
- La qualité et la représentativité des données d'entraînement sont le facteur déterminant de la performance réelle d'un système de détection basé ML.
- Un modèle performant en laboratoire peut se dégrader en production à cause de la dérive conceptuelle et de l'adaptation active des attaquants ; la supervision continue est indispensable.
- L'adoption de l'IA en sécurité pose aussi des questions de gouvernance (responsabilité, auditabilité, coût) qui dépassent le seul aspect technique et seront reprises tout au long du module.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
