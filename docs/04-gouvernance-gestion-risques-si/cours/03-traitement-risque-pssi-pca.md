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

### 1.1 Combiner les stratégies plutôt que choisir une seule option

Dans la pratique, ces quatre stratégies ne sont pas exclusives les unes des autres : elles se combinent fréquemment pour un même risque. Prenons l'exemple d'un risque de compromission d'un serveur applicatif exposé sur Internet : une organisation peut simultanément **réduire** le risque (durcissement du serveur, pare-feu applicatif, correctifs réguliers), **transférer** une partie de l'impact financier résiduel (cyber-assurance couvrant les frais de remédiation), et **accepter** ce qui reste après ces mesures (le risque résiduel qu'une vulnérabilité inconnue de type « zero-day » soit exploitée malgré tout). La stratégie d'**évitement** pur reste souvent la moins utilisée en pratique, car elle suppose de renoncer à une activité ou un service ayant une valeur métier — un arbitrage rarement neutre, à ne prendre qu'après avoir sérieusement évalué le coût d'opportunité pour l'organisation.

### 1.2 L'acceptation du risque : une décision qui doit rester traçable

L'acceptation du risque est souvent la stratégie la moins bien maîtrisée en pratique, car elle est parfois confondue avec une simple absence de décision (« on ne fait rien parce que personne n'a tranché ») plutôt qu'avec une décision explicite et documentée. Une acceptation de risque digne de ce nom doit répondre à plusieurs exigences :

- Elle doit être **formalisée par écrit**, avec une justification claire (coût de traitement disproportionné par rapport à l'impact, absence de solution technique satisfaisante, échéance de traitement planifiée mais non encore atteinte).
- Elle doit être **validée par une autorité compétente**, dont le niveau hiérarchique est proportionné à la criticité du risque accepté — un risque mineur peut être accepté par un responsable opérationnel, tandis qu'un risque majeur doit remonter à la direction générale (voir chapitre 1).
- Elle doit être **revue périodiquement** : un risque accepté aujourd'hui peut devenir inacceptable demain si son contexte évolue (nouvelle vulnérabilité découverte, changement réglementaire, évolution de la criticité de l'actif concerné).

Une organisation qui accumule des risques acceptés de manière informelle, sans traçabilité ni révision, s'expose à un phénomène bien connu des auditeurs : la « dérive silencieuse » du niveau de risque global, qui n'est révélée qu'au moment d'un incident.

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

### 2.1 Extrait illustratif d'une politique thématique (exemple pédagogique)

Pour rendre plus concret ce que recouvre une « politique thématique », voici un extrait fictif simplifié d'une politique de gestion des mots de passe, tel qu'on pourrait le trouver dans une organisation pédagogique :

> **Politique thématique — Gestion des mots de passe (extrait, exemple pédagogique)**
>
> 1. *Objet* : la présente politique fixe les exigences minimales applicables à tout mot de passe utilisé pour accéder aux systèmes d'information de l'établissement.
> 2. *Exigences de robustesse* : tout mot de passe doit comporter un nombre minimal de caractères et combiner plusieurs catégories (majuscules, minuscules, chiffres, caractères spéciaux), conformément aux recommandations en vigueur.
> 3. *Authentification renforcée* : l'authentification multifacteur est obligatoire pour tout compte disposant de privilèges d'administration.
> 4. *Confidentialité* : un mot de passe est strictement personnel et ne doit jamais être partagé, y compris avec un collègue ou un service support.
> 5. *Gestion des incidents* : toute suspicion de compromission d'un mot de passe doit être signalée sans délai au service compétent, qui procède à sa réinitialisation immédiate.
> 6. *Révision* : la présente politique est revue annuellement par le RSSI et approuvée par la direction.

Cet exemple, volontairement simplifié, illustre le niveau de précision attendu d'une politique thématique : suffisamment concret pour être applicable et contrôlable, sans pour autant se substituer à une procédure opérationnelle détaillée (qui définirait, par exemple, les paramètres techniques exacts de complexité configurés dans l'annuaire d'entreprise).

### 2.2 Diffusion, appropriation et sanctions : les conditions d'efficacité réelle

La rédaction d'une PSSI n'est que la première étape d'un processus plus large, souvent négligé par les organisations peu matures :

