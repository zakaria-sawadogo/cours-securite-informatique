# Chapitre 1 — Introduction : principes de security by design et privacy by design

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/01-introduction-security-by-design.txt){ target=_blank }

## 1. Définition

*Security by design* désigne l'approche consistant à intégrer les exigences de sécurité **dès la phase de conception** d'un système, plutôt que de les ajouter a posteriori sous forme de correctifs après un incident ou un audit. C'est un changement de posture : la sécurité devient une contrainte de conception au même titre que la performance ou l'ergonomie, pas une couche additionnelle optionnelle.

Cette approche s'oppose à ce que l'on appelle parfois la « sécurité en pansement » (*bolt-on security*) : un système conçu sans considération de sécurité, auquel on ajoute ensuite des dispositifs de protection externes (pare-feu, antivirus, module d'authentification greffé après coup) pour compenser des faiblesses structurelles. Le problème de cette approche corrective n'est pas seulement son coût, mais son efficacité intrinsèquement limitée : un dispositif ajouté après coup ne peut compenser que les symptômes visibles d'un problème de conception, pas sa cause racine. Un système dont l'architecture logique mélange par exemple code métier et requêtes SQL construites par concaténation de chaînes reste vulnérable à l'injection quelle que soit la sophistication du pare-feu applicatif placé devant lui.

*Security by design* repose sur trois idées structurantes que le reste de ce module développe :

- la sécurité est une **propriété du système**, résultant de choix de conception, pas seulement de la présence de mécanismes de protection ;
- elle doit être pensée **en amont**, dès la définition des exigences et de l'architecture, et non uniquement lors des tests ou de l'exploitation ;
- elle nécessite une **démarche systématique et reproductible** (modélisation des menaces, principes de conception éprouvés, processus outillé), pas seulement la vigilance individuelle de développeurs expérimentés.

## 2. Pourquoi corriger après coup coûte plus cher

Le coût de correction d'une faille de conception augmente très fortement à mesure que le projet avance : une faille identifiée en phase de conception coûte typiquement un ordre de grandeur de moins à corriger qu'une faille découverte en production, en particulier si elle nécessite de repenser une architecture déjà déployée et adoptée par des utilisateurs. Cette observation motive l'intégration précoce de la sécurité (« shift left »), reprise au chapitre 5 (DevSecOps).

Plusieurs mécanismes concrets expliquent cette augmentation du coût au fil du cycle de vie :

- **Effet d'engagement architectural** : une décision de conception initiale (par exemple, l'absence de séparation claire entre les composants de confiance et les composants exposés) se propage dans de nombreux modules qui en dépendent implicitement. La corriger tardivement impose souvent de retoucher un grand nombre de composants interconnectés, alors qu'elle aurait pu être évitée par un simple choix différent au moment de la conception.
- **Effet de régression** : plus un système est déployé et utilisé, plus une modification structurelle risque de casser des comportements existants sur lesquels des utilisateurs ou des systèmes tiers se sont appuyés (compatibilité ascendante, intégrations externes), ce qui alourdit les tests de non-régression nécessaires.
- **Effet d'urgence post-incident** : une correction menée dans l'urgence après un incident de sécurité avéré se fait souvent sous pression, avec moins de recul et de tests que dans un contexte de conception planifiée, ce qui augmente le risque d'introduire de nouveaux défauts en corrigeant le premier.
- **Coût organisationnel additionnel** : un incident de sécurité découvert en production entraîne typiquement des coûts qui dépassent le seul correctif technique (communication de crise, notification éventuelle des autorités ou des personnes concernées selon la réglementation applicable, perte de confiance des utilisateurs, temps d'immobilisation d'équipes entières pour l'investigation).

!!! example "Illustration pédagogique (cas fictif)"
    Considérons un établissement fictif, « BanqueTest SA », qui développe une application de gestion de comptes. Si l'équipe de conception choisit dès le départ de valider systématiquement l'identité du propriétaire d'un compte avant chaque opération sensible (médiation complète, voir chapitre 3), le coût de cette vérification est négligeable : quelques lignes de code additionnelles dans une couche d'accès aux données déjà centralisée. Si en revanche cette vérification est omise à la conception et qu'une faille d'accès non autorisé à des comptes tiers est découverte après la mise en production, la correction impose de retoucher potentiellement des dizaines de points d'accès aux données disséminés dans l'application, de rejouer des tests de sécurité complets, et de gérer la communication envers les clients concernés.

## 3. Security by design vs sécurité périmétrique

Un modèle de sécurité historique reposait sur une **frontière protégée** (pare-feu, périmètre réseau) protégeant un intérieur considéré comme de confiance. Ce modèle s'avère fragile dès qu'un attaquant franchit le périmètre (par hameçonnage, compromission d'un poste, prestataire tiers compromis) : à l'intérieur, peu de contrôles supplémentaires ralentissent sa progression. *Security by design* pousse à concevoir des systèmes résilients même en présence d'une compromission partielle, préfigurant le modèle zero trust (chapitre 4).

L'image souvent utilisée est celle du « château fort contre l'œuf » : un château fort protégé par des murailles épaisses mais dont l'intérieur est peu compartimenté s'effondre rapidement une fois la muraille franchie ; un œuf, en apparence plus fragile, oppose en réalité une résistance structurelle répartie sur toute sa coquille. *Security by design* cherche à construire des systèmes plus proches du second modèle : la résistance à la compromission n'est pas concentrée sur un unique point de contrôle périphérique, mais répartie à travers l'ensemble de l'architecture (moindre privilège, segmentation, médiation complète à chaque frontière interne, pas seulement à la frontière externe).

## 4. Les grands principes, aperçu

Ce module détaille au chapitre 3 une liste de principes de conception sécurisée reconnus (moindre privilège, défense en profondeur, échec sécurisé, séparation des tâches, simplicité de conception, médiation complète), formalisés notamment par Saltzer et Schroeder dès 1975 et toujours d'actualité. Ces principes ne sont pas spécifiques à une technologie ou un langage de programmation particulier : ils s'appliquent aussi bien à la conception d'un système d'exploitation, d'une application web moderne, que d'une architecture Cloud distribuée. Leur ancienneté relative (près de cinquante ans pour les travaux fondateurs) n'est pas un signe d'obsolescence mais, au contraire, un indice de leur robustesse conceptuelle : ils décrivent des propriétés structurelles des systèmes, indépendantes des modes technologiques.

## 5. Privacy by design

Concept complémentaire, formalisé par Ann Cavoukian (fin des années 1990-2000) et aujourd'hui explicitement exigé par plusieurs réglementations de protection des données (ex. article 25 du RGPD, « protection des données dès la conception et par défaut ») : la protection des données personnelles doit être une caractéristique par défaut d'un système, pas une option activable. Ce concept est développé au chapitre 6.

Il est utile de distinguer d'emblée *security by design* et *privacy by design* : le premier vise la protection du système et de ses actifs contre des actions malveillantes (confidentialité, intégrité, disponibilité au sens large) ; le second vise spécifiquement la protection des personnes dont les données sont traitées, y compris contre des usages parfaitement licites mais excessifs (collecte disproportionnée, conservation trop longue, profilage non désiré). Un système peut être techniquement très sécurisé (chiffrement fort, contrôle d'accès rigoureux) tout en étant peu respectueux de la vie privée s'il collecte et conserve, de façon sécurisée, bien plus de données personnelles que nécessaire. Les deux démarches sont complémentaires, pas substituables l'une à l'autre.

## 6. Sécurité et fonctionnalité : un arbitrage, pas une opposition

Une idée reçue oppose sécurité et facilité d'usage. Une conception mature cherche plutôt à rendre le **comportement sécurisé le comportement par défaut et le plus simple** pour l'utilisateur (ex. authentification à double facteur activée par défaut plutôt que reléguée à un réglage avancé rarement découvert). Une mesure de sécurité qui dégrade excessivement l'expérience utilisateur est souvent contournée en pratique (mots de passe notés sur un post-it, désactivation d'un contrôle jugé trop contraignant) — un échec de conception, pas seulement un problème de discipline des utilisateurs.

