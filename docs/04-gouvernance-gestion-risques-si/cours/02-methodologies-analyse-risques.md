# Chapitre 2 — Méthodologies d'analyse de risques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=04-gouvernance-gestion-risques-si/slides/02-methodologies-analyse-risques.txt){ target=_blank }

## 1. Vocabulaire de base de la gestion des risques

| Terme | Définition |
|---|---|
| **Actif (asset)** | tout élément ayant de la valeur pour l'organisation (donnée, système, processus, image) |
| **Menace** | événement potentiel pouvant porter atteinte à un actif (attaque, erreur humaine, panne, catastrophe naturelle) |
| **Vulnérabilité** | faiblesse exploitable par une menace |
| **Risque** | combinaison de la probabilité qu'une menace exploite une vulnérabilité et de l'impact qui en résulterait |
| **Mesure de sécurité (contrôle)** | action réduisant la probabilité et/ou l'impact d'un risque |
| **Risque résiduel** | risque subsistant après application des mesures de sécurité |

Formule couramment utilisée pour qualifier un risque : **Risque = Probabilité × Impact** (souvent affinée par des échelles qualitatives à 3-5 niveaux plutôt qu'un calcul numérique naïf).

Il est important de noter que cette formule, bien qu'omniprésente dans la littérature et les outils pratiques, est une simplification. Elle suppose implicitement une indépendance entre probabilité et impact, alors qu'en réalité ces deux dimensions peuvent être corrélées (un attaquant très motivé, donc rendant l'occurrence plus probable, vise souvent des cibles à fort impact). Elle ne rend pas compte non plus de la vitesse de propagation d'un risque, ni de sa capacité à se combiner avec d'autres risques (effet de cascade). Ces limites justifient que les méthodologies structurées présentées dans ce chapitre ne se contentent pas d'un simple produit arithmétique, mais organisent une réflexion plus riche autour du contexte, des sources de menace et des scénarios plausibles.

### 1.1 Distinguer risque brut, risque net et risque résiduel

La terminologie de la gestion des risques distingue plusieurs états successifs d'un même risque, une distinction essentielle pour éviter les confusions fréquentes en TP :

- **Risque brut (ou inhérent)** : niveau de risque en l'absence de toute mesure de sécurité — c'est un état théorique, utile pour hiérarchiser les enjeux avant traitement.
- **Risque net (ou actuel)** : niveau de risque compte tenu des mesures de sécurité déjà en place au moment de l'analyse.
- **Risque résiduel** : niveau de risque restant après la mise en œuvre des mesures de traitement additionnelles décidées à l'issue de l'analyse (voir chapitre 3).

Une erreur fréquente chez les débutants consiste à évaluer un risque « à froid » (brut) puis à décider de le traiter sans jamais formaliser explicitement le risque net de départ, ce qui empêche ensuite de mesurer objectivement l'efficacité des mesures ajoutées.

### 1.2 Approche par les actifs, approche par la menace

Deux logiques structurent historiquement l'analyse de risques, et il est utile de les distinguer avant d'aborder les méthodes concrètes :

- **L'approche par les actifs** part d'un inventaire des biens à protéger (données, systèmes, processus), évalue leur criticité, puis identifie les menaces et vulnérabilités qui pèsent sur chacun. Elle est exhaustive mais peut devenir lourde sur un système d'information étendu, et risque de diluer l'attention sur des scénarios peu réalistes.
- **L'approche par la menace (ou par les scénarios)** part des sources de risque plausibles (attaquant, concurrent, erreur interne) et de leurs objectifs, puis reconstruit les chemins d'attaque susceptibles d'atteindre les actifs critiques. Elle est plus ciblée et opérationnelle, mais dépend de la capacité des analystes à anticiper des scénarios réalistes.

Comme on le verra, EBIOS RM privilégie la seconde approche tandis qu'ISO 27005 reste plus proche de la première, ce qui explique en partie leur complémentarité plutôt que leur substituabilité.

## 2. EBIOS Risk Manager (méthode ANSSI)

EBIOS RM structure l'analyse en cinq ateliers :

1. **Cadrage et socle de sécurité** : périmètre de l'étude, missions et valeurs métier, événements redoutés, socle de mesures de sécurité déjà en place.
2. **Sources de risque** : identification des sources de risque (attaquant opportuniste, concurrent, État, initié malveillant) et de leurs objectifs visés.
3. **Scénarios stratégiques** : chemins d'attaque à haut niveau, cartographie de l'écosystème (fournisseurs, partenaires comme vecteurs indirects).
4. **Scénarios opérationnels** : déroulé technique détaillé d'une attaque, mode opératoire probable.
5. **Traitement du risque** : synthèse des risques, décision de traitement, plan d'action et suivi.

EBIOS RM se distingue par son approche « par la menace » (partir des scénarios d'attaque plausibles) en complément d'une approche « par les actifs ».

### 2.1 Illustration pédagogique d'un scénario opérationnel (cas fictif)

Pour rendre concret l'atelier 4, considérons une organisation pédagogique fictive, la Direction des Systèmes d'Information Fictive (DSIF), gérant une application de gestion des notes des étudiants. Un scénario opérationnel plausible pourrait se dérouler ainsi : un attaquant opportuniste (source de risque identifiée en atelier 2) envoie un courriel de hameçonnage ciblé à un enseignant disposant d'un accès administrateur à l'application ; l'enseignant, trompé, saisit ses identifiants sur une page frauduleuse ; l'attaquant récupère ces identifiants et se connecte au système ; il modifie des notes ou exfiltre des données personnelles d'étudiants. Ce déroulé, volontairement simple, illustre le niveau de détail attendu en atelier 4 : chaque étape doit être suffisamment concrète pour permettre, en atelier 5, d'identifier des mesures de traitement précises (authentification multifacteur pour les comptes à privilèges, journalisation des modifications de notes, sensibilisation ciblée des enseignants disposant d'accès étendus).

### 2.2 Points d'attention méthodologiques

EBIOS RM insiste particulièrement sur deux notions qui la distinguent d'approches plus mécaniques :

- **L'écosystème** : une organisation n'est jamais isolée ; ses fournisseurs, prestataires informatiques et partenaires peuvent constituer des points d'entrée indirects vers ses actifs critiques. L'atelier 3 impose explicitement de cartographier ces parties prenantes et d'évaluer leur niveau de menace et de dépendance.
- **La vraisemblance plutôt que la seule probabilité statistique** : en l'absence de données statistiques fiables sur la fréquence d'attaques ciblées (contrairement à des risques industriels bien documentés), EBIOS RM privilégie un jugement argumenté de vraisemblance, construit collectivement par des experts métier et sécurité, plutôt qu'un calcul de probabilité prétendument objectif mais reposant sur des données fragiles.

## 3. ISO/IEC 27005

Norme internationale de référence pour la gestion des risques SSI, structurée en :

1. Établissement du contexte (critères de risque, périmètre).
2. Appréciation du risque (identification, analyse, évaluation).
3. Traitement du risque.
4. Acceptation du risque.
5. Communication et concertation.
6. Surveillance et revue.

ISO 27005 est le référentiel généralement utilisé pour nourrir le processus de gestion des risques exigé par la clause 6 de l'ISO 27001, dans une logique de certification.

### 3.1 Articulation avec l'ISO/IEC 27001 et l'Annexe A

L'ISO 27005 ne fonctionne pas seule : elle est conçue pour alimenter le système de management de la sécurité de l'information (SMSI) décrit par l'ISO 27001. Concrètement, l'appréciation des risques menée selon ISO 27005 débouche sur un **plan de traitement du risque**, qui doit se référer aux mesures de sécurité listées dans l'Annexe A de l'ISO 27001 (ou dans l'ISO 27002 pour leur description détaillée). Cette articulation a une conséquence pratique importante pour les organisations visant une certification : chaque mesure de l'Annexe A jugée non applicable doit être explicitement justifiée dans une **Déclaration d'applicabilité** (Statement of Applicability, SoA), document qui fait le pont entre l'analyse de risques et le référentiel de contrôles normalisé.

### 3.2 Le registre des risques comme livrable central

Au-delà des étapes méthodologiques, ISO 27005 aboutit concrètement à la tenue d'un **registre des risques**, document vivant recensant l'ensemble des risques identifiés, leur évaluation et leur statut de traitement. Un extrait pédagogique simplifié pourrait se présenter ainsi :

| ID | Actif concerné | Menace / scénario | Probabilité | Impact | Niveau de risque brut | Mesures existantes | Risque net | Traitement décidé | Responsable | Échéance |
|---|---|---|---|---|---|---|---|---|---|---|
| R-014 | Serveur de messagerie institutionnelle | Compromission par hameçonnage d'un compte administrateur | Probable | Majeur | Élevé | Filtrage antispam basique | Élevé | Réduire (authentification multifacteur) | RSSI | T3 |
| R-027 | Base de données des dossiers étudiants | Fuite de données par erreur de configuration d'un accès externe | Possible | Critique | Élevé | Contrôle d'accès par rôle | Modéré | Réduire (audit trimestriel des accès) | DSI | T2 |
| R-033 | Site web vitrine de l'établissement | Défacement par vulnérabilité applicative non corrigée | Possible | Modéré | Modéré | Aucune | Modéré | Accepter (impact limité, coût de correction disproportionné) | RSSI | — |

Ce registre n'a de valeur que s'il est **maintenu à jour** : réévalué périodiquement (a minima annuellement, mais aussi à chaque changement significatif du système d'information), et présenté aux instances de gouvernance décrites au chapitre 1 pour arbitrage.

## 4. NIST Risk Management Framework (RMF)

Cadre en sept étapes (Prepare, Categorize, Select, Implement, Assess, Authorize, Monitor), largement utilisé dans le secteur public américain et par des organisations internationales, avec une articulation forte entre catégorisation des systèmes (impact potentiel) et sélection de mesures de sécurité issues d'un catalogue (NIST SP 800-53).

Le RMF se distingue par sa logique de **catégorisation préalable** : avant même d'analyser des scénarios de menace précis, chaque système d'information est classé selon un niveau d'impact potentiel (faible, modéré, élevé) sur trois critères — confidentialité, intégrité, disponibilité. Cette catégorisation détermine ensuite directement l'ensemble minimal de mesures de sécurité à sélectionner dans le catalogue NIST SP 800-53, avant d'être ajustées au contexte spécifique. Cette approche « par catalogue » présente l'avantage d'une grande reproductibilité et facilite les comparaisons entre systèmes similaires, mais elle est parfois critiquée pour son caractère plus mécanique, moins centré sur la créativité des scénarios d'attaque que ne l'est EBIOS RM.

## 5. Comparaison synthétique

| Critère | EBIOS RM | ISO 27005 | NIST RMF |
|---|---|---|---|
| Origine | France (ANSSI) | International (ISO) | États-Unis (NIST) |
| Approche dominante | par la menace (scénarios d'attaque) | par les actifs et le contexte | par la catégorisation et un catalogue de contrôles |
| Usage typique | administrations, OIV, grandes entreprises francophones | certification ISO 27001 | secteur public américain, sous-traitants fédéraux |

Ces méthodes ne sont pas mutuellement exclusives : une organisation peut s'appuyer sur ISO 27005 pour son SMSI certifié tout en utilisant les ateliers EBIOS RM pour affiner l'analyse des scénarios d'attaque les plus critiques.

### 5.1 Autres approches complémentaires à connaître

Au-delà des trois méthodologies principales détaillées ci-dessus, il est utile de connaître, au moins de nom, d'autres approches fréquemment citées dans la littérature et les organisations :

- **MEHARI** (Méthode Harmonisée d'Analyse de Risques), développée par le CLUSIF, propose une approche fondée sur des questionnaires structurés d'évaluation des mesures de sécurité et un calcul semi-quantitatif du risque ; elle a historiquement été très utilisée dans les organisations francophones avant la montée en puissance d'EBIOS RM.
- **OCTAVE** (Operationally Critical Threat, Asset, and Vulnerability Evaluation), développée par le CERT/SEI américain, met l'accent sur une analyse pilotée par les équipes métier elles-mêmes plutôt que par des experts techniques externes, dans une logique d'appropriation interne du risque.
- **FAIR** (Factor Analysis of Information Risk) propose une approche quantitative visant à exprimer le risque en termes financiers (perte annuelle attendue), utile pour dialoguer avec des directions financières habituées à raisonner en probabilités et montants monétaires plutôt qu'en échelles qualitatives.

Ces méthodes ne sont pas approfondies dans ce cours, mais leur existence illustre que le choix méthodologique reste un sujet actif, dépendant de la culture de l'organisation, de son secteur et de la maturité de son dialogue entre sécurité et direction financière.

## 6. Échelles d'évaluation

Un exemple d'échelle qualitative simple à 4 niveaux, utilisable en TP :

| Niveau | Probabilité | Impact |
|---|---|---|
| 1 | Rare | Négligeable |
| 2 | Possible | Modéré |
| 3 | Probable | Majeur |
| 4 | Quasi certain | Critique (met en péril la continuité) |

La combinaison probabilité × impact positionne chaque risque sur une **matrice de criticité**, outil central de priorisation (repris en TP2).

### 6.1 Exemple de matrice de criticité et son usage

Une matrice de criticité croise les niveaux de probabilité (en lignes) et d'impact (en colonnes) pour produire un niveau de risque global, généralement codé par couleur pour faciliter la lecture par un public non spécialiste (vert, jaune, orange, rouge). Un exemple pédagogique construit à partir de l'échelle à 4 niveaux ci-dessus :

| Probabilité \ Impact | Négligeable | Modéré | Majeur | Critique |
|---|---|---|---|---|
| **Rare** | Faible | Faible | Modéré | Modéré |
| **Possible** | Faible | Modéré | Élevé | Élevé |
| **Probable** | Modéré | Élevé | Élevé | Critique |
| **Quasi certain** | Modéré | Élevé | Critique | Critique |

L'intérêt principal de cet outil n'est pas tant sa précision arithmétique — les échelles qualitatives restent par nature approximatives — que sa capacité à **prioriser objectivement** un grand nombre de risques identifiés lors d'un atelier collectif, en évitant que la discussion ne se perde dans des débats interminables sur des risques mineurs au détriment des risques réellement critiques. En pratique, une organisation fixera un seuil (par exemple : tout risque « Élevé » ou « Critique » doit faire l'objet d'un plan de traitement formalisé et d'un suivi en comité de pilotage, tandis qu'un risque « Faible » peut être accepté sans formalisme excessif).

### 6.2 Limites et précautions d'usage des échelles qualitatives

Les échelles qualitatives, malgré leur simplicité d'usage, comportent des biais qu'il convient de connaître pour les utiliser à bon escient en TP comme en pratique professionnelle :

- **Le biais d'ancrage** : les participants à un atelier d'évaluation ont tendance à converger vers les niveaux intermédiaires par prudence, ce qui peut aplatir artificiellement les écarts entre risques réellement très différents.
- **Le manque de granularité aux extrêmes** : deux risques classés tous deux « Critique » peuvent en réalité avoir des ordres de grandeur d'impact très différents (un impact critique pouvant être dix fois supérieur à un autre) sans que la matrice qualitative ne le distingue.
- **La subjectivité de l'évaluation** : la probabilité et l'impact restent des jugements d'experts, potentiellement divergents selon les participants ; c'est pourquoi les ateliers collectifs (comme ceux d'EBIOS RM) sont préférables à une évaluation individuelle isolée, afin de confronter les points de vue et d'aboutir à un consensus argumenté.

## À retenir

- Risque = fonction de la probabilité d'occurrence et de l'impact, jamais l'un sans l'autre ; il convient de distinguer risque brut, risque net et risque résiduel selon l'étape du traitement.
- EBIOS RM structure l'analyse « par la menace » en cinq ateliers, en portant une attention particulière à l'écosystème (fournisseurs, partenaires) ; ISO 27005 structure l'analyse « par le contexte et les actifs » en six étapes et s'articule directement avec le SMSI de l'ISO 27001 via la Déclaration d'applicabilité.
- Le choix de la méthode dépend souvent du contexte réglementaire et sectoriel de l'organisation ; d'autres approches (MEHARI, OCTAVE, FAIR) existent et répondent à des besoins spécifiques, notamment la quantification financière du risque.
- Le registre des risques et la matrice de criticité sont les deux outils opérationnels centraux qui traduisent la méthodologie choisie en un support de priorisation et de suivi concret, mais leurs limites (biais d'ancrage, subjectivité) doivent être connues et discutées collectivement.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
