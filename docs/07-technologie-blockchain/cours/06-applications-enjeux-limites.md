# Chapitre 6 — Applications, enjeux et limites

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/06-applications-enjeux-limites.txt){ target=_blank }

## 1. Panorama des applications au-delà des cryptomonnaies

- **Finance décentralisée (DeFi)** : prêts, échanges, produits dérivés sans intermédiaire financier centralisé — avec les risques de sécurité vus au chapitre 5 et une volatilité/régulation encore incertaine.
- **Traçabilité de chaîne logistique** : suivi de provenance de produits (agroalimentaire, textile, minerais), où la blockchain garantit l'intégrité des enregistrements une fois saisis, sans garantir la véracité de la saisie initiale (problème de l'oracle, rappel chapitre 4).
- **Identité numérique décentralisée (self-sovereign identity)** : permettre à un individu de contrôler et prouver sélectivement des attributs de son identité sans dépendre d'un registre central unique.
- **Certification et notarisation** : preuve d'existence et d'horodatage d'un document à une date donnée (ancrage de l'empreinte du document sur une blockchain publique) — usage simple et robuste, sans manipuler de valeur financière.
- **Vote électronique** : cas d'usage souvent évoqué mais particulièrement délicat en pratique, car il combine des exigences difficilement conciliables (anonymat de l'électeur, vérifiabilité publique du résultat, résistance à la coercition) — objet de recherche active plutôt que de déploiements matures à grande échelle.

### Le contrat de vote du chapitre 4, revisité à la lumière de ces exigences

