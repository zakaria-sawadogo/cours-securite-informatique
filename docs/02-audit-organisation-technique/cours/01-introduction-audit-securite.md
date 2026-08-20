# Chapitre 1 — Introduction à l'audit de sécurité

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/01-introduction-audit-securite.txt){ target=_blank }

## 1. Définition

Un audit de sécurité est une évaluation systématique et indépendante du niveau de sécurité d'un système d'information, visant à vérifier sa conformité à un référentiel (normatif, réglementaire ou contractuel) et à identifier les écarts entre l'état constaté et l'état attendu.

Trois mots de cette définition méritent d'être précisés, car ils distinguent un audit d'une simple prestation technique :

- **Systématique** : l'audit suit une méthodologie reproductible (checklist, référentiel, guide de test) et non une exploration au hasard. Deux auditeurs différents, appliquant la même méthodologie sur le même périmètre, doivent aboutir à des constats globalement comparables.
- **Indépendante** : l'auditeur ne doit pas être juge et partie. Un administrateur système qui « audite » sa propre configuration peut avoir des angles morts (biais de confirmation, réticence à pointer ses propres manquements) ; c'est pourquoi les référentiels de certification exigent souvent une séparation entre l'équipe qui exploite le système et l'équipe qui l'audite, voire un auditeur externe.
- **Référentiel** : un audit ne se fait jamais « dans l'absolu ». Il compare toujours un existant à un cadre de référence explicite — une norme (ISO/IEC 27001), un standard technique (CIS Benchmarks), une réglementation (RGPD) ou un cahier des charges contractuel. Sans référentiel affiché, un rapport d'audit n'est qu'une opinion, difficile à contester mais aussi difficile à faire reconnaître par un tiers (assureur, régulateur, client).

Il est utile de distinguer l'audit de sécurité d'activités voisines avec lesquelles il est parfois confondu :

- Le **test d'intrusion (pentest)** est une composante technique de l'audit, centrée sur la démonstration d'exploitabilité, mais un audit peut être bien plus large (revue organisationnelle, revue de configuration sans exploitation active).
- Le **contrôle de conformité** (*compliance check*) vérifie mécaniquement des cases d'une checklist réglementaire ; un audit de sécurité va au-delà en évaluant l'efficacité réelle des mesures, pas seulement leur existence formelle.
- La **veille de vulnérabilités** (scan récurrent, bulletin de sécurité) est un flux d'information continu, alors que l'audit est un exercice ponctuel et structuré, avec un début, une fin et un livrable.

## 2. Pourquoi auditer ?

- **Conformité réglementaire** : obligations légales (protection des données, secteurs régulés — banque, santé).
- **Exigence contractuelle** : un client ou partenaire exige un niveau de sécurité prouvé (ex. certification ISO 27001, conformité PCI-DSS pour le paiement en ligne).
- **Gestion des risques** : identifier les vulnérabilités avant qu'un attaquant ne les exploite.
- **Amélioration continue** : mesurer les progrès entre deux audits successifs.

Au-delà de ces motivations « officielles », un auditeur doit garder à l'esprit que la commande d'un audit répond souvent à un déclencheur concret : préparation d'une certification, exigence d'un investisseur ou d'un assureur cyber, suite à un incident de sécurité (même mineur), changement de RSSI, ou encore obligation contractuelle imposée par un grand donneur d'ordre à ses sous-traitants. Comprendre ce déclencheur aide l'auditeur à calibrer le ton du rapport et les priorités de la restitution : un audit demandé après un incident attend des réponses rapides et concrètes, tandis qu'un audit de certification attend une couverture exhaustive du référentiel, y compris sur des points formels.

