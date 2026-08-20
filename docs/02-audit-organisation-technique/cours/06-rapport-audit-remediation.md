# Chapitre 6 — Rapport d'audit et plan de remédiation

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/06-rapport-audit-remediation.txt){ target=_blank }

## 1. Objectifs du rapport

Le rapport est le principal livrable d'un audit : c'est lui qui sera lu, discuté et utilisé pour arbitrer des décisions et des budgets. Un audit techniquement excellent mais mal restitué perd une grande partie de sa valeur.

Cette affirmation, qui peut surprendre des étudiants davantage attirés par la dimension technique de l'audit, mérite d'être développée : dans la grande majorité des organisations, les personnes qui décident d'allouer un budget de remédiation, de recruter un poste supplémentaire en sécurité ou de revoir une architecture ne liront jamais le détail des commandes exécutées ni les preuves de concept techniques. Elles liront la synthèse, regarderont éventuellement un tableau de priorités, et prendront leur décision sur cette base. Un auditeur qui néglige la qualité rédactionnelle de son rapport — même après un travail technique irréprochable — prend le risque que ses constats les plus critiques restent sans suite faute d'avoir été compris et priorisés par les bonnes personnes.

Le rapport remplit en réalité plusieurs fonctions simultanées, qu'il faut garder à l'esprit lors de sa rédaction :

- une fonction **décisionnelle** (aider la direction à arbitrer des priorités et des budgets) ;
- une fonction **opérationnelle** (permettre aux équipes techniques de reproduire, comprendre et corriger chaque constat) ;
- une fonction **de preuve** (démontrer, pour un audit de certification ou une obligation contractuelle, qu'une évaluation a bien été réalisée selon une méthodologie donnée) ;
- parfois une fonction **juridique** (élément de preuve de diligence raisonnable en cas de contentieux ultérieur, notamment après un incident de sécurité).

Ces fonctions multiples expliquent pourquoi un rapport d'audit sérieux se structure toujours en plusieurs niveaux de lecture, du plus synthétique au plus détaillé, plutôt qu'en un document uniforme.

## 2. Structure type d'un rapport d'audit

1. **Synthèse à destination de la direction (executive summary)** : 1 à 2 pages, sans jargon technique, avec le niveau de risque global et les priorités.
2. **Contexte et périmètre** : rappel du mandat, dates, méthode, limites de l'audit.
3. **Méthodologie** : référentiel utilisé, mode d'accès (black/grey/white box), outils.
4. **Constats détaillés**, un par vulnérabilité ou non-conformité, avec pour chacun :
   - titre et catégorie (ex. « Injection SQL sur le formulaire de connexion ») ;
   - criticité (score CVSS ou échelle qualitative Critique/Élevé/Moyen/Faible) ;
   - description technique et preuve de concept (capture, extrait de log, commande) ;
   - impact métier ;
   - recommandation de remédiation, si possible priorisée et estimée en effort.
5. **Synthèse des recommandations** sous forme de tableau priorisé.
6. **Annexes** : détails techniques, sorties d'outils, méthodologie complète.

Chacune de ces sections répond à un besoin de lecture différent, ce qui justifie qu'elles soient nettement séparées plutôt que mélangées.

**La synthèse à destination de la direction** (section 1) est probablement la partie la plus difficile à rédiger, précisément parce qu'elle doit être courte et dépourvue de jargon technique tout en restant fidèle à la réalité des constats. Une bonne synthèse répond en une à deux pages à trois questions simples : où en est l'organisation en matière de sécurité par rapport au référentiel ou à l'objectif visé ? Quels sont les deux ou trois risques les plus importants à traiter en priorité ? Que faut-il faire, à quel horizon, pour les traiter ? Elle évite absolument le jargon technique non expliqué (un terme comme « SSRF » ou « Kerberoasting » n'a aucune place dans cette section sans reformulation en langage courant) et privilégie des formulations orientées impact métier plutôt que mécanisme technique.

