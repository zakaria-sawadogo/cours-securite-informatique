# Chapitre 4 — Pilotage, indicateurs et conformité réglementaire

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=04-gouvernance-gestion-risques-si/slides/04-pilotage-indicateurs-conformite.txt){ target=_blank }

## 1. Pourquoi piloter par les indicateurs

Une gouvernance SSI mature ne se limite pas à produire des documents (PSSI, analyses de risque) : elle démontre, dans la durée, une amélioration mesurable. Les indicateurs permettent de communiquer objectivement avec la direction, de détecter une dérive avant qu'elle ne devienne un incident, et de justifier les investissements.

Ce chapitre clôt logiquement le module : les chapitres précédents ont posé le cadre de gouvernance (chapitre 1), la méthodologie d'analyse des risques (chapitre 2) et les modalités de traitement (chapitre 3). Le pilotage par indicateurs est ce qui referme le cycle PDCA introduit au chapitre 1 : il permet de vérifier, avec des données plutôt que des impressions, que les décisions prises produisent réellement les effets attendus, et d'alimenter le cycle suivant sur des bases objectives plutôt que sur une simple reconduction des pratiques existantes.

### 1.1 Les fonctions d'un indicateur, au-delà du simple constat

Un indicateur SSI bien conçu remplit simultanément plusieurs fonctions, qu'il est utile de distinguer explicitement pour éviter de produire des indicateurs purement décoratifs :