Il faut aussi rappeler ce qu'un audit **n'est pas** : ce n'est pas une garantie d'absence de vulnérabilité (un audit couvre un périmètre et une période donnés, avec les limites de temps et d'outillage propres à toute prestation), et ce n'est pas un exercice à charge contre les équipes techniques. Un audit mal compris par les équipes auditées génère de la défiance et nuit à la qualité des entretiens et de la collecte d'information ; une bonne communication en amont sur les objectifs de l'audit (amélioration, pas sanction) conditionne largement sa réussite.

## 3. Les grandes familles d'audit

| Type d'audit | Objet | Exemple de livrable |
|---|---|---|
| **Audit organisationnel** | politiques, procédures, gouvernance | rapport de conformité ISO 27001 |
| **Audit technique** | configuration des systèmes, réseaux, applications | rapport de test d'intrusion |
| **Audit de conformité** | respect d'une réglementation précise | rapport PCI-DSS, RGPD |
| **Audit de code** | revue du code source | rapport SAST (analyse statique) |
| **Audit physique** | sécurité des locaux, contrôle d'accès | rapport d'inspection physique |

Ce module couvre principalement les deux premières familles, qui se complètent : un système peut être « techniquement » bien configuré mais dépourvu de procédures (pas de gestion des correctifs formalisée), ou disposer de belles procédures jamais appliquées en pratique.

Ces familles ne sont pas des catégories imperméables ; en pratique, une mission d'audit combine souvent plusieurs volets. Un audit de conformité PCI-DSS, par exemple, embarque à la fois des exigences organisationnelles (politique de sécurité formalisée, gestion des accès) et des exigences techniques précises (segmentation réseau, chiffrement des données de carte, scans de vulnérabilités trimestriels). De même, un audit physique n'est pas anecdotique en sécurité de l'information : un attaquant qui accède physiquement à une salle serveur non protégée, ou qui retrouve des documents sensibles jetés sans destruction (technique dite du *dumpster diving*), contourne d'un coup toutes les mesures logiques mises en place. Un auditeur expérimenté garde toujours à l'esprit cette complémentarité et signale, même hors périmètre strict de sa mission, les risques évidents relevant d'une autre famille d'audit.

### Approfondissement : audit vs évaluation continue

La tendance actuelle en cybersécurité est de compléter les audits ponctuels (photographie à un instant *t*) par une évaluation continue : scans de vulnérabilités automatisés récurrents, plateformes de gestion de la surface d'attaque externe (*Attack Surface Management*), programmes de *bug bounty*. L'audit périodique garde toutefois un rôle irremplaçable : il mobilise un regard humain et expert, capable d'identifier des failles logiques ou organisationnelles qu'aucun outil automatisé ne détecte, et il produit un livrable formel opposable (utile pour une certification, un dossier d'assurance ou une preuve de diligence raisonnable).

## 4. Les acteurs et postures

- **Auditeur interne** : appartient à l'organisation auditée, connaît le contexte, indépendance relative.
- **Auditeur externe** : société tierce, plus grande indépendance, souvent exigée par les référentiels de certification.
- **Boîte noire (black box)** : l'auditeur ne dispose d'aucune information préalable, simule un attaquant externe.
- **Boîte grise (grey box)** : informations partielles (comptes utilisateurs, documentation), simule une menace interne ou un attaquant ayant déjà un premier accès.
- **Boîte blanche (white box)** : accès complet (code source, architecture, comptes admin), maximise la couverture de l'audit.

Le choix entre ces trois modes d'accès n'est pas qu'une question de réalisme du scénario : c'est aussi un arbitrage coût/couverture. Un audit en boîte noire se rapproche le plus d'une attaque réelle mais consomme une part importante du temps disponible en pure reconnaissance, au détriment de l'analyse en profondeur ; à budget de temps égal, il couvre donc généralement moins de vulnérabilités qu'un audit en boîte blanche. À l'inverse, un audit en boîte blanche maximise la couverture mais peut manquer certains constats liés à l'exposition réelle du système (ce qu'un attaquant externe verrait en premier). Un bon cadrage d'audit précise explicitement le mode retenu et le justifie par rapport à l'objectif de la mission :

| Objectif de la mission | Mode d'accès généralement recommandé |
|---|---|
| Simuler une attaque externe réaliste, tester la détection | boîte noire |
| Simuler un employé malveillant ou un compte compromis | boîte grise |
| Certification, revue de sécurité exhaustive avant mise en production | boîte blanche |
| Audit de code source d'une application critique | boîte blanche (obligatoire pour le SAST et la revue manuelle) |

### Piège courant : confondre indépendance et compétence

Un auditeur externe n'est pas automatiquement plus compétent qu'une équipe interne — il est simplement structurellement plus indépendant du résultat de l'audit (il n'a pas construit le système qu'il évalue, et sa rémunération ne dépend pas d'un satisfecit). Il reste indispensable de vérifier les compétences et références de l'auditeur (certifications reconnues comme OSCP, CEH, ou l'agrément PASSI dans le cadre du référentiel ANSSI) : l'indépendance garantit l'absence de conflit d'intérêt, pas la qualité technique du travail rendu.

## 5. Cadre légal et éthique

Tout audit technique — en particulier un test d'intrusion — implique des actions qui, sans autorisation, seraient qualifiées d'accès et de maintien frauduleux dans un système de traitement automatisé de données. Un audit ne peut être mené que dans le cadre d'un **mandat écrit** précisant :

- le périmètre exact autorisé (adresses IP, applications, plages horaires) ;
- les techniques autorisées et interdites (ex. déni de service exclu par défaut) ;
- les responsables à contacter en cas d'incident pendant l'audit ;
- les modalités de confidentialité et de destruction des données collectées.

