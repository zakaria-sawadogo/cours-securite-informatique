# Chapitre 2 — Audit organisationnel : normes et référentiels

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/02-audit-organisationnel-referentiels.txt){ target=_blank }

## 1. ISO/IEC 27001 et 27002

- **ISO/IEC 27001** : norme certifiable définissant les exigences d'un Système de Management de la Sécurité de l'Information (SMSI). Structurée autour du cycle **PDCA** (Plan-Do-Check-Act).
- **ISO/IEC 27002** : catalogue de mesures de sécurité (bonnes pratiques) organisées en thèmes (organisationnel, humain, physique, technologique) que l'ISO 27001 référence.

La distinction entre ces deux normes est une source fréquente de confusion chez les auditeurs débutants : l'ISO 27001 est une norme de **management** — elle décrit un système de pilotage (comment identifier les risques, décider des mesures à appliquer, vérifier leur efficacité, s'améliorer) — alors que l'ISO 27002 est une bibliothèque de **mesures concrètes** (contrôles) dans laquelle on peut piocher pour construire la « Déclaration d'applicabilité » (*Statement of Applicability*, SoA) exigée par l'ISO 27001. Une organisation peut être **certifiée ISO 27001** (le système de management est audité et jugé conforme), mais on ne « certifie » pas l'ISO 27002 en tant que telle : elle sert de référence de bonnes pratiques, y compris hors de toute démarche de certification.

Le cycle **PDCA** structure la logique d'amélioration continue du SMSI :

- **Plan** : établir le contexte, apprécier les risques, définir la politique de sécurité et les objectifs.
- **Do** : mettre en œuvre les mesures de traitement du risque retenues (contrôles techniques et organisationnels).
- **Check** : surveiller et mesurer l'efficacité du SMSI (audits internes, indicateurs, revue de direction).
- **Act** : corriger les non-conformités identifiées et faire évoluer le SMSI en continu.

### Clauses principales de l'ISO 27001 (4 à 10)

| Clause | Contenu |
|---|---|
| 4 | Contexte de l'organisation |
| 5 | Leadership (engagement de la direction, politique SSI) |
| 6 | Planification (appréciation et traitement des risques) |
| 7 | Support (ressources, compétences, sensibilisation) |
| 8 | Fonctionnement (mise en œuvre opérationnelle) |
| 9 | Évaluation des performances (audit interne, revue de direction) |
| 10 | Amélioration (non-conformités, actions correctives) |

Un audit organisationnel vérifie, pour chaque clause, l'existence de **preuves documentaires** (politiques, procédures) et de **preuves de mise en œuvre effective** (entretiens, échantillons de tickets, journaux d'activité).