- **Une fonction de suivi** : mesurer l'évolution d'un phénomène dans le temps (une tendance est presque toujours plus informative qu'une valeur isolée à un instant donné).
- **Une fonction d'alerte** : signaler un dépassement de seuil qui appelle une action corrective, idéalement avant qu'un incident ne survienne plutôt qu'après coup.
- **Une fonction de justification** : documenter, à l'attention de la direction ou d'un auditeur externe, que les moyens alloués produisent un effet mesurable, ce qui légitime les demandes de budget futures.
- **Une fonction de comparaison** : permettre de se situer par rapport à un objectif interne, à une valeur cible fixée en gouvernance, ou éventuellement à d'autres entités comparables lorsque des données de référence existent.

Un même indicateur ne remplit pas nécessairement toutes ces fonctions à la fois ; il est donc utile, lors de la conception d'un tableau de bord, de clarifier explicitement à quelle fonction chaque indicateur retenu répond.

## 2. Caractéristiques d'un bon indicateur

Un indicateur utile est généralement décrit par les critères **SMART** : Spécifique, Mesurable, Atteignable, Réaliste, Temporellement défini. Il doit surtout être **actionnable** : un indicateur qui ne débouche sur aucune décision possible n'a pas d'intérêt opérationnel.

### 2.1 Pièges fréquents dans la conception d'indicateurs

L'expérience montre que certains écueils reviennent fréquemment dans la construction de tableaux de bord SSI, et qu'il est utile de connaître pour les éviter :

- **La multiplication d'indicateurs peu signifiants** : un tableau de bord comportant plusieurs dizaines d'indicateurs devient illisible et dilue l'attention de la direction sur ce qui compte réellement ; mieux vaut un nombre restreint d'indicateurs réellement pilotés qu'une liste exhaustive jamais analysée en profondeur.
- **La confusion entre indicateur d'activité et indicateur de résultat** : compter le nombre de formations de sensibilisation dispensées (indicateur d'activité) est différent de mesurer le taux de clic sur une campagne de simulation de phishing (indicateur de résultat, qui reflète mieux l'effet réel de la sensibilisation).
- **L'absence de cible associée** : un indicateur affiché sans valeur de référence ni seuil d'alerte ne permet pas de savoir si la situation constatée est satisfaisante ou préoccupante.
- **Le choix d'indicateurs valorisants plutôt qu'informatifs** : la tentation de ne présenter que des indicateurs favorables à l'équipe sécurité (en omettant ceux qui révéleraient des difficultés) nuit à la crédibilité du dispositif de pilotage sur le long terme et peut retarder la détection de problèmes réels.

## 3. Exemples d'indicateurs SSI

| Catégorie | Exemple d'indicateur |
|---|---|
| Vulnérabilités | délai moyen d'application des correctifs critiques (patch management) |
| Incidents | nombre d'incidents de sécurité déclarés par mois, temps moyen de détection (MTTD), temps moyen de résolution (MTTR) |
| Sensibilisation | taux de clics sur des campagnes de simulation de phishing |
| Conformité | pourcentage d'exigences ISO 27001 couvertes, nombre de non-conformités ouvertes |
| Continuité | résultat des derniers tests de PCA/PRA (RTO/RPO atteints ou non) |
| Accès | nombre de comptes à privilèges, délai de révocation des accès après départ d'un collaborateur |

### 3.1 Exemple illustratif de fiche d'indicateur (cas pédagogique)

Pour qu'un indicateur soit véritablement exploitable, il est recommandé de le documenter au-delà de sa simple valeur chiffrée, dans une fiche qui en précise le calcul et l'usage attendu. Un exemple pédagogique fictif :

> **Fiche indicateur — Délai moyen d'application des correctifs critiques**
>
> - *Définition* : nombre moyen de jours écoulés entre la publication d'un correctif de sécurité classé critique par l'éditeur et son déploiement effectif sur les systèmes concernés.
> - *Source de données* : outil de gestion des correctifs (patch management) et inventaire des actifs.
> - *Fréquence de calcul* : mensuelle.
> - *Cible fixée par la gouvernance* : délai inférieur à un seuil défini en comité de pilotage, cohérent avec l'appétence au risque de l'organisation.
> - *Seuil d'alerte* : dépassement du seuil sur deux mois consécutifs déclenche une analyse de cause et un plan d'action présenté en comité de pilotage.
> - *Limite connue* : l'indicateur ne distingue pas, dans sa version simple, les systèmes critiques des systèmes secondaires ; une déclinaison par criticité d'actif est recommandée pour affiner le pilotage.

Documenter ainsi chaque indicateur clé évite les désaccords récurrents sur sa méthode de calcul et facilite sa reprise par un successeur du RSSI, un enjeu de continuité souvent sous-estimé.

### 3.2 Exemple d'évolution d'indicateurs sur plusieurs périodes

Au-delà de la valeur instantanée, la tendance dans le temps est souvent l'information la plus utile pour la direction. Un exemple pédagogique simplifié de suivi trimestriel :

| Indicateur | T1 | T2 | T3 | T4 | Cible | Tendance |
|---|---|---|---|---|---|---|
| Délai moyen de correction des vulnérabilités critiques (jours) | 21 | 17 | 14 | 12 | ≤ 15 | en amélioration |
| Nombre d'incidents de sécurité déclarés | 6 | 4 | 5 | 3 | — | stable à en amélioration |
| Taux de clic sur simulation de phishing | 22 % | 18 % | 15 % | 13 % | ≤ 10 % | en amélioration, cible non encore atteinte |
| Non-conformités ouvertes (audit interne) | 9 | 11 | 7 | 5 | 0 | en amélioration |

Ce type de tableau, même construit sur des données pédagogiques fictives, illustre l'intérêt du suivi dans la durée : il permet de distinguer une variation ponctuelle sans conséquence d'une tendance structurelle qui appelle une action de fond, et de démontrer concrètement à la direction l'effet des mesures prises (par exemple, une baisse continue du taux de clic sur les campagnes de simulation de phishing peut être mise en regard d'un programme de sensibilisation renforcé).

## 4. Tableau de bord SSI

Un tableau de bord synthétise ces indicateurs pour différents publics :

- **Vision direction** : quelques indicateurs agrégés, tendance (amélioration/dégradation), niveau de risque global.
- **Vision RSSI/opérationnelle** : indicateurs détaillés par domaine, permettant d'identifier les points d'action prioritaires.

La fréquence de mise à jour (mensuelle, trimestrielle) et le format (comité de pilotage, revue de direction) doivent être définis dans la gouvernance elle-même.