Ce constat rejoint un principe plus général de conception centrée sur l'humain (*usable security*) : un contrôle de sécurité qui n'est pas utilisé, ou qui est contourné, a une efficacité pratique nulle voire négative (il crée un faux sentiment de protection). Concevoir un mécanisme de sécurité implique donc de considérer, dès l'origine, le parcours réel de l'utilisateur : combien d'étapes supplémentaires le mécanisme impose-t-il ? Existe-t-il un chemin de contournement plus simple mais moins sûr que l'utilisateur empruntera naturellement sous pression de temps ? Les concepteurs expérimentés cherchent systématiquement à faire correspondre le chemin le plus court pour l'utilisateur avec le chemin le plus sûr, plutôt que de multiplier les frictions en espérant que la discipline individuelle compense un mauvais choix de conception.

## 7. Articulation avec le reste du programme

Ce module synthétise et opérationnalise des notions vues ailleurs dans le programme : vulnérabilités identifiées en *Audit organisation et technique*, gestion des risques du module *Gouvernance*, primitives cryptographiques à intégrer correctement (*Cryptographie*). *Security by design* est la discipline qui transforme ces connaissances en pratiques de conception concrètes, en amont du développement.

Le tableau suivant résume comment chaque chapitre de ce module s'articule avec les autres briques du programme :

| Chapitre de ce module | Lien avec le reste du programme |
|---|---|
| 2. Modélisation des menaces | Utilise les catégories de vulnérabilités et de scénarios d'attaque étudiées en *Audit organisation et technique* comme matière première pour l'identification systématique des menaces |
| 3. Principes de conception sécurisée | S'applique directement aux recommandations correctives issues d'un audit technique |
| 4. Architecture zero trust | Prolonge la segmentation réseau déjà abordée en *Audit organisation et technique* |
| 5. DevSecOps | Opérationnalise la gestion des vulnérabilités et des correctifs dans un cycle de développement outillé |
| 6. Privacy by design | S'articule avec les obligations réglementaires et la gestion des risques du module *Gouvernance* |

## À retenir

- *Security by design* intègre la sécurité dès la conception, réduisant fortement le coût de correction par rapport à une approche corrective a posteriori, car une faille de conception corrigée tôt évite l'effet d'engagement architectural, de régression et d'urgence post-incident propres aux corrections tardives.
- Le modèle de sécurité périmétrique seul est insuffisant face à des attaquants capables de franchir la frontière ; une conception résiliente à la compromission partielle est nécessaire, à l'image d'une résistance structurelle répartie plutôt que concentrée sur un unique point de contrôle.
- Une mesure de sécurité qui dégrade excessivement l'usage est souvent contournée : la sécurité par défaut et simple d'usage est un objectif de conception, pas un vœu pieux — le chemin le plus simple pour l'utilisateur doit coïncider avec le chemin le plus sûr.
- *Security by design* et *privacy by design* sont complémentaires mais distincts : le premier protège le système contre des actions malveillantes, le second protège spécifiquement les personnes dont les données sont traitées.
- Ce module transforme des connaissances déjà rencontrées ailleurs dans le programme (audit, gouvernance des risques, cryptographie) en pratiques concrètes de conception, applicables en amont du développement.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
