# Chapitre 1 — Introduction et panorama OWASP Top 10

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=09-securite-des-applications/slides/01-introduction-owasp-top10.txt){ target=_blank }

## 1. Pourquoi la sécurité applicative est un enjeu central

Les applications (web, mobiles, API) constituent aujourd'hui la surface d'attaque la plus exposée et la plus dynamique des systèmes d'information : accessibles depuis Internet, mises à jour fréquemment, développées avec de nombreuses dépendances tierces. Ce module approfondit chaque classe de vulnérabilité déjà introduite au module *Audit organisation et technique* (chapitre 5), avec un objectif de compréhension technique fine et de mise en œuvre de contre-mesures.

Plusieurs facteurs structurels expliquent pourquoi la couche applicative concentre une part croissante du risque :

- **Exposition directe** : contrairement à un serveur de fichiers interne, une application web ou une API est, par nature, accessible depuis l'extérieur du périmètre réseau, souvent sans intermédiaire filtrant le trafic applicatif en profondeur.
- **Rythme de changement élevé** : le code applicatif évolue en continu (intégration et déploiement continus — CI/CD), ce qui multiplie les occasions d'introduire une régression de sécurité par rapport à une infrastructure plus stable.
- **Dépendances tierces massives** : une application moderne s'appuie sur des dizaines, voire des centaines de bibliothèques externes, dont chacune peut receler une vulnérabilité qui n'a rien à voir avec le code écrit par l'équipe de développement (thème approfondi dans la catégorie « composants vulnérables » ci-dessous).
- **Complexité fonctionnelle** : plus une application offre de fonctionnalités et de points d'entrée (formulaires, téléversements de fichiers, API, webhooks), plus sa surface d'attaque s'étend mécaniquement.

### Sécurité applicative et cycle de vie du logiciel (SDLC)

Un principe désormais largement partagé dans l'industrie est celui du **« shift left »** : plus une vulnérabilité est détectée tôt dans le cycle de développement (conception, code, intégration) plutôt qu'une fois l'application en production, moins son traitement est coûteux et risqué. Ce module s'inscrit dans cette logique en formant à la compréhension technique des vulnérabilités *avant* leur exploitation, en cohérence avec les principes de *security by design* déjà vus dans un module précédent : la sécurité n'est pas une étape ajoutée après coup, mais une propriété construite dès la conception.

## 2. Rappel du classement OWASP Top 10 (édition de référence)

L'**OWASP (Open Worldwide Application Security Project)** est une fondation à but non lucratif qui publie, entretient et met à jour des référentiels de sécurité applicative reconnus internationalement. Le **OWASP Top 10** en est le produit le plus connu : un classement des dix catégories de risques applicatifs jugées les plus critiques, révisé périodiquement à partir de données de contributeurs et de retours de la communauté de la sécurité applicative.

| # | Catégorie | Description synthétique |
|---|---|---|
| 1 | Contrôle d'accès défaillant (*broken access control*) | Un utilisateur parvient à agir en dehors des permissions qui lui sont normalement attribuées (accès aux données d'un autre utilisateur, élévation de privilèges). |
| 2 | Défaillances cryptographiques | Données sensibles insuffisamment protégées en transit ou au repos (algorithmes obsolètes, absence de chiffrement, mauvaise gestion des clés). |
| 3 | Injection | Une donnée non fiable est interprétée comme du code par un interpréteur (SQL, shell, LDAP...) — objet du chapitre 2. |
| 4 | Conception non sécurisée (*insecure design*) | Faille structurelle présente dès la phase de conception, qu'aucune implémentation soignée ne peut corriger a posteriori. |
| 5 | Mauvaise configuration de sécurité | Paramètres par défaut non durcis, fonctionnalités inutiles activées, messages d'erreur trop verbeux. |
| 6 | Composants vulnérables et obsolètes | Utilisation de bibliothèques ou de frameworks contenant des vulnérabilités connues et non corrigées. |
| 7 | Défaillances d'identification et d'authentification | Mécanismes d'authentification ou de session mal conçus ou mal implémentés — objet du chapitre 3. |
| 8 | Défaillances d'intégrité des données et logiciels | Absence de vérification de l'intégrité du code ou des données (mises à jour non signées, désérialisation non fiable). |
| 9 | Journalisation et surveillance insuffisantes | Impossibilité de détecter, d'investiguer ou de réagir à une compromission faute de traces exploitables. |
| 10 | Falsification de requête côté serveur (SSRF) | Le serveur est amené à émettre, à l'insu de son propriétaire, une requête vers une ressource choisie par l'attaquant. |

Ce classement évolue périodiquement en fonction des données réelles de vulnérabilités observées ; il reste la référence pédagogique et professionnelle la plus largement adoptée pour structurer un programme de sécurité applicative. Il convient toutefois de le considérer comme un **point de départ pédagogique et non comme une checklist exhaustive** : une application peut être vulnérable à un risque non couvert explicitement par ces dix catégories, et la maîtrise des *principes* sous-jacents (validation, moindre privilège, défense en profondeur) importe davantage que la mémorisation du classement lui-même.