**Le contexte et le périmètre** (section 2) doivent explicitement mentionner les **limites** de l'audit : ce qui n'a pas été testé (par exclusion contractuelle, par manque de temps, ou parce que hors périmètre), les conditions dans lesquelles les tests ont été réalisés (environnement de test ou de production, disponibilité limitée d'un compte de test) et toute contrainte ayant pu affecter la couverture réelle de l'audit. Cette transparence protège à la fois le commanditaire (qui sait précisément ce qui a été couvert) et l'auditeur (qui ne peut être tenu responsable de vulnérabilités hors du périmètre convenu).

**La méthodologie** (section 3) doit être suffisamment précise pour qu'un tiers puisse comprendre comment les constats ont été obtenus, sans nécessairement entrer dans le détail exhaustif de chaque commande (réservé aux annexes).

**Les constats détaillés** (section 4) constituent le cœur technique du rapport. Chaque constat doit être **autoportant** : un lecteur qui ne consulterait que ce constat isolément (par exemple, extrait et transmis à un prestataire externe pour correction) doit pouvoir comprendre le problème, sa gravité et la marche à suivre sans avoir à lire l'intégralité du rapport.

**La synthèse des recommandations** (section 5) reprend, sous forme de tableau compact, l'ensemble des constats avec leur priorité, ce qui permet un pilotage rapide sans revenir au détail de chaque constat — c'est souvent cette section qui sert directement de base au plan de remédiation traité en section 4 de ce chapitre.

**Les annexes** (section 6) accueillent tout ce qui alourdirait la lecture du corps du rapport sans en changer la compréhension : sorties brutes d'outils (scans complets), captures d'écran supplémentaires, glossaire technique.

### Exemple de constat détaillé rédigé intégralement

Pour illustrer concrètement la structure attendue d'un constat (section 4), voici un exemple pédagogique complet, fictif, construit sur un scénario classique :

> **C-03 — Absence d'authentification multifacteur sur l'accès VPN administrateur**
> **Catégorie** : Défaillances d'identification et d'authentification (réf. OWASP A07)
> **Criticité** : Élevée (CVSS 3.1 base score 8.1 — AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N)
> **Description** : l'accès VPN utilisé par les administrateurs pour l'administration à distance de l'infrastructure ne requiert qu'un identifiant et un mot de passe, sans second facteur d'authentification. Un test réalisé le [date] avec un compte de test fourni a permis de confirmer l'absence de toute sollicitation de second facteur lors de la connexion.
> **Impact métier** : en cas de compromission d'un mot de passe administrateur (hameçonnage, réutilisation d'un mot de passe divulgué lors d'une fuite tierce), un attaquant externe pourrait obtenir un accès direct au réseau d'administration, avec un potentiel d'impact majeur sur la confidentialité et l'intégrité de l'ensemble du système d'information.
> **Recommandation** : déployer une authentification multifacteur (application d'authentification ou clé matérielle) obligatoire pour tous les comptes disposant d'un accès VPN à privilèges administratifs, en priorité absolue avant tout autre chantier de remédiation identifié dans ce rapport.
> **Effort estimé** : faible à moyen (dépend de la solution MFA déjà disponible dans l'environnement d'identité existant).

## 3. Qualifier la criticité

Une bonne pratique consiste à combiner **vraisemblance** (facilité d'exploitation, exposition) et **impact** (confidentialité, intégrité, disponibilité, conformité réglementaire) dans une matrice de risque, plutôt que d'utiliser un score technique brut (CVSS) sans contextualisation métier — une vulnérabilité CVSS 9.8 sur un serveur de test isolé n'a pas le même risque réel qu'une vulnérabilité CVSS 6.0 sur le système de paiement en production.

Cette combinaison peut être formalisée dans une **matrice de risque** à deux dimensions, outil visuel simple et largement compris, y compris par un public non technique :

| Impact \ Vraisemblance | Faible | Moyenne | Élevée |
|---|---|---|---|
| **Faible** | Faible | Faible | Moyen |
| **Moyen** | Faible | Moyen | Élevé |
| **Élevé** | Moyen | Élevé | Critique |

L'utilisation d'une telle matrice suppose que l'auditeur documente explicitement, pour chaque constat, les facteurs qui ont motivé le positionnement retenu sur chaque axe :

- pour la **vraisemblance** : la vulnérabilité est-elle exploitable sans authentification préalable ? Nécessite-t-elle des compétences techniques avancées ou un outil public largement disponible ? Le système concerné est-il directement exposé sur Internet ou uniquement accessible depuis un réseau interne déjà restreint ?
- pour l'**impact** : quelles données ou fonctionnalités sont concernées (données personnelles, données financières, fonctionnalité non critique) ? Le système touché est-il en production ou dans un environnement de moindre sensibilité ? Existe-t-il une obligation réglementaire spécifique renforçant l'impact (notification obligatoire en cas de fuite de données personnelles, par exemple) ?

### Piège courant : la priorisation uniquement par score CVSS brut

Prioriser un plan de remédiation en triant mécaniquement les constats par score CVSS décroissant, sans tenir compte du contexte métier, est une erreur fréquente chez les auditeurs juniors et chez certains outils automatisés qui ne proposent que cette vue par défaut. Le score CVSS de base ne prend pas en compte l'exposition réelle (un service accessible uniquement depuis un réseau d'administration fortement restreint n'a pas le même niveau de risque réel qu'un service identique exposé directement sur Internet), ni la sensibilité des données concernées, ni les mesures compensatoires éventuellement déjà en place (segmentation, surveillance renforcée). Un rapport de qualité explique toujours, pour chaque constat, comment le score technique a été ajusté — à la hausse ou à la baisse — par le contexte métier.

## 4. Plan de remédiation

Le plan de remédiation transforme chaque recommandation en action suivie :

| Constat | Priorité | Action | Responsable | Échéance | Statut |
|---|---|---|---|---|---|
| Mot de passe admin par défaut | Critique | Changer et appliquer une politique de mots de passe forts | Équipe infra | J+7 | À faire |
| Absence de MFA sur le VPN | Élevé | Déployer une authentification à double facteur | RSSI | J+30 | En cours |
| Composant obsolète (CVE connue) | Élevé | Mettre à jour la bibliothèque concernée | Équipe dev | J+15 | À faire |

Un plan de remédiation efficace repose sur quelques principes de gestion de projet appliqués au contexte de la sécurité :

- **Un seul responsable identifié par action** : une action assignée à une « équipe » plutôt qu'à une personne nommée a statistiquement moins de chances d'être menée à terme dans les délais, faute d'appropriation individuelle claire.
- **Des échéances réalistes et différenciées selon la criticité** : une vulnérabilité critique directement exploitable depuis Internet appelle une correction en urgence (heures à quelques jours), tandis qu'une non-conformité organisationnelle de moindre impact peut légitimement s'inscrire dans un cycle de plusieurs mois — le plan doit refléter cette différenciation plutôt que d'imposer un rythme uniforme.
- **Des mesures compensatoires en cas de correction différée** : lorsqu'une correction définitive ne peut être appliquée immédiatement (dépendance à un fournisseur, contrainte de compatibilité), le plan doit prévoir des mesures temporaires réduisant le risque en attendant (restriction d'accès réseau, surveillance renforcée, désactivation temporaire d'une fonctionnalité non essentielle).
- **Un suivi régulier et visible** : un plan de remédiation qui n'est revu qu'à l'occasion de l'audit suivant perd une grande partie de son utilité ; un point de suivi périodique (hebdomadaire pour les actions critiques, mensuel pour les autres) permet de détecter rapidement les retards et d'en comprendre les causes (manque de ressources, dépendance technique bloquante, changement de priorité).

### Bonne pratique : documenter les risques acceptés

Il arrive qu'une organisation décide, en toute connaissance de cause, de ne pas corriger un constat donné (coût disproportionné par rapport au risque résiduel, incompatibilité avec une contrainte métier, système en fin de vie déjà planifiée). Ce choix est légitime dans le cadre d'une démarche de gestion du risque mature, à condition d'être **formalisé** : décision documentée, validée par une personne habilitée (généralement le RSSI ou la direction), avec une date de réexamen. Un auditeur qui constate, lors d'un contre-audit, qu'un constat critique n'a pas été corrigé sans qu'aucune décision d'acceptation du risque n'ait été formalisée doit le signaler comme une non-conformité en tant que telle, distincte du constat technique initial.

## 5. Contre-audit et suivi

Un audit de suivi (souvent partiel, ciblé sur les constats précédents) permet de vérifier l'application effective des corrections. C'est une pratique attendue dans les cycles de certification (audits de surveillance ISO 27001) et une preuve de maturité pour l'organisation auditée.

Un contre-audit efficace ne se limite pas à revérifier mécaniquement chaque constat initial : il doit également vérifier que la correction appliquée n'a pas introduit de nouveau problème (une mise à jour de composant, par exemple, peut corriger une vulnérabilité tout en modifiant un comportement fonctionnel de façon non anticipée) et que la correction est **durable**, pas seulement appliquée ponctuellement pour l'occasion du contre-audit (un piège classique consiste, pour l'organisation auditée, à corriger temporairement un constat juste avant le contre-audit sans pérenniser la mesure — un contrôle un peu plus tard dans le temps permet de détecter ce type de régression).

Le rythme des contre-audits dépend du contexte : dans un cycle de certification ISO 27001, des audits de surveillance sont généralement réalisés annuellement entre deux audits de certification complets (eux-mêmes réalisés à intervalle pluriannuel selon le schéma de certification retenu) ; en dehors de tout cadre de certification, une organisation mature planifie généralement un contre-audit ciblé sur les constats critiques dans les mois suivant leur identification, puis un audit complet à intervalle régulier (annuel étant une fréquence courante, à adapter selon le niveau de risque et le rythme d'évolution du système d'information).

## 6. Communication et postures

- Adapter le niveau de détail au public (direction vs équipes techniques) — d'où l'intérêt de séparer synthèse et annexes techniques.
- Rester factuel et non accusatoire : l'audit évalue un système, pas les personnes.
- Respecter la confidentialité du rapport, qui contient potentiellement des informations exploitables par un attaquant (diffusion strictement limitée aux parties autorisées, destruction des données brutes après la période contractuelle convenue).

Ces trois principes méritent d'être développés, car la posture de communication de l'auditeur conditionne fortement la façon dont ses constats seront reçus et suivis d'effet.

**Adapter le niveau de détail au public** ne signifie pas simplement raccourcir le texte : cela implique de reformuler activement les constats techniques en termes de risque métier compréhensible. Dire à un directeur général qu'une application « présente une vulnérabilité SQLi CVSS 9.8 » a beaucoup moins d'impact décisionnel que dire qu'« un attaquant externe pourrait, sans authentification, consulter ou modifier l'intégralité de la base de données clients, avec un risque de sanction réglementaire et d'atteinte à la réputation en cas de fuite ».

**Rester factuel et non accusatoire** est un principe déontologique central de l'audit. Un rapport rédigé sur un ton accusatoire ou moralisateur (« l'équipe n'a manifestement jamais pris la sécurité au sérieux ») produit généralement l'effet inverse de celui recherché : il braque les équipes concernées, nuit à la qualité de la collaboration lors des audits suivants, et détourne l'attention du fond (les constats et leur correction) vers la forme (le ressenti d'être mis en cause personnellement). La formulation recommandée reste toujours centrée sur le système et le processus, jamais sur les personnes : « le processus de gestion des correctifs ne couvre pas l'ensemble du parc serveur » plutôt que « l'administrateur système n'a pas appliqué les correctifs ».

**Respecter la confidentialité du rapport** est une obligation à la fois contractuelle et éthique évidente, mais dont les modalités pratiques méritent d'être anticipées dès le cadrage de la mission (chapitre 1) : qui a le droit de recevoir le rapport complet, avec ses preuves de concept techniques exploitables, et qui ne devrait recevoir que la synthèse ? Sous quelle forme et pendant combien de temps les données brutes de l'audit (captures, exports de scan, journaux collectés) sont-elles conservées, et selon quelles modalités sont-elles détruites à l'issue de la période contractuelle convenue ? Ces questions, souvent traitées trop tardivement, devraient idéalement être actées dès le mandat initial plutôt qu'au moment de la remise du rapport.

### Approfondissement : la restitution orale, un exercice à part entière

Au-delà du document écrit, la restitution orale du rapport (réunion de présentation à la direction et aux équipes techniques) est un moment clé souvent sous-préparé. Une bonne restitution anticipe les questions et objections probables, prépare une hiérarchisation claire des trois à cinq messages essentiels à faire passer (plutôt que de dérouler mécaniquement l'ensemble des constats), et laisse une place réelle au dialogue plutôt qu'à une simple lecture du document. C'est souvent lors de cette restitution que se joue l'adhésion réelle de l'organisation au plan de remédiation proposé.

## À retenir

- Un rapport d'audit efficace sépare clairement synthèse décisionnelle et détails techniques, et s'adresse à des publics aux besoins de lecture différents (direction, équipes techniques, auditeurs de certification).
- Chaque constat doit être actionnable et autoportant : criticité contextualisée, preuve, impact métier, recommandation et, idéalement, effort estimé.
- La criticité doit toujours combiner vraisemblance et impact métier dans une matrice de risque, jamais se limiter à un score technique brut trié mécaniquement.
- Le plan de remédiation transforme chaque recommandation en action suivie, avec un responsable nommé, une échéance réaliste et différenciée, et éventuellement des mesures compensatoires ; les risques non corrigés doivent être formellement acceptés plutôt que simplement ignorés.
- Le contre-audit et le suivi transforment l'audit en amélioration continue plutôt qu'en photographie isolée ; il doit vérifier non seulement la correction mais aussi sa pérennité.
- La posture de communication — adaptation au public, ton factuel et non accusatoire, confidentialité rigoureuse — conditionne autant l'impact réel de l'audit que la qualité technique des constats eux-mêmes.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