Le contrat pédagogique `VoteSimple` présenté au chapitre 4 illustre concrètement les tensions évoquées ci-dessus : il assure une vérifiabilité publique parfaite (chacun peut recompter les voix directement depuis la blockchain), mais aucun anonymat (chaque vote reste associé de façon permanente à l'adresse de l'électeur qui l'a émis) et aucune résistance à la coercition (un tiers pourrait exiger d'un électeur qu'il prouve, en lui montrant la transaction correspondante, avoir voté dans un sens donné). Concilier ces trois exigences simultanément nécessite des constructions cryptographiques bien plus sophistiquées (preuves à divulgation nulle de connaissance, signatures en anneau, notamment), qui dépassent le cadre de ce cours introductif mais expliquent pourquoi le vote électronique décentralisé reste, à ce jour, davantage un sujet de recherche qu'une solution mature déployée à grande échelle pour des élections officielles.

### Identité numérique décentralisée : approfondissement

Le concept de *self-sovereign identity* mérite d'être détaillé davantage, car il illustre un usage de la blockchain qui ne repose pas nécessairement sur la manipulation de valeur financière. L'idée centrale est de permettre à un individu de contrôler directement des **attestations vérifiables** (par exemple, un diplôme délivré par une université, ou une preuve de majorité) émises et signées cryptographiquement par une autorité de confiance (l'université, l'État), sans que cette autorité doive être interrogée à chaque vérification ultérieure. La blockchain intervient alors typiquement non pas pour stocker directement les données personnelles (ce qui poserait un problème majeur au regard du droit à l'effacement des données personnelles, développé en section 3), mais pour ancrer un registre public des clés de signature des autorités habilitées à émettre de telles attestations, ou pour révoquer une attestation compromise — un usage plus restreint et mieux délimité que ne le laisse parfois entendre la communication marketing autour de ce sujet.

## 2. Trilemme de la blockchain (scalabilité, sécurité, décentralisation)

Les concepteurs de blockchains publiques font face à un compromis reconnu : il est difficile d'optimiser simultanément la **scalabilité** (nombre de transactions traitées par seconde), la **sécurité** et la **décentralisation**. Les architectures existantes privilégient généralement deux de ces trois propriétés au détriment de la troisième (ex. Bitcoin privilégie sécurité et décentralisation au prix d'une faible scalabilité ; certaines blockchains à permission privilégient scalabilité et sécurité au prix d'une décentralisation réduite).

### Pourquoi ce compromis est-il structurel et non simplement un problème d'ingénierie non résolu ?

Il est pédagogiquement utile d'expliquer *pourquoi* ce trilemme n'est pas qu'une limite technologique provisoire, mais découle de contraintes structurelles reliées aux chapitres précédents de ce cours :

- augmenter le **débit** de transactions traitées implique généralement d'augmenter la taille des blocs ou de réduire l'intervalle de temps entre deux blocs ;
- augmenter la taille des blocs ou réduire l'intervalle entre blocs augmente le temps nécessaire à leur **propagation** sur l'ensemble du réseau pair-à-pair, augmentant mécaniquement la probabilité de forks accidentels (chapitre 3) ;
- pour limiter ce risque, il faudrait réduire le nombre de nœuds participant à la validation (moins de nœuds à qui propager, moins de latence de convergence) — ce qui revient précisément à sacrifier la **décentralisation** ;
- à l'inverse, maintenir un grand nombre de nœuds validateurs indépendants et géographiquement dispersés (forte décentralisation) impose de conserver des blocs suffisamment petits et un intervalle de temps suffisamment grand pour que la propagation reste fiable, ce qui plafonne mécaniquement le débit maximal.

Ce raisonnement, volontairement simplifié, illustre que le trilemme n'oppose pas des choix arbitraires mais des contraintes physiques et réseau bien réelles (latence de propagation, bande passante disponible chez chaque nœud) — un rappel utile face aux discours commerciaux annonçant parfois avoir « résolu » le trilemme sans compromis, alors qu'il s'agit presque toujours d'un déplacement du curseur vers un point différent du compromis, pas d'une élimination du compromis lui-même.

### Approches de mise à l'échelle (aperçu)

- **Solutions de couche 2 (Layer 2)** : traiter la majorité des transactions hors chaîne principale, en n'ancrant sur la chaîne principale qu'un résumé périodique (ex. rollups sur Ethereum).
- **Sharding** : répartir l'état et le traitement entre plusieurs sous-réseaux (« shards ») traités en parallèle.

### Approfondissement : le principe des rollups

Les *rollups*, mentionnés ci-dessus comme exemple de solution de couche 2, méritent d'être détaillés à titre d'illustration pédagogique du principe général des solutions hors chaîne. Le principe consiste à exécuter et regrouper (« rouler ») un grand nombre de transactions hors de la chaîne principale, sur un réseau secondaire dédié, puis à ne publier sur la chaîne principale qu'un résumé compact de l'effet cumulé de ces transactions (par exemple une nouvelle racine d'état, au sens du state trie présenté au chapitre 2), accompagné d'une preuve de la validité de ce résumé. Deux grandes familles existent, se distinguant par la nature de cette preuve :

- les **rollups optimistes** publient le résumé en supposant qu'il est correct par défaut, mais ouvrent une fenêtre de contestation pendant laquelle tout observateur peut soumettre une preuve de fraude s'il détecte une exécution incorrecte, auquel cas le résumé contesté est annulé ;
- les **rollups à validité (zk-rollups)** accompagnent chaque résumé d'une preuve cryptographique à divulgation nulle de connaissance, vérifiable rapidement par la chaîne principale, garantissant mathématiquement la correction du résumé sans attendre de fenêtre de contestation ni faire confiance à un tiers.

Dans les deux cas, la chaîne principale continue de jouer son rôle de source ultime de sécurité et de disponibilité des données (elle reste seule garante de l'ordre final et de l'intégrité globale), tandis que l'essentiel du volume de calcul et de transactions est déporté hors chaîne — un exemple concret de la façon dont l'écosystème contourne partiellement le trilemme en séparant les rôles entre plusieurs couches de réseau plutôt qu'en cherchant une solution monolithique unique.

### Approfondissement : limites pratiques du sharding

Le sharding, bien qu'il permette en théorie un débit proportionnel au nombre de sous-réseaux (« shards »), introduit des difficultés d'ingénierie substantielles qui expliquent sa mise en œuvre progressive et prudente dans l'écosystème : une transaction impliquant des comptes répartis sur deux shards différents (une **transaction inter-shard**) nécessite une coordination supplémentaire, plus coûteuse et plus lente qu'une transaction interne à un seul shard ; la sécurité de chaque shard pris individuellement doit rester suffisante (un attaquant ne doit pas pouvoir concentrer une majorité de puissance de validation sur un seul shard plus vulnérable, un risque parfois appelé attaque « à un seul shard ») ; enfin, la disponibilité et la cohérence des données doivent être garanties à travers l'ensemble des shards pour que le réseau conserve une vue globale fiable de son propre état.

## 3. Enjeux réglementaires

- **Lutte contre le blanchiment (AML) et connaissance du client (KYC)** : tension entre pseudonymat des blockchains publiques et obligations réglementaires imposées aux plateformes d'échange.
- **Qualification juridique des cryptoactifs** : selon les juridictions, un jeton peut être qualifié différemment (monnaie, valeur mobilière, actif numérique sui generis), avec des conséquences fiscales et réglementaires très différentes.
- **Protection des consommateurs** : volatilité, risque de perte totale et irréversible en cas d'erreur de manipulation (perte de clé privée) ou d'escroquerie (absence de recours équivalent à un système bancaire traditionnel).
- **Encadrement progressif** : plusieurs juridictions ont mis en place ou développent des cadres réglementaires dédiés aux actifs numériques et aux prestataires de services associés ; la situation évolue rapidement et diffère fortement d'un pays à l'autre.

### La tension pseudonymat/traçabilité en détail

Un point mérite d'être approfondi pour dissiper une confusion fréquente : une blockchain publique comme Bitcoin ou Ethereum n'est en réalité **pas anonyme, mais pseudonyme**. Chaque adresse est un identifiant durable (une chaîne de caractères dérivée d'une clé publique, chapitre 2), et l'intégralité des transactions associées à cette adresse est publiquement consultable et immuable. Ce pseudonymat offre une confidentialité bien moindre qu'il n'y paraît de prime abord : des techniques d'analyse de graphe des transactions (regrouper des adresses probablement contrôlées par un même acteur, à partir de motifs de dépense caractéristiques) permettent, dans de nombreux cas documentés, de relier une adresse à une identité réelle, en particulier au moment où des fonds transitent par une plateforme d'échange soumise à des obligations de connaissance du client (KYC). C'est précisément cette combinaison — pseudonymat des transactions sur la chaîne, mais identification obligatoire aux points d'entrée et de sortie que constituent les plateformes réglementées — qui structure l'essentiel de l'approche réglementaire actuelle de la lutte contre le blanchiment appliquée aux cryptoactifs, plutôt qu'une tentative d'interdire ou de casser techniquement le pseudonymat lui-même.

## 4. Limites techniques et environnementales

- Consommation énergétique significative pour les réseaux en preuve de travail (chapitre 3), bien que la transition vers la preuve d'enjeu réduise fortement cet impact pour les réseaux concernés.
- Irréversibilité des transactions : un avantage en termes de finalité, mais un risque important en cas d'erreur humaine ou de vol (pas de « bouton d'annulation »).
- Complexité d'usage pour l'utilisateur final (gestion de clés privées, absence de recours en cas de perte), un frein réel à l'adoption grand public.

### Le stockage : une ressource qui ne cesse de croître

Une limite technique moins souvent discutée que le débit ou l'énergie mérite d'être signalée : dans une blockchain publique classique (sans mécanisme d'élagage spécifique), chaque nœud complet doit conserver l'intégralité de l'historique depuis le tout premier bloc pour pouvoir valider indépendamment n'importe quelle transaction — un volume de données qui croît de façon monotone au fil du temps, sans jamais diminuer. Cette croissance continue pose un dilemme structurel : plus l'historique devient volumineux, plus il devient coûteux (en stockage, en bande passante) pour un nouveau participant de rejoindre le réseau en tant que nœud complet, ce qui tend à concentrer, avec le temps, le rôle de validation complète chez un nombre plus restreint d'acteurs disposant des ressources nécessaires — une tension directe avec l'objectif de décentralisation à long terme, distincte du trilemme scalabilité/sécurité/décentralisation présenté en section 2 mais qui lui est étroitement liée.

### Sécurité de la clé privée : le talon d'Achille de l'utilisateur final

La complexité d'usage mentionnée ci-dessus mérite d'être reliée explicitement au module *Cryptographie* de ce cours : la sécurité de l'intégralité des fonds associés à une adresse repose entièrement sur la confidentialité d'une seule clé privée (ou de la phrase mnémonique qui permet de la régénérer). Contrairement à un compte bancaire classique, il n'existe généralement aucun mécanisme de récupération de mot de passe oublié, aucune procédure de contestation d'une opération frauduleuse déjà confirmée, et aucune assurance obligatoire couvrant la perte. Cette situation déplace une responsabilité de sécurité normalement assumée par une institution financière (protection de l'infrastructure, procédures de recours) directement sur l'utilisateur individuel, qui doit lui-même appliquer de bonnes pratiques de gestion de clés (stockage hors ligne dans un « portefeuille froid » pour des montants significatifs, sauvegardes multiples de la phrase mnémonique, méfiance envers le hameçonnage ciblant spécifiquement les détenteurs de cryptoactifs) — un exemple concret des principes de gestion de clés étudiés de façon plus générale dans le module *Cryptographie*, appliqué ici à un contexte où l'erreur est immédiatement et irréversiblement sanctionnée financièrement.

## 5. Regard critique : quand la blockchain est-elle pertinente ?

Une grille de questions utile avant d'adopter une solution blockchain pour un projet donné : a-t-on réellement besoin de décentralisation, ou une base de données classique avec un tiers de confiance suffit-elle ? Plusieurs parties mutuellement méfiantes doivent-elles partager un registre sans intermédiaire ? L'immutabilité est-elle un avantage réel pour ce cas d'usage, ou une contrainte gênante (ex. droit à l'effacement en matière de données personnelles) ? De nombreux projets « blockchain » annoncés ces dernières années auraient pu être résolus plus simplement par une architecture centralisée classique.

### Une grille de décision synthétique

Le tableau suivant formalise, à titre d'outil pédagogique pour ce cours, la grille de questions ci-dessus sous une forme structurée, utilisable comme point de départ pour l'analyse d'un projet fictif proposé en exercice :

| Question | Si la réponse est « non » | Si la réponse est « oui » |
|---|---|---|
| Plusieurs parties mutuellement méfiantes doivent-elles partager un état commun sans intermédiaire de confiance unique ? | une base de données classique, éventuellement répliquée, répond probablement mieux au besoin | la décentralisation apporte une valeur réelle à considérer |
| Le débit de transactions requis est-il compatible avec les mécanismes de consensus disponibles (chapitre 3), éventuellement via une solution de couche 2 ? | risque de goulot d'étranglement majeur, à anticiper dès la conception | la contrainte de scalabilité est gérable |
| L'immutabilité est-elle un avantage recherché (auditabilité, non-répudiation) plutôt qu'une contrainte (droit à l'effacement, correction d'erreurs) ? | l'immutabilité pourrait devenir un passif juridique ou opérationnel | l'immutabilité renforce la proposition de valeur du projet |
| Les données sensibles peuvent-elles rester hors chaîne (hachées ou référencées uniquement) plutôt que stockées en clair sur un registre public ? | risque de conflit avec la réglementation sur les données personnelles | la conception peut concilier transparence et protection des données |

Cette grille ne prétend pas fournir une réponse automatique, mais structurer la discussion critique attendue de tout futur ingénieur confronté à une proposition de projet « blockchain » — un exercice d'esprit critique directement transposable aux travaux pratiques de ce module.

## À retenir

- Les applications de la blockchain dépassent les cryptomonnaies (identité décentralisée, traçabilité, certification) mais restent souvent confrontées au problème de l'oracle (chapitre 4) et au trilemme scalabilité/sécurité/décentralisation, un compromis structurel lié aux contraintes physiques de propagation réseau, pas une simple limite d'ingénierie provisoire.
- Les solutions de couche 2 (rollups optimistes ou à validité) et le sharding atténuent partiellement ce trilemme en répartissant les rôles entre plusieurs couches ou sous-réseaux, sans jamais l'éliminer complètement.
- Le pseudonymat (et non l'anonymat) des blockchains publiques structure l'approche réglementaire actuelle de lutte contre le blanchiment, centrée sur l'identification aux points d'entrée/sortie (plateformes d'échange) plutôt que sur la chaîne elle-même.
- Le cadre réglementaire des cryptoactifs est encore en construction et varie fortement selon les juridictions ; la qualification juridique d'un jeton a des conséquences fiscales et réglementaires directes.
- La gestion de la clé privée déplace vers l'utilisateur final une responsabilité de sécurité normalement assumée par une institution financière, sans mécanisme de recours en cas de perte ou de vol — un point de fragilité majeur pour l'adoption grand public.
- La pertinence d'une solution blockchain doit être évaluée par rapport à un besoin réel de décentralisation, à l'aide d'une grille de décision explicite, et non adoptée par principe ou effet de mode.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