### 4.1 Adapter la forme du tableau de bord à son public

Un même ensemble de données sous-jacentes peut être présenté de façon très différente selon le public visé, et cette adaptation est elle-même un exercice de gouvernance à part entière :

- Pour la **direction générale**, la forme privilégie la synthèse visuelle (codes couleur, tendances, un nombre restreint d'indicateurs clés) et la mise en perspective avec les enjeux métier (impact potentiel sur la continuité des opérations, coûts évités, position par rapport aux obligations réglementaires), plutôt que le détail technique.
- Pour le **comité de pilotage SSI**, le tableau de bord peut être plus détaillé, décliné par domaine (vulnérabilités, incidents, conformité, continuité), avec des indicateurs permettant d'identifier précisément où porter l'effort.
- Pour les **équipes opérationnelles**, un niveau de détail encore plus fin est pertinent (par exemple, la liste nominative des systèmes non corrigés), qui n'aurait pas de sens présenté tel quel en revue de direction.

Cette hiérarchisation de l'information, du détail opérationnel vers la synthèse stratégique, reprend directement la distinction des trois niveaux de gouvernance (stratégique, tactique, opérationnel) introduite au chapitre 1 : chaque niveau doit recevoir une information calibrée à son rôle dans la prise de décision, ni trop pauvre pour éclairer utilement, ni trop riche pour rester exploitable dans le temps disponible.

## 5. Conformité réglementaire : panorama non exhaustif

| Cadre réglementaire | Portée | Point clé |
|---|---|---|
| **RGPD** (et lois nationales équivalentes de protection des données) | données à caractère personnel | notification des violations, analyse d'impact (AIPD) pour les traitements à risque |
| **Réglementations sectorielles** (banque, assurance, santé, télécoms) | secteurs régulés | exigences souvent renforcées (ex. résilience opérationnelle dans le secteur financier) |
| **Réglementations nationales de cybersécurité** (souvent inspirées de directives régionales) | opérateurs d'importance vitale ou essentiels | obligations de déclaration d'incident, exigences techniques minimales |
| **Exigences contractuelles** | relations clients/fournisseurs | souvent alignées sur ISO 27001 ou des questionnaires de sécurité spécifiques |

La conformité n'est pas un objectif en soi mais un sous-ensemble de la gestion des risques : une organisation conforme n'est pas nécessairement sécurisée, et inversement une organisation bien sécurisée doit néanmoins documenter sa conformité pour ses obligations légales et contractuelles.

### 5.1 Le risque du « théâtre de conformité »

Un écueil bien documenté dans la littérature professionnelle mérite d'être souligné : le « théâtre de conformité » (compliance theater), c'est-à-dire une situation où l'organisation consacre l'essentiel de son énergie à produire de la documentation et à cocher des cases lors d'audits, sans que cela se traduise par une amélioration réelle du niveau de sécurité effectif. Ce risque est particulièrement présent lorsque la conformité est pilotée en silo, déconnectée du processus de gestion des risques : une checklist de conformité peut être remplie de bonne foi tout en laissant de côté des risques réels non couverts par le référentiel appliqué. La bonne pratique consiste à faire de la conformité un **sous-produit** d'une gestion des risques bien menée (les mesures issues du traitement du risque couvrent naturellement une large part des exigences de conformité), plutôt que l'inverse (une conformité poursuivie pour elle-même, indépendamment d'une analyse réelle des risques encourus).

### 5.2 Exemple de suivi de conformité à un référentiel (cas pédagogique)

Le suivi de conformité à un référentiel comme l'ISO 27001 peut se matérialiser par un tableau de suivi des exigences, décliné par domaine de contrôle. Un extrait pédagogique simplifié :

| Domaine de contrôle | Exigences applicables | Exigences couvertes | Taux de couverture | Non-conformités ouvertes |
|---|---|---|---|---|
| Politique de sécurité | 4 | 4 | 100 % | 0 |
| Gestion des accès | 12 | 9 | 75 % | 2 |
| Gestion des incidents | 8 | 8 | 100 % | 0 |
| Continuité d'activité | 6 | 3 | 50 % | 3 |
| Gestion des fournisseurs | 7 | 5 | 71 % | 1 |