Ce document est généralement appelé **règles d'engagement** (*rules of engagement*) et doit être signé par une personne réellement habilitée à autoriser des tests sur le périmètre concerné — pas seulement un interlocuteur technique de bonne volonté. Dans le cas fréquent d'une infrastructure hébergée chez un tiers (fournisseur Cloud, hébergeur mutualisé), l'autorisation du seul client ne suffit généralement pas : la plupart des fournisseurs Cloud imposent une procédure de notification préalable, voire une autorisation explicite de leur part, avant tout test d'intrusion sur les ressources qu'ils hébergent, sous peine de suspension du compte ou de poursuites.

D'autres bonnes pratiques éthiques et contractuelles complètent le mandat :

- **Clause de non-divulgation (NDA)** : l'auditeur a nécessairement accès à des informations sensibles (failles non corrigées, données de configuration, parfois des données à caractère personnel rencontrées incidemment) ; leur confidentialité doit être contractualisée.
- **Divulgation responsable** : si l'audit révèle une vulnérabilité critique susceptible d'affecter des tiers (ex. faille dans un composant largement diffusé), l'auditeur doit suivre un processus de divulgation coordonnée plutôt qu'une publication immédiate, afin de laisser le temps de corriger.
- **Fenêtre de test et point de contact d'urgence** : convenir d'une plage horaire et d'un contact joignable en cas d'incident (service dégradé, alerte déclenchée chez un partenaire de supervision) évite qu'un test légitime ne déclenche une réponse à incident disproportionnée côté client.

### Piège courant : le périmètre implicite

Une erreur fréquente, y compris chez des auditeurs expérimentés, consiste à considérer que « tout ce qui appartient à l'entreprise » est dans le périmètre. Or une même adresse IP ou un même nom de domaine peut être partagé avec des tiers (hébergement mutualisé, sous-traitant, filiale non couverte par le mandat) : tester au-delà du périmètre écrit, même de bonne foi, expose l'auditeur à une responsabilité pénale et civile. En cas de doute sur l'appartenance d'un actif au périmètre, la règle est simple : ne pas tester et faire confirmer par écrit auprès du donneur d'ordre.

## 6. Le cycle de vie d'un audit

1. **Cadrage** : définition du périmètre, des objectifs, du mandat.
2. **Collecte d'information / reconnaissance**.
3. **Analyse** (organisationnelle et/ou technique).
4. **Identification et qualification des écarts/vulnérabilités**.
5. **Rédaction du rapport** (constats, criticité, recommandations).
6. **Restitution** et plan d'action.
7. **Suivi** (contre-audit / vérification de la remédiation).

Ce cycle, bien qu'illustré ici de façon linéaire, est en pratique itératif : l'analyse d'un premier constat peut relancer une phase de collecte d'information complémentaire (par exemple, la découverte d'un serveur non documenté oblige à revenir en reconnaissance), et la rédaction du rapport fait souvent remonter des questions qui nécessitent de revérifier un constat avant publication. Un point souvent sous-estimé par les étudiants est l'étape de **cadrage** : c'est elle qui détermine la durée réaliste de la mission, les ressources nécessaires et les livrables attendus. Un cadrage bâclé (périmètre flou, objectifs mal définis) est la cause la plus fréquente d'audits qui dépassent leur budget de temps ou qui livrent un rapport qui ne répond pas aux attentes du commanditaire.

Chaque phase produit généralement un artefact intermédiaire qui alimente les suivantes : le cadrage produit le mandat et les règles d'engagement ; la reconnaissance produit une cartographie du périmètre (inventaire d'actifs, schéma d'architecture reconstitué) ; l'analyse produit une liste brute de constats ; la qualification y ajoute criticité et preuve ; le rapport en fait une synthèse actionnable. Les chapitres suivants de ce module détaillent chacune de ces phases : le chapitre 2 pour l'audit organisationnel, les chapitres 3 à 5 pour les différentes déclinaisons de l'audit technique, et le chapitre 6 pour la rédaction du rapport et le suivi de la remédiation.

## À retenir

- Un audit combine une dimension organisationnelle et une dimension technique, qui se complètent plus qu'elles ne se substituent l'une à l'autre.
- Le mandat écrit et les règles d'engagement sont un prérequis non négociable à toute activité d'audit technique, y compris vis-à-vis d'un fournisseur Cloud tiers.
- Le mode d'accès (black/grey/white box) détermine la couverture et le réalisme de l'audit : il s'agit d'un arbitrage explicite à documenter dans le cadrage, pas d'un simple détail technique.
- L'indépendance de l'auditeur (interne ou externe) garantit l'absence de conflit d'intérêt, mais ne remplace pas la vérification de sa compétence.
- Le cycle de vie de l'audit est itératif : un audit qui ne prévoit pas de phase de suivi (contre-audit) reste une photographie isolée sans garantie de progrès réel.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