### CWE, CVE et OWASP Top 10 : trois référentiels à ne pas confondre

- **CWE (Common Weakness Enumeration)** : catalogue de *types génériques* de faiblesses logicielles (ex. CWE-89 pour l'injection SQL), indépendant de tout logiciel particulier. L'OWASP Top 10 s'appuie largement sur des regroupements de CWE.
- **CVE (Common Vulnerabilities and Exposures)** : identifiant unique attribué à une vulnérabilité *précise*, découverte dans un logiciel ou une version donnée (ex. une faille spécifique d'une bibliothèque particulière). Une CVE correspond en général à une ou plusieurs CWE.
- **OWASP Top 10** : classement pédagogique et stratégique de *catégories* de risques applicatifs, destiné à orienter les priorités de formation et de traitement, plutôt qu'un inventaire exhaustif de vulnérabilités individuelles.

## 3. Principe général : ne jamais faire confiance aux entrées

Le fil conducteur de la quasi-totalité des vulnérabilités applicatives est le même : **une donnée provenant d'une source non fiable (utilisateur, système tiers) est traitée comme si elle était fiable**, sans validation ni neutralisation appropriée avant d'être utilisée dans un contexte sensible (requête base de données, commande système, page HTML, chemin de fichier). Ce principe unificateur permet d'aborder chaque classe de vulnérabilité du module comme une variation d'un même problème fondamental, appliqué à des contextes différents.

Le tableau suivant illustre ce principe unique appliqué à des contextes très différents, chacun développé dans un chapitre ultérieur :

| Contexte d'utilisation de la donnée | Conséquence d'une confiance mal placée | Chapitre |
|---|---|---|
| Requête SQL | Injection SQL | 2 |
| Page HTML rendue au navigateur | Cross-Site Scripting (XSS) | 4 |
| Commande shell système | Injection de commande | 2 |
| Chemin de fichier sur le serveur | Traversée de répertoire (*path traversal*), lecture ou écriture de fichiers arbitraires | 2 et 6 |
| Fonction de désérialisation | Exécution de code arbitraire via un objet forgé | 5 et 6 |
| Requête HTTP sortante émise par le serveur | SSRF | 6 |

Il ne s'agit pas d'une liste close : toute nouvelle technologie qui interprète une donnée d'entrée d'une façon ou d'une autre est potentiellement concernée par une nouvelle variante de ce même principe.

### « Toute entrée » inclut plus que les formulaires

Une erreur pédagogique fréquente consiste à limiter mentalement la notion d'« entrée utilisateur » aux champs de formulaire visibles. En réalité, constituent également des entrées non fiables : les en-têtes HTTP (dont le `User-Agent`, le `Referer`, ou des en-têtes personnalisés), les cookies, les paramètres d'URL, le contenu de fichiers téléversés (y compris leurs métadonnées), les réponses d'API tierces, et même des données déjà stockées en base si elles ont été insérées via un canal non fiable en amont. Le principe de méfiance systématique s'applique à toute donnée qui n'a pas été produite et contrôlée exclusivement par le code de confiance lui-même.

## 4. Validation, encodage, échappement : trois mécanismes distincts à ne pas confondre

- **Validation** : vérifier qu'une entrée respecte un format attendu (ex. une adresse e-mail respecte une syntaxe donnée), idéalement par liste blanche (n'accepter que ce qui est explicitement autorisé) plutôt que par liste noire (tenter de bloquer ce qui est connu comme dangereux, toujours incomplète).
- **Encodage/échappement contextuel** : transformer une donnée pour qu'elle soit interprétée comme une donnée inerte et non comme du code exécutable dans le contexte de sortie visé (ex. échapper les caractères spéciaux HTML avant insertion dans une page, pour prévenir le XSS — chapitre 4).
- **Paramétrage** : séparer structurellement le code de la donnée (ex. requêtes préparées en base de données — chapitre 2), la meilleure défense car elle élimine la classe de vulnérabilité par construction plutôt que de tenter de neutraliser après coup une donnée déjà mélangée au code.

### Pourquoi la liste blanche est structurellement supérieure à la liste noire

Une liste noire tente d'énumérer *a priori* tous les motifs dangereux possibles (ex. bloquer les chaînes contenant `<script>`). Ce raisonnement échoue presque systématiquement, pour deux raisons : d'une part, il existe souvent de multiples façons syntaxiques d'exprimer la même charge malveillante (variations de casse, encodages alternatifs, espaces insérés), rendant l'énumération incomplète par construction ; d'autre part, une liste noire doit être mise à jour à chaque nouvelle technique de contournement découverte, alors qu'une liste blanche (n'accepter que des caractères ou motifs explicitement autorisés, ex. uniquement des chiffres pour un identifiant numérique) reste valide indépendamment des techniques de contournement inventées ultérieurement.

### Encodage : un mécanisme intrinsèquement contextuel

Un piège pédagogique fréquent consiste à croire qu'il existe « un » encodage universel suffisant pour se protéger. En réalité, l'encodage correct dépend strictement du contexte de sortie où la donnée est insérée :

| Contexte de sortie | Encodage attendu | Exemple de transformation |
|---|---|---|
| Corps HTML | Encodage d'entités HTML | `<` devient `&lt;` |
| Attribut HTML | Encodage d'entités HTML (y compris guillemets) | `"` devient `&quot;` |
| Bloc JavaScript inline | Échappement JavaScript spécifique | `'` devient `\'` |
| Paramètre d'URL | Encodage pourcentage (*percent-encoding*) | l'espace devient `%20` |
| Requête SQL | Paramétrage (préféré) ou échappement propre au moteur | — |

Appliquer un encodage HTML à une donnée insérée dans un bloc `<script>` par exemple, ne protège pas contre une injection JavaScript : chaque contexte exige sa propre règle d'encodage, ce qui explique pourquoi les frameworks modernes automatisent ce choix en fonction du contexte de rendu plutôt que de laisser le développeur le déterminer manuellement à chaque insertion.

## 5. Défense en profondeur appliquée à la sécurité applicative

Rappel du module *Security by design* : aucune contre-mesure unique n'est infaillible. Une application robuste combine validation des entrées, requêtes paramétrées, contrôle d'accès systématique, moindre privilège des comptes de service, journalisation, et surveillance — de sorte qu'une défaillance isolée d'une couche ne suffise pas à compromettre l'ensemble.

Concrètement, pour une seule et même vulnérabilité potentielle (par exemple une injection SQL), plusieurs couches de défense indépendantes peuvent chacune réduire le risque, même si une couche précédente échoue :

1. **Validation d'entrée** en amont (liste blanche sur le format attendu).
2. **Paramétrage** de la requête (élimine la classe de vulnérabilité par construction).
3. **Moindre privilège** du compte de base de données utilisé par l'application (même en cas d'injection réussie, l'impact reste borné aux permissions du compte).
4. **Journalisation et détection** d'un comportement anormal (requêtes en échec répétées, motifs typiques d'injection).
5. **Isolation réseau** de la base de données, non directement joignable depuis Internet.

Ce raisonnement en couches — dit aussi principe du « fail securely » (en cas d'échec d'un contrôle, le système doit basculer vers un état restrictif et non vers un état permissif par défaut) — constitue la grille de lecture appliquée systématiquement dans les chapitres suivants.

## 6. Méthodologie de ce module

Chaque chapitre suivant traite une famille de vulnérabilités selon la même structure : mécanisme technique de la vulnérabilité, exemple d'exploitation, impact, contre-mesures. Les TP associés permettent une mise en pratique contrôlée sur des cibles pédagogiques dédiées.

### Panorama des outils et méthodes de détection

Au-delà de la compréhension manuelle de chaque vulnérabilité, il existe des familles d'outils standards permettant d'automatiser une partie de leur détection, dont la connaissance générale relève de la culture professionnelle attendue en fin de module :

| Catégorie d'outil | Principe | Moment d'utilisation typique |
|---|---|---|
| **SAST** (*Static Application Security Testing*) | Analyse du code source sans l'exécuter, à la recherche de motifs de code potentiellement vulnérables. | Pendant le développement, intégrable en CI/CD. |
| **DAST** (*Dynamic Application Security Testing*) | Analyse de l'application en cours d'exécution, en lui envoyant des requêtes de test (« boîte noire »). | Sur un environnement de test ou de pré-production. |
| **SCA** (*Software Composition Analysis*) | Inventaire des dépendances tierces et vérification de leur exposition à des vulnérabilités connues (catégorie 6 du Top 10). | En continu, à chaque mise à jour de dépendance. |
| **Test d'intrusion (pentest) manuel** | Exploitation active, par un humain, de vulnérabilités identifiées ou découvertes, avec une compréhension du contexte métier que l'automatisation ne restitue pas. | Ponctuellement, en complément des outils automatisés. |

Aucun de ces outils n'est suffisant isolément : le SAST génère des faux positifs et ne « voit » pas les vulnérabilités de logique métier, le DAST ne couvre que les chemins effectivement testés, le SCA ne détecte que les vulnérabilités déjà répertoriées publiquement. Leur combinaison, complétée par une revue humaine, reste la pratique recommandée.

## À retenir

- L'OWASP Top 10 structure ce module ; chaque chapitre suivant approfondit une ou plusieurs de ses catégories. Il constitue un point de départ pédagogique, pas une checklist exhaustive.
- Le principe unificateur des vulnérabilités applicatives est le traitement d'une entrée non fiable comme si elle était fiable, quel que soit le contexte technique (SQL, HTML, shell, chemin de fichier, désérialisation, requête sortante).
- Validation par liste blanche, encodage contextuel et paramétrage sont trois mécanismes de défense distincts, souvent combinés ; l'encodage doit impérativement être adapté au contexte de sortie exact.
- La défense en profondeur consiste à empiler plusieurs contrôles indépendants, de sorte que l'échec d'une seule couche ne suffise pas à compromettre l'application entière.
- SAST, DAST, SCA et test d'intrusion manuel sont des familles d'outils complémentaires de détection, chacune avec ses forces et ses limites propres.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