Ce type de tableau permet d'identifier immédiatement les domaines nécessitant un effort prioritaire (ici, la continuité d'activité, en cohérence avec le chapitre 3) et de nourrir directement le plan d'action présenté en revue de direction, plutôt que de se limiter à une appréciation globale et peu actionnable du type « l'organisation est conforme à 79 % ».

## 6. Revue de direction et amélioration continue

L'ISO 27001 impose une revue de direction périodique du SMSI, qui doit examiner : les résultats des audits internes/externes, l'état d'avancement du traitement des risques, les indicateurs de performance, les retours des parties intéressées, et déboucher sur des décisions (allocation de ressources, ajustement des priorités). C'est le point de bouclage du cycle PDCA introduit au chapitre 1.

### 6.1 Ordre du jour type d'une revue de direction (exemple pédagogique)

Pour rendre concrète cette instance centrale de la gouvernance, voici un exemple pédagogique de trame d'ordre du jour, tel qu'on pourrait le retrouver dans une organisation appliquant l'ISO 27001 :

1. Suivi des décisions de la revue de direction précédente (actions réalisées, en cours, en retard).
2. Évolution du contexte interne et externe susceptible d'affecter le SMSI (nouvelle réglementation, changement organisationnel, nouvelle menace identifiée).
3. Résultats des audits internes et externes réalisés depuis la dernière revue.
4. Retours des parties intéressées pertinentes (clients, fournisseurs, régulateurs).
5. État du registre des risques : nouveaux risques identifiés, évolution des risques existants, efficacité des mesures de traitement (chapitres 2 et 3).
6. Synthèse des indicateurs de performance et de conformité (sections précédentes de ce chapitre).
7. Opportunités d'amélioration continue identifiées.
8. Décisions : allocation de ressources, ajustement des priorités, validation d'acceptations de risque majeures (chapitre 3).

Cette trame illustre concrètement comment l'ensemble des notions vues dans ce module — gouvernance et rôles (chapitre 1), méthodologie de gestion des risques (chapitre 2), traitement et continuité (chapitre 3), pilotage et conformité (chapitre 4) — converge vers une même instance de décision périodique, qui est le véritable moteur de l'amélioration continue d'une gouvernance SSI mature.

### 6.2 De la revue de direction au cycle suivant

Les décisions issues de la revue de direction ne sont pas une fin en soi : elles doivent explicitement réalimenter le cycle de gouvernance décrit dès le premier chapitre. Un ajustement de priorité décidé en revue de direction peut ainsi se traduire par une révision du registre des risques, une mise à jour de la PSSI, une allocation budgétaire nouvelle pour un projet de mise en conformité, ou encore la définition de nouveaux indicateurs à suivre lors du cycle suivant. C'est cette boucle continue, et non une succession de documents produits une fois pour toutes, qui caractérise une gouvernance de la sécurité de l'information réellement mature.

## À retenir

- Un indicateur SSI doit être actionnable, pas seulement descriptif ; il remplit une ou plusieurs fonctions précises (suivi, alerte, justification, comparaison) qu'il est utile d'identifier explicitement lors de sa conception.
- Le tableau de bord se décline selon le public (direction, comité de pilotage, opérationnel) pour rester pertinent à chaque niveau, en écho aux trois niveaux de gouvernance introduits au chapitre 1.
- La conformité réglementaire est une composante de la gestion des risques, pas un objectif indépendant ; elle se pilote au même titre que les autres indicateurs, et doit éviter l'écueil du « théâtre de conformité » en restant ancrée dans une gestion des risques réelle.
- La revue de direction est le point de bouclage du cycle PDCA : elle transforme les indicateurs, les résultats d'audit et l'état du registre des risques en décisions concrètes, qui relancent à leur tour le cycle de gouvernance.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
