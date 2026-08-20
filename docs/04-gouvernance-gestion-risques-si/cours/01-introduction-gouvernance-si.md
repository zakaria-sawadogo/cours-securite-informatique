# Chapitre 1 — Introduction à la gouvernance de la sécurité de l'information

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=04-gouvernance-gestion-risques-si/slides/01-introduction-gouvernance-si.txt){ target=_blank }

## 1. Définition de la gouvernance SSI

La gouvernance de la sécurité de l'information désigne l'ensemble des structures, processus et mécanismes de décision qui permettent à une organisation de définir, piloter et faire évoluer sa stratégie de sécurité, en cohérence avec ses objectifs métiers et son appétence au risque. Elle se distingue de la sécurité opérationnelle (mise en œuvre technique quotidienne) par son niveau : la gouvernance définit le « quoi » et le « pourquoi », l'opérationnel exécute le « comment ».

Il est utile de distinguer trois niveaux imbriqués, souvent confondus par les étudiants et parfois par les praticiens eux-mêmes :

- **Le niveau stratégique (gouvernance)** : il fixe les orientations, valide l'appétence au risque, alloue les ressources et rend compte aux instances de gouvernance (conseil d'administration, comité exécutif). Il répond à la question « où voulons-nous aller et combien sommes-nous prêts à investir ? ».
- **Le niveau tactique (management de la sécurité)** : il traduit les orientations stratégiques en programme d'action — politique, gestion des risques, plan de traitement, tableaux de bord. C'est le niveau du RSSI en tant que chef d'orchestre.
- **Le niveau opérationnel (sécurité technique)** : il met en œuvre concrètement les mesures — configuration des pare-feux, durcissement des systèmes, réponse aux alertes, gestion des correctifs.

Cette distinction est fondamentale : un déficit de gouvernance ne se corrige jamais uniquement par des moyens techniques supplémentaires. Une organisation peut disposer d'outils de sécurité performants (antivirus, pare-feu de nouvelle génération, SIEM) et rester néanmoins vulnérable si personne, au niveau stratégique, n'a défini de priorités claires, alloué de budget récurrent ou assigné de responsabilités précises. À l'inverse, une gouvernance solide sans compétence technique suffisante reste également insuffisante : la gouvernance encadre et oriente l'action technique, elle ne s'y substitue pas.

### 1.1 Gouvernance SSI et gouvernance d'entreprise

La gouvernance de la sécurité de l'information n'est pas une discipline isolée : elle s'inscrit dans la gouvernance globale de l'organisation, au même titre que la gouvernance financière ou la gouvernance des risques opérationnels. Cet ancrage a une conséquence pratique importante : le RSSI doit être capable de traduire des enjeux techniques (une vulnérabilité, un incident, une dette technique) en langage de gestion des risques compréhensible par des décideurs non spécialistes — impact financier potentiel, probabilité, comparaison avec d'autres risques de l'organisation (risque de change, risque juridique, risque de réputation). C'est ce travail de traduction qui légitime la place de la sécurité dans les instances de décision.

## 2. Pourquoi la sécurité doit être gouvernée et non seulement « techniquement gérée »