Ces sept clauses (4 à 10) constituent le corps normatif « obligatoire » de l'ISO 27001 : une organisation candidate à la certification doit démontrer sa conformité à chacune d'elles, sans exception, quelle que soit sa taille ou son secteur. C'est ce qui différencie l'ISO 27001 de nombreux autres référentiels sectoriels dont certaines exigences peuvent être exclues selon le contexte. L'annexe A de la norme (dans sa version 2022, réorganisée en quatre thèmes : organisationnel, humain, physique, technologique) liste les 93 contrôles de référence — c'est cette annexe qui reprend, en version condensée, le contenu détaillé de l'ISO 27002. Un point technique important pour l'auditeur : une organisation peut légitimement exclure un contrôle de l'annexe A de sa Déclaration d'applicabilité, à condition de justifier cette exclusion (par exemple, les contrôles relatifs au développement logiciel sécurisé peuvent être hors périmètre pour une organisation qui n'a aucune activité de développement en interne) — l'auditeur doit vérifier que chaque exclusion est argumentée et cohérente avec le contexte réel de l'organisation, et non utilisée pour esquiver un point sensible.

### Piège courant : confondre certification et sécurité réelle

Une organisation certifiée ISO 27001 n'est pas nécessairement « sécurisée » au sens absolu : la certification atteste que le système de management fonctionne (les risques sont identifiés, traités, suivis), pas que le niveau de risque résiduel est nul. Une organisation peut être parfaitement certifiée tout en ayant accepté consciemment un risque élevé sur un actif secondaire, si cette acceptation est documentée et validée par la direction dans le cadre du processus de traitement du risque. L'auditeur doit donc toujours distinguer, dans son évaluation, la conformité au **processus** de management du risque et le niveau de risque **résiduel** effectivement accepté.

## 2. Autres référentiels usuels

| Référentiel | Portée | Usage typique |
|---|---|---|
| **PCI-DSS** | données de cartes bancaires | commerçants et prestataires de paiement |
| **NIST Cybersecurity Framework** | gestion du risque cyber (Identify, Protect, Detect, Respond, Recover) | référence internationale, souvent utilisée hors certification |
| **ANSSI (guides et référentiels)** | administrations et OIV en France, largement repris en Afrique francophone | hygiène informatique, PASSI (prestataires d'audit) |
| **RGPD** | protection des données personnelles | conformité juridique, articulé avec la sécurité technique |
| **CIS Controls** | mesures techniques priorisées | audit technique et durcissement (*hardening*) |

Il est utile de comprendre la nature différente de chacun de ces référentiels, car cela conditionne la façon dont un auditeur doit les mobiliser :

- **PCI-DSS** est un standard **contractuel** porté par les grands réseaux de cartes bancaires (et non par un organisme de normalisation international) : il est très prescriptif (règles précises de segmentation réseau, de chiffrement, de rétention de logs) et son non-respect peut entraîner la perte du droit à traiter des paiements par carte, indépendamment de toute action judiciaire. Il distingue plusieurs niveaux de conformité selon le volume de transactions traitées, avec des modalités d'évaluation différentes (auto-évaluation via questionnaire ou audit par un évaluateur qualifié QSA).
- Le **NIST Cybersecurity Framework** (CSF) n'est pas certifiable au sens strict : c'est un cadre de référence, structuré en cinq (puis six dans sa version 2.0, avec l'ajout de la fonction *Govern*) fonctions — Identifier, Protéger, Détecter, Répondre, Récupérer — utilisé pour structurer une démarche de gestion du risque cyber et communiquer sur la maturité d'une organisation, souvent en complément d'une norme certifiable comme l'ISO 27001.
- Les référentiels de l'**ANSSI** (guide d'hygiène informatique, référentiel PASSI pour les prestataires d'audit) sont particulièrement structurants dans l'espace francophone : le guide d'hygiène informatique, en particulier, propose une liste de mesures priorisées et accessibles, souvent utilisée comme grille de premier niveau avant une démarche plus lourde de type ISO 27001.
- Le **RGPD** n'est pas à proprement parler un référentiel de sécurité mais un texte réglementaire relatif à la protection des données personnelles ; il impose cependant des obligations de sécurité explicites (article 32 : mesures techniques et organisationnelles appropriées), ce qui en fait un objet d'audit à part entière, souvent mené conjointement par un auditeur sécurité et un correspondant/délégué à la protection des données (DPO).
- Les **CIS Controls** et les **CIS Benchmarks** associés sont des référentiels très opérationnels, orientés configuration technique, qui seront largement mobilisés dans les chapitres consacrés à l'audit technique (chapitres 3 et 4).

### Approfondissement : comment choisir le bon référentiel

Face à une organisation qui n'a jamais formalisé de démarche sécurité, l'auditeur doit souvent recommander un référentiel adapté à sa maturité et à son secteur plutôt que d'imposer d'emblée le plus exigeant. Quelques repères pratiques :

- Une petite structure sans obligation réglementaire particulière peut démarrer efficacement avec un guide d'hygiène informatique (type ANSSI) ou les CIS Controls, plus accessibles qu'une certification complète.
- Une organisation qui traite des paiements par carte n'a pas le choix : PCI-DSS s'impose contractuellement dès lors qu'elle stocke, traite ou transmet des données de carte.
- Une organisation qui vise un marché international ou une clientèle exigeante (banque, assurance, grand groupe) a souvent intérêt à viser l'ISO 27001, référentiel le plus reconnu et le plus universellement audité par les tiers.
- Ces référentiels ne sont pas exclusifs : une même organisation peut être certifiée ISO 27001, se référer au NIST CSF pour communiquer avec des partenaires anglophones, et appliquer PCI-DSS sur son périmètre de paiement.

## 3. Méthodologie d'un audit organisationnel

1. **Revue documentaire** : politiques de sécurité, chartes, procédures de gestion des incidents, plan de continuité d'activité (PCA/PRA).
2. **Entretiens** avec les parties prenantes (RSSI, DSI, RH, métiers) pour vérifier l'application réelle des procédures.
3. **Échantillonnage** : contrôle de tickets, de comptes utilisateurs, de journaux, de contrats prestataires.
4. **Analyse d'écart (gap analysis)** entre l'état constaté et les exigences du référentiel.
5. **Notation de maturité**, souvent sur une échelle (ex. modèle CMMI adapté : 0 = inexistant à 5 = optimisé).

Chacune de ces étapes appelle des précisions méthodologiques importantes pour un auditeur en formation.

**La revue documentaire** ne se limite pas à vérifier qu'un document existe : il faut vérifier sa date de dernière mise à jour (une politique de sécurité qui n'a pas été révisée depuis cinq ans est un signal d'alerte, même si son contenu reste globalement pertinent), son niveau d'approbation (a-t-elle été validée par la direction, ou reste-t-elle un brouillon jamais formellement adopté ?) et sa diffusion effective (les collaborateurs concernés savent-ils qu'elle existe et où la trouver ?).

**Les entretiens** sont l'outil principal pour distinguer le document de la pratique réelle. Un bon auditeur prépare une grille de questions par référentiel et par interlocuteur, mais reste capable de rebondir sur les réponses (« Pouvez-vous me montrer un exemple concret ? », « Que se passe-t-il si un ticket de demande d'accès n'est pas traité dans le délai prévu ? »). Les questions ouvertes révèlent davantage que les questions fermées : demander « comment gérez-vous le départ d'un collaborateur ? » donne beaucoup plus d'information que « avez-vous une procédure de départ ? » (à laquelle la réponse sera presque toujours « oui »).

**L'échantillonnage** est indispensable car il est rarement possible (ni utile) de vérifier 100 % des cas : sur une population de 500 comptes utilisateurs, un auditeur sélectionnera par exemple un échantillon représentatif (comptes à privilèges, comptes créés récemment, comptes de prestataires externes) plutôt que de tout contrôler. La méthode d'échantillonnage doit être documentée dans le rapport, afin que le lecteur comprenne la portée réelle du constat (« sur un échantillon de 20 comptes à privilèges contrôlés, 6 n'avaient pas fait l'objet d'une revue depuis plus d'un an » est plus informatif et plus honnête que « la revue des comptes est insuffisante »).

**L'analyse d'écart** consiste à confronter systématiquement chaque exigence du référentiel à l'état constaté ; elle se matérialise généralement dans un tableau de correspondance (souvent appelé *compliance matrix*).

**La notation de maturité** permet de synthétiser un grand nombre de constats en une vision d'ensemble exploitable par la direction, à condition que l'échelle utilisée soit clairement définie et appliquée de façon cohérente d'un domaine à l'autre.

### Une échelle de maturité détaillée (exemple)

| Niveau | Libellé | Description |
|---|---|---|
| 0 | Inexistant | Aucune mesure, aucune conscience du risque associé |
| 1 | Initial / ad hoc | Mesures ponctuelles, non formalisées, dépendantes d'individus |
| 2 | Reproductible | Pratiques répétées mais peu documentées, pas de contrôle systématique |
| 3 | Défini | Procédures documentées, diffusées et appliquées de façon cohérente |
| 4 | Maîtrisé | Efficacité mesurée par des indicateurs, écarts détectés et corrigés |
| 5 | Optimisé | Amélioration continue, benchmarking, anticipation des risques émergents |

### Piège courant : l'entretien complaisant

Un piège classique de l'audit organisationnel est l'entretien où l'interlocuteur, consciemment ou non, présente une version idéalisée des pratiques. Pour limiter ce biais, l'auditeur doit systématiquement croiser les déclarations des entretiens avec des preuves tangibles (extraits de journaux, captures d'écran d'outils de gestion des accès, tickets réellement clôturés) plutôt que de se satisfaire d'une affirmation verbale. La règle d'or de l'audit — *« trust, but verify »* (faire confiance, mais vérifier) — s'applique particulièrement à ce type d'entretien.

## 4. Exemple de grille d'évaluation simplifiée

| Domaine | Exigence | Constat | Niveau de maturité (0-5) |
|---|---|---|---|
| Gestion des accès | Revue périodique des droits | Aucune revue formalisée depuis 18 mois | 1 |
| Gestion des correctifs | Politique de patch management documentée | Politique existe, appliquée à 60 % des serveurs | 2 |
| Sensibilisation | Formation annuelle du personnel | Formation dispensée à l'embauche uniquement | 2 |

Une bonne pratique consiste à accompagner chaque ligne de ce type de grille d'une **preuve** (référence au document consulté, extrait d'entretien, capture d'écran) et d'une **recommandation** rattachée directement à l'écart constaté, plutôt que de renvoyer systématiquement vers une synthèse générale en fin de rapport. Voici un exemple enrichi pour la première ligne du tableau ci-dessus :

- **Preuve** : entretien avec le responsable des systèmes le [date], confirmé par l'absence de tout compte rendu de revue des droits dans le registre de gouvernance fourni.
- **Risque associé** : des comptes obsolètes (anciens employés, anciens prestataires) peuvent conserver des accès actifs, augmentant la surface d'attaque et le risque d'accès non autorisé.
- **Recommandation** : instaurer une revue trimestrielle des droits d'accès, formalisée par un compte rendu signé du responsable de chaque périmètre applicatif, avec un focus prioritaire sur les comptes à privilèges.

## 5. Risques d'un audit organisationnel mal mené

- Se limiter à la lecture des documents sans vérifier l'application réelle (« conformité de façade »).
- Confondre existence d'une procédure et efficacité de la procédure.
- Absence de priorisation : toutes les non-conformités ne présentent pas le même risque.

À ces trois risques classiques s'ajoutent d'autres écueils fréquents chez les auditeurs en formation :

- **Le formalisme excessif** : produire un rapport de 150 pages qui reprend mécaniquement chaque clause du référentiel sans hiérarchisation ni synthèse exploitable, au détriment de la lisibilité pour la direction.
- **L'oubli du contexte métier** : recommander une mesure de sécurité techniquement irréprochable mais totalement disproportionnée par rapport à la taille, au budget ou à l'activité réelle de l'organisation auditée (par exemple, exiger d'une PME de dix salariés un centre opérationnel de sécurité 24/7).
- **La confusion entre non-conformité et vulnérabilité** : une non-conformité organisationnelle (absence de procédure documentée) n'est pas automatiquement synonyme de vulnérabilité exploitable ; il faut relier explicitement l'écart organisationnel au risque technique concret qu'il engendre pour que le constat soit compréhensible et actionnable.

## À retenir

- L'audit organisationnel évalue des processus et de la documentation, pas uniquement des configurations techniques ; il repose sur trois sources de preuve complémentaires — documents, entretiens et échantillons — qu'il faut systématiquement croiser.
- ISO 27001 (norme de management, certifiable) et ISO 27002 (catalogue de mesures) sont complémentaires et se combinent souvent avec des référentiels sectoriels (PCI-DSS) ou nationaux (ANSSI) selon le contexte de l'organisation auditée.
- Le choix du référentiel doit être adapté à la maturité, au secteur et aux obligations contractuelles ou réglementaires de l'organisation ; il n'existe pas de référentiel universellement optimal.
- Une non-conformité doit toujours être qualifiée par un niveau de criticité, une preuve tangible et une recommandation actionnable — jamais énoncée comme une simple observation isolée.
- La certification à un référentiel atteste la conformité d'un processus de management du risque, pas l'absence de risque résiduel : l'auditeur doit distinguer ces deux notions.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