- **La diffusion** doit être organisée activement (formation à l'arrivée, campagnes de rappel, intégration dans le règlement intérieur ou les contrats de travail) plutôt que de reposer sur la simple mise à disposition d'un document sur un intranet que personne ne consulte spontanément.
- **L'appropriation** suppose que les principes de la PSSI soient traduits en gestes concrets et compréhensibles par chaque catégorie de collaborateurs, plutôt que présentés comme un texte juridique abstrait.
- **Les sanctions** en cas de non-respect doivent être proportionnées, connues à l'avance, et appliquées de manière cohérente — une politique qui prévoit des sanctions jamais appliquées perd rapidement toute crédibilité auprès des collaborateurs.
- **La révision périodique** doit tenir compte à la fois de l'évolution des menaces (une politique rédigée avant la généralisation du télétravail, par exemple, doit être mise à jour pour en tenir compte) et des retours d'expérience issus des incidents et des audits.

## 3. Plan de Continuité d'Activité (PCA) et Plan de Reprise d'Activité (PRA)

- **PCA** : ensemble des dispositions visant à assurer, en cas de crise majeure, le maintien (éventuellement dégradé) des activités essentielles de l'organisation.
- **PRA** : volet plus spécifiquement technique du PCA, centré sur la reprise des systèmes d'information après un sinistre.

### Notions clés

| Indicateur | Définition |
|---|---|
| **RTO** (Recovery Time Objective) | durée maximale d'interruption acceptable avant reprise du service |
| **RPO** (Recovery Point Objective) | perte de données maximale acceptable, exprimée en temps (ex. RPO de 4h = perte maximale des 4 dernières heures de données) |

Ces deux indicateurs déterminent directement l'architecture de résilience à mettre en œuvre (fréquence de sauvegarde, site de secours actif/passif, réplication temps réel).

### Exemple illustratif de déclinaison RTO/RPO par processus (cas pédagogique)

Tous les processus d'une organisation n'ont pas la même criticité, et un PCA efficace différencie explicitement ses exigences selon les processus métier concernés, plutôt que d'appliquer un objectif uniforme à l'ensemble du système d'information — ce qui serait à la fois coûteux (pour les processus peu critiques) et insuffisant (pour les processus les plus sensibles). Un exemple pédagogique simplifié, pour une organisation fictive :

| Processus | Criticité | RTO cible | RPO cible | Stratégie de résilience associée |
|---|---|---|---|---|
| Système de paie | Critique | quelques heures | quasi nul | site de secours actif, réplication continue |
| Messagerie institutionnelle | Élevée | moins d'une journée | quelques heures | sauvegardes fréquentes, restauration rapide |
| Portail pédagogique en ligne | Modérée | quelques jours | une journée | sauvegardes quotidiennes, reprise sur site secondaire passif |
| Application de gestion des archives historiques | Faible | plusieurs semaines | une semaine | sauvegardes hebdomadaires |

Ce tableau illustre un principe important : plus le RTO et le RPO cibles sont courts, plus l'architecture technique associée est coûteuse (réplication en temps réel, site de secours actif nécessitant une infrastructure dupliquée). La détermination du RTO/RPO n'est donc pas un exercice purement technique : c'est un arbitrage économique, validé par les propriétaires de processus métier et la direction, qui doit mettre en balance le coût de la résilience et le coût d'une interruption non maîtrisée.

### Étapes d'élaboration d'un PCA

1. Analyse d'impact sur l'activité (BIA — Business Impact Analysis) : identifier les processus critiques et leur RTO/RPO cible.
2. Définition des stratégies de continuité (site de secours, télétravail de crise, procédures dégradées).
3. Rédaction des plans opérationnels (qui fait quoi, avec quelles ressources).
4. Tests réguliers (exercices de crise, tests techniques de bascule).
5. Mise à jour continue.

Un PCA non testé est un PCA dont l'efficacité réelle est inconnue : les tests réguliers sont aussi importants que la rédaction du plan lui-même.

### Typologie des tests de PCA/PRA

Les tests de continuité peuvent prendre différentes formes, de complexité et de réalisme croissants, qu'il est utile de distinguer :

- **Test documentaire (walkthrough)** : relecture collective du plan par les parties prenantes, permettant de vérifier sa cohérence sans mobiliser de ressources techniques ; peu coûteux mais peu représentatif des difficultés réelles.
- **Exercice de simulation (tabletop exercise)** : les acteurs clés (RSSI, DSI, direction, communication) réagissent oralement à un scénario de crise fictif présenté par un animateur, sans bascule technique réelle ; permet de tester la coordination et la prise de décision.
- **Test technique partiel** : bascule effective d'un composant limité (par exemple, restauration d'une sauvegarde sur un environnement isolé) pour valider la faisabilité technique d'une procédure précise.
- **Test grandeur réelle (bascule complète)** : activation effective du site de secours et des procédures dégradées, dans des conditions proches d'une crise réelle ; le plus représentatif mais aussi le plus coûteux et risqué à organiser, généralement réservé aux processus les plus critiques et planifié avec grand soin pour ne pas perturber l'activité réelle.