- La sécurité a un coût et engage des arbitrages (budget, ressources humaines, priorités) qui relèvent de décisions de direction, pas seulement d'équipes techniques.
- Un incident de sécurité majeur a des conséquences qui dépassent l'informatique (juridiques, financières, réputationnelles) : la direction doit être impliquée dans l'appétence au risque acceptée.
- Sans portage par la direction, les recommandations issues des audits (module précédent) restent rarement suivies d'effet.
- La sécurité entre en tension avec d'autres objectifs légitimes de l'organisation (rapidité de mise sur le marché, simplicité d'usage, coûts maîtrisés) : seule une instance de gouvernance disposant d'une vision transverse peut arbitrer ces tensions de façon cohérente, plutôt que de laisser chaque équipe technique trancher isolément selon ses propres priorités.
- La sécurité est par nature transversale : un même risque (par exemple la compromission d'un compte à privilèges) peut engager simultanément le DSI, les ressources humaines, le service juridique et la communication. Seule une gouvernance formalisée permet de coordonner ces acteurs avant qu'un incident ne survienne, plutôt que de les découvrir au moment de la crise.

### 2.1 Illustration pédagogique (cas fictif)

Considérons une organisation fictive, la Société Fictive de Services Financiers (SFSF), qui souhaite illustrer ce principe. Le responsable technique constate qu'un système hérité (legacy) n'est plus supporté par son éditeur et présente des vulnérabilités connues. Sans structure de gouvernance, ce constat reste un ticket technique parmi d'autres, noyé dans le flux quotidien, et le risque n'est jamais explicitement arbitré. Avec une gouvernance SSI structurée, ce constat est formalisé en risque (probabilité d'exploitation, impact potentiel sur la continuité des opérations), inscrit au registre des risques, présenté en comité de pilotage sécurité, puis arbitré par la direction qui décide, en toute connaissance de cause, d'un calendrier de migration et d'un budget associé. La différence n'est pas la détection du problème — un technicien compétent l'aurait identifié dans les deux cas — mais la capacité de l'organisation à transformer ce constat en décision documentée et suivie.

## 3. Les rôles clés

| Rôle | Responsabilité principale |
|---|---|
| **Direction générale** | définit l'appétence au risque, arbitre les budgets, porte la culture sécurité |
| **RSSI** (Responsable de la Sécurité des Systèmes d'Information) | pilote la stratégie SSI, anime la gouvernance, rapporte à la direction |
| **DSI** | met en œuvre les moyens techniques, souvent en tension avec le RSSI sur les priorités (disponibilité vs sécurité) |
| **Propriétaires de processus métier (business owners)** | identifient la valeur et la criticité des actifs qu'ils exploitent, arbitrent le traitement du risque sur leur périmètre |
| **Correspondants sécurité / DPO** | relais opérationnels, conformité réglementaire (protection des données) |

Un point de vigilance de gouvernance fréquent : le RSSI, pour rester indépendant, ne devrait idéalement pas être rattaché hiérarchiquement à la DSI qu'il est censé challenger. Un rattachement direct à la direction générale, à la direction des risques ou à la direction juridique est souvent préféré dans les organisations matures, précisément pour éviter qu'un même responsable (le DSI) n'arbitre à la fois la performance opérationnelle des systèmes (dont il est comptable) et le niveau d'exigence sécurité qui peut la contraindre (délais de mise en production, coûts de durcissement, disponibilité). Ce choix de rattachement n'est cependant pas universel : dans de plus petites structures, où les ressources ne permettent pas de multiplier les lignes hiérarchiques, un compromis pragmatique consiste à garantir au RSSI un accès direct et régulier à la direction générale, même s'il reste administrativement rattaché à la DSI.

### 3.1 Le RSSI : un rôle de traduction et d'influence plus que d'exécution directe

Contrairement à une idée reçue, le RSSI ne « fait » généralement pas la sécurité au sens de la configuration technique des systèmes — cette responsabilité incombe aux équipes DSI et aux administrateurs. Son rôle est davantage celui d'un animateur transversal : il définit la stratégie, anime la gouvernance des risques, propose des arbitrages à la direction, sensibilise les métiers et vérifie, via des indicateurs et des audits, que les mesures décidées sont effectivement mises en œuvre. Cette dimension d'influence sans autorité hiérarchique directe sur les équipes techniques explique pourquoi le soutien explicite de la direction générale est une condition de succès quasi systématique du poste.

## 4. Les composantes d'un système de gouvernance SSI

1. **Politique de sécurité** (PSSI) : document de référence formalisant les orientations, principes et responsabilités.
2. **Organisation** : comités de pilotage, instances de décision, circuits d'escalade.
3. **Gestion des risques** : méthodologie d'identification, d'évaluation et de traitement des risques (chapitre 2).
4. **Conformité** : suivi des obligations légales, réglementaires et contractuelles.
5. **Pilotage et amélioration continue** : indicateurs, tableaux de bord, audits internes, revues de direction.

Ces cinq composantes ne fonctionnent pas de manière isolée : elles forment un système cohérent où chaque élément nourrit les autres. La politique de sécurité fixe le cadre dans lequel la gestion des risques opère ; les résultats de la gestion des risques alimentent le plan de traitement et, par extension, les priorités budgétaires ; le suivi de conformité fournit une partie des exigences que la gestion des risques doit couvrir ; et le pilotage par indicateurs permet de vérifier, dans la durée, que l'ensemble du dispositif produit les effets attendus. Un système de gouvernance qui ne comporterait qu'une politique bien rédigée, sans mécanisme de suivi ni de mesure de son application effective, resterait un exercice largement théorique.

### 4.1 Les instances de gouvernance en pratique

Au-delà des documents, la gouvernance SSI se matérialise concrètement par des instances qui se réunissent à intervalles réguliers, avec un ordre du jour et des comptes rendus formalisés. On distingue typiquement :

| Instance | Fréquence usuelle | Composition | Objet |
|---|---|---|---|
| **Comité de sécurité opérationnel** | mensuelle | RSSI, correspondants sécurité, équipes techniques | suivi des incidents, des vulnérabilités, avancement des actions correctives |
| **Comité de pilotage SSI** | trimestrielle | RSSI, DSI, représentants métiers | arbitrages sur le plan de traitement des risques, suivi des indicateurs |
| **Revue de direction** | annuelle (a minima) | direction générale, RSSI, DSI | validation de la stratégie, de l'appétence au risque, allocation budgétaire (voir chapitre 4) |

Cette cascade d'instances permet de faire remonter l'information pertinente à chaque niveau sans noyer la direction générale dans des détails opérationnels, tout en garantissant que les décisions prises au sommet redescendent effectivement vers les équipes chargées de les exécuter.

## 5. Cadres de référence en gouvernance SSI

- **ISO/IEC 27001** : SMSI (système de management de la sécurité de l'information), avec un accent fort sur le pilotage par la direction (clause 5) et une exigence explicite d'amélioration continue.
- **COBIT** (Control Objectives for Information and Related Technologies) : cadre de gouvernance des systèmes d'information au sens large, incluant la sécurité, qui distingue explicitement gouvernance (orientation, supervision) et management (planification, réalisation, suivi) — une distinction très proche de celle introduite en section 1.
- **NIST Cybersecurity Framework** : structure la gouvernance autour de fonctions couvrant l'identification des risques, la protection, la détection, la réponse et la reprise après incident, largement adoptée au-delà du contexte américain pour sa simplicité de communication auprès de publics non techniques.

Ces cadres ne sont pas concurrents mais complémentaires : une organisation peut, par exemple, structurer son SMSI selon ISO 27001 pour viser une certification, tout en utilisant le vocabulaire du NIST Cybersecurity Framework pour communiquer plus simplement avec sa direction générale sur son niveau de maturité global. Le choix dépend souvent du secteur d'activité, du contexte réglementaire et des exigences des partenaires ou clients (certains appels d'offres exigent explicitement une certification ISO 27001, par exemple).

## 6. Maturité et amélioration continue

La gouvernance SSI n'est pas un état figé : elle progresse par cycles (souvent modélisés en PDCA — Plan, Do, Check, Act). Une organisation immature réagit aux incidents au coup par coup ; une organisation mature anticipe par une gestion structurée des risques et ajuste ses priorités sur la base d'indicateurs et de retours d'expérience.

### 6.1 Modèles de maturité

Pour objectiver ce niveau de maturité plutôt que de rester dans une appréciation subjective, de nombreuses organisations utilisent des grilles d'évaluation à plusieurs niveaux, inspirées des modèles de maturité de type CMM (Capability Maturity Model), adaptés à la sécurité de l'information. Un exemple pédagogique simplifié à cinq niveaux :

| Niveau | Intitulé | Caractéristique typique |
|---|---|---|
| 1 | Initial / ad hoc | pas de processus formalisé, actions réactives, dépendance de quelques individus clés |
| 2 | Reproductible | premières pratiques documentées, mais appliquées de façon inégale selon les équipes |
| 3 | Défini | processus formalisés et communiqués à l'échelle de l'organisation (PSSI, gestion des risques) |
| 4 | Piloté | processus mesurés par des indicateurs, décisions fondées sur des données |
| 5 | Optimisé | amélioration continue institutionnalisée, anticipation proactive des risques émergents |

Ce type de grille sert avant tout d'outil de dialogue avec la direction : elle permet de situer objectivement l'organisation, de fixer une cible réaliste (viser le niveau 5 partout n'est ni possible ni toujours pertinent) et de prioriser les investissements sur les domaines où l'écart de maturité est le plus risqué au regard des enjeux métier.

### 6.2 Le cycle PDCA appliqué à la gouvernance SSI

Le cycle PDCA, popularisé en gestion de la qualité puis repris par l'ISO 27001, structure l'amélioration continue en quatre phases répétées :

- **Plan** : établir le contexte, identifier les risques, définir les objectifs et le plan de traitement.
- **Do** : mettre en œuvre les mesures décidées (déploiement technique, formation, procédures).
- **Check** : vérifier l'efficacité des mesures via audits, indicateurs, tests.
- **Act** : ajuster la stratégie, corriger les écarts constatés, et relancer un nouveau cycle.

Ce cycle n'est pas un événement ponctuel mais un rythme continu de l'organisation : c'est précisément cette récurrence qui distingue une gouvernance mature d'une démarche de conformité ponctuelle réalisée uniquement à l'approche d'un audit ou d'une certification.

## À retenir

- La gouvernance SSI relève de la direction, pas uniquement des équipes techniques : elle arbitre l'appétence au risque et les moyens alloués, et articule trois niveaux (stratégique, tactique, opérationnel).
- Un RSSI efficace a besoin d'un positionnement organisationnel qui préserve son indépendance vis-à-vis de la DSI, et joue avant tout un rôle de traduction et d'animation transversale plutôt que d'exécution technique directe.
- La gouvernance s'appuie sur un cycle continu : politique, gestion des risques, conformité, pilotage, amélioration — matérialisé par des instances régulières (comité opérationnel, comité de pilotage, revue de direction).
- Des cadres de référence reconnus (ISO/IEC 27001, COBIT, NIST Cybersecurity Framework) et des modèles de maturité permettent de structurer et d'objectiver la démarche plutôt que de la laisser dépendre d'appréciations subjectives.
- La maturité d'une gouvernance SSI se mesure et progresse par cycles (PDCA) ; elle n'est jamais définitivement acquise.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