Une pratique de maturité consiste à alterner ces différents types de test sur un cycle pluriannuel, plutôt que de se limiter systématiquement au test documentaire le moins exigeant — une tentation fréquente compte tenu du coût et de la complexité d'organisation des tests grandeur réelle.

## 4. Gestion de crise cyber

Distincte mais complémentaire du PCA/PRA : une cellule de crise dédiée (souvent associant RSSI, DSI, communication, juridique, direction) doit pouvoir être activée rapidement en cas d'incident majeur (ex. rançongiciel), avec des procédures de décision pré-établies (isoler, communiquer, notifier les autorités compétentes le cas échéant).

### 4.1 Les phases typiques d'une gestion de crise cyber

Une gestion de crise structurée s'organise généralement autour de phases successives, qu'il est utile de mémoriser pour comprendre l'articulation entre réaction technique et pilotage de gouvernance :

1. **Détection et qualification** : un événement est détecté (alerte technique, signalement d'un utilisateur, notification d'un tiers) et qualifié comme incident de sécurité potentiellement majeur, déclenchant l'activation de la cellule de crise.
2. **Endiguement (containment)** : mesures immédiates visant à limiter la propagation de l'incident (isolement de systèmes compromis, coupure de connexions réseau suspectes), en arbitrant entre l'urgence technique et la préservation des preuves nécessaires à l'investigation ultérieure.
3. **Investigation et éradication** : analyse approfondie de l'origine et de l'étendue de la compromission, puis suppression effective de la cause (réinstallation de systèmes compromis, révocation d'identifiants exposés, correction de la vulnérabilité exploitée).
4. **Communication** : information coordonnée des parties prenantes internes (direction, collaborateurs) et externes (clients, partenaires, autorités compétentes selon les obligations réglementaires applicables), pilotée conjointement par la communication, le juridique et le RSSI pour éviter les messages contradictoires.
5. **Reprise d'activité** : retour progressif à un fonctionnement normal, en s'appuyant si nécessaire sur le PCA/PRA lorsque l'incident a affecté la disponibilité des systèmes.
6. **Retour d'expérience (post-mortem)** : analyse a posteriori de la crise, sans recherche de responsabilité individuelle punitive mais dans une logique d'amélioration continue, débouchant sur des actions correctives qui alimentent le registre des risques (chapitre 2) et, le cas échéant, une révision de la PSSI.

Ce découpage en phases rappelle que la gestion de crise n'est pas qu'une affaire technique : la coordination entre acteurs très différents (techniques, juridiques, communication, direction) est souvent le facteur le plus déterminant de la capacité d'une organisation à traverser une crise cyber sans dommage disproportionné, davantage encore que la seule sophistication des outils techniques déployés.

## À retenir

- Quatre stratégies de traitement du risque : éviter, réduire, transférer, accepter — le choix relève d'un arbitrage validé par la gouvernance, et ces stratégies se combinent souvent pour un même risque plutôt que de s'exclure mutuellement.
- L'acceptation du risque doit être une décision explicite, tracée, validée au bon niveau hiérarchique et revue périodiquement — jamais une absence de décision par défaut.
- La PSSI se décline en plusieurs niveaux, de la politique générale aux procédures opérationnelles, et doit être vivante (diffusée, appropriée, appliquée, révisée) ; une politique non diffusée ni actualisée est un signe classique d'immaturité.
- RTO et RPO traduisent en exigences concrètes la criticité des processus métier, doivent être différenciés selon les processus, et dimensionnent le PCA/PRA ; un plan non testé reste théorique, et les modalités de test (documentaire, simulation, technique partiel, grandeur réelle) doivent être variées dans le temps.
- La gestion de crise cyber s'organise en phases successives (détection, endiguement, investigation, communication, reprise, retour d'expérience) qui mobilisent des compétences bien au-delà du seul champ technique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
