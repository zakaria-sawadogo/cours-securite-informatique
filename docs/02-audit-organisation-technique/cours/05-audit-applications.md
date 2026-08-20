# Chapitre 5 — Audit des applications et du code

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/05-audit-applications.txt){ target=_blank }

## 1. Pourquoi auditer les applications spécifiquement

Les applications web et métier constituent la surface d'attaque la plus exposée et la plus évolutive (déploiements fréquents, code sur mesure). Ce chapitre prépare le cours *Sécurité des applications*, qui approfondit chaque type de vulnérabilité.

Cette spécificité mérite d'être expliquée : contrairement à un équipement réseau ou à un système d'exploitation, dont la sécurité repose largement sur des configurations standardisées et des correctifs fournis par l'éditeur, une application métier développée sur mesure ne bénéficie d'aucun filet de sécurité équivalent. Chaque ligne de code métier est une occasion potentielle d'introduire une vulnérabilité, et cette surface évolue à chaque nouvelle fonctionnalité livrée — dans un contexte d'intégration et de déploiement continus (CI/CD), il n'est pas rare qu'une application en production évolue plusieurs fois par semaine, voire par jour. Cette caractéristique explique pourquoi l'audit applicatif ne peut se limiter à un contrôle ponctuel annuel : il doit s'articuler avec des contrôles automatisés intégrés au cycle de développement, un point développé plus loin dans ce chapitre (section 3).

Un autre facteur d'exposition est la nature même des applications web : elles sont, par construction, accessibles à un grand nombre d'utilisateurs, parfois anonymes, ce qui en fait une cible de choix pour des attaques automatisées à grande échelle (recherche opportuniste de vulnérabilités connues sur des composants tiers largement diffusés) autant que pour des attaques ciblées.

## 2. OWASP Top 10 comme grille d'audit

Le classement OWASP Top 10 recense les catégories de vulnérabilités applicatives les plus critiques et sert de checklist minimale pour tout audit applicatif :

1. Contrôle d'accès défaillant (*broken access control*)
2. Défaillances cryptographiques
3. Injection (SQL, commande, etc.)
4. Conception non sécurisée (*insecure design*)
5. Mauvaise configuration de sécurité
6. Composants vulnérables et obsolètes
7. Défaillances d'identification et d'authentification
8. Défaillances d'intégrité des données et logiciels
9. Journalisation et surveillance insuffisantes
10. Falsification de requête côté serveur (SSRF)

L'OWASP (*Open Web Application Security Project*) publie et met à jour périodiquement ce classement à partir de données agrégées auprès d'organisations et de contributeurs de la communauté ; il ne s'agit pas d'un palmarès figé mais d'un instantané évolutif des catégories de risque jugées les plus significatives à un moment donné. Pour un auditeur, l'intérêt principal du Top 10 n'est pas tant la liste elle-même — qui reste une catégorisation de haut niveau — que la richesse documentaire qui l'accompagne (guide de test associé, exemples de code vulnérable et corrigé, références vers des ressources complémentaires), largement mobilisée dans la conception de méthodologies de test détaillées.

Il est utile de préciser certaines catégories qui prêtent parfois à confusion :

- **Contrôle d'accès défaillant** (catégorie n°1 dans les versions récentes du classement) regroupe des défauts très divers : accès direct à une ressource par manipulation d'identifiant dans l'URL (IDOR, *Insecure Direct Object Reference*), élévation de privilèges horizontale (accéder aux données d'un autre utilisateur de même niveau) ou verticale (accéder à des fonctionnalités réservées à un rôle supérieur), absence de vérification côté serveur d'une restriction appliquée uniquement côté client.
- **Conception non sécurisée** (*insecure design*) se distingue d'une simple erreur d'implémentation : il s'agit d'un défaut présent dès la phase de conception (par exemple, un mécanisme de récupération de mot de passe reposant sur des questions secrètes devinables), qui ne peut être corrigé par un simple correctif de code mais nécessite de repenser la fonctionnalité elle-même.
- **SSRF** (*Server-Side Request Forgery*) est une catégorie plus récemment mise en avant : elle consiste à forcer le serveur applicatif à émettre, pour le compte de l'attaquant, des requêtes vers des ressources normalement inaccessibles depuis l'extérieur (services internes, métadonnées d'instance Cloud) — une vulnérabilité de plus en plus critique avec la généralisation des architectures Cloud, où les métadonnées d'instance exposent parfois des identifiants temporaires à privilèges élevés.

### Piège courant : réduire l'audit applicatif au seul Top 10

Le Top 10 est une base de départ, pas une checklist exhaustive : il ne couvre volontairement que les dix catégories jugées les plus significatives à l'échelle globale, et ne remplace pas une analyse spécifique à la logique métier de l'application auditée. Une application de gestion financière, par exemple, peut présenter des vulnérabilités logiques très spécifiques (contournement d'un plafond de transaction, manipulation du calcul d'une remise) qui n'entrent dans aucune des dix catégories au sens strict mais représentent un risque métier tout aussi réel. Un auditeur expérimenté utilise le Top 10 comme point de départ méthodologique, puis l'enrichit systématiquement d'une analyse dédiée à la logique métier propre à chaque application.

## 3. Approches d'audit applicatif

| Approche | Description | Outils typiques |
|---|---|---|
| **DAST** (Dynamic Application Security Testing) | tests en boîte noire sur l'application en fonctionnement | OWASP ZAP, Burp Suite |
| **SAST** (Static Application Security Testing) | analyse du code source sans exécution | SonarQube, Semgrep, Bandit (Python), Cppcheck (C/C++) |
| **SCA** (Software Composition Analysis) | analyse des dépendances tierces et de leurs vulnérabilités connues | OWASP Dependency-Check, `npm audit` |
| **Revue manuelle de code** | lecture experte, notamment pour la logique métier | — |

Une démarche mature combine ces approches : le SAST/SCA s'intègre au pipeline CI/CD pour un contrôle continu, tandis que le DAST et la revue manuelle interviennent à intervalles réguliers ou avant une mise en production majeure.

Chacune de ces quatre approches présente des forces et des limites qu'un auditeur doit connaître pour les combiner efficacement plutôt que de considérer qu'une seule suffit :

- Le **DAST** teste l'application « de l'extérieur », comme le ferait un attaquant réel, sans connaissance du code source. Son avantage est de refléter fidèlement ce qui est réellement exposé et exploitable ; sa limite est qu'il ne couvre que les chemins d'exécution effectivement atteints pendant le test (un formulaire non découvert par le crawler de l'outil ne sera jamais testé) et qu'il peine à détecter des vulnérabilités sans effet observable immédiat.
- Le **SAST** analyse le code source sans l'exécuter, ce qui lui permet de couvrir l'intégralité du code, y compris les chemins rarement empruntés en pratique ; en contrepartie, il génère généralement davantage de faux positifs qu'un DAST bien calibré, car il manque le contexte d'exécution réel (une donnée signalée comme non filtrée par l'analyse statique peut, en pratique, être filtrée par un composant tiers non analysé).
- Le **SCA** répond à un problème spécifique et de plus en plus critique : la majorité du code d'une application moderne provient de bibliothèques tierces (open source ou commerciales), dont les vulnérabilités connues (CVE) sont documentées dans des bases publiques. Un SCA compare l'inventaire des dépendances utilisées (souvent formalisé dans une nomenclature logicielle, ou *Software Bill of Materials*, SBOM) à ces bases pour signaler les composants à mettre à jour.
- La **revue manuelle de code** reste irremplaçable pour la logique métier et les vulnérabilités de conception, qu'aucun outil automatisé ne peut détecter par nature, faute de comprendre l'intention fonctionnelle du code.

```bash
# Exemple d'utilisation d'un outil SCA en ligne de commande
npm audit --audit-level=high

# Exemple d'analyse SAST légère avec Semgrep sur un dépôt de code
semgrep --config=auto ./src
```

### Bonne pratique : intégrer les contrôles dans le pipeline CI/CD (DevSecOps)

L'intégration du SAST et du SCA directement dans le pipeline d'intégration continue — de sorte qu'une vulnérabilité critique nouvellement introduite bloque automatiquement la fusion d'une modification de code ou le déploiement — est aujourd'hui considérée comme une pratique de référence, souvent désignée sous le terme de **DevSecOps** (intégration de la sécurité comme responsabilité partagée tout au long du cycle de développement, plutôt que comme une étape isolée en fin de projet). Un audit applicatif peut utilement évaluer, en plus des vulnérabilités elles-mêmes, la maturité de cette intégration : la présence de contrôles automatisés dans le pipeline est un indicateur fort de la capacité de l'organisation à maintenir un niveau de sécurité dans la durée, bien au-delà de l'instant de l'audit.

## 4. Méthodologie d'un pentest applicatif

1. **Cartographie** de l'application (pages, fonctionnalités, rôles utilisateurs, points d'entrée de données).
2. **Tests d'authentification et de gestion de session** (politique de mot de passe, expiration de session, protection contre le brute force).
3. **Tests de contrôle d'accès** (un utilisateur standard peut-il accéder à des ressources d'un autre utilisateur ou d'un administrateur en modifiant un identifiant dans l'URL — IDOR ?).
4. **Tests d'injection** (SQL, commande système, LDAP) sur chaque point de saisie.
5. **Tests côté client** (XSS stocké/réfléchi, CSRF).
6. **Tests de configuration** (en-têtes de sécurité HTTP, gestion des erreurs, informations sensibles exposées).

Cette méthodologie appelle plusieurs précisions pratiques utiles à un auditeur en formation :

**La cartographie** (étape 1) doit être réalisée avec au minimum un compte pour chaque rôle applicatif distinct (utilisateur standard, gestionnaire, administrateur), afin de pouvoir ensuite comparer systématiquement ce que chaque rôle peut et ne peut pas atteindre — une étape indispensable pour la détection des défaillances de contrôle d'accès à l'étape 3.

**Les tests d'authentification** (étape 2) portent notamment sur : la robustesse de la politique de mot de passe (longueur, complexité, vérification contre des listes de mots de passe compromis connus) ; la protection contre les attaques par force brute (verrouillage temporaire de compte, limitation du débit de requêtes) ; la sécurité du mécanisme de récupération de mot de passe (souvent le maillon faible, car conçu comme une voie de contournement de l'authentification normale) ; et la gestion du cycle de vie de la session (expiration après inactivité, invalidation effective du jeton lors de la déconnexion, régénération de l'identifiant de session après authentification pour prévenir la fixation de session).

**Les tests de contrôle d'accès** (étape 3), souvent réalisés en confrontant systématiquement les requêtes générées par un compte à privilèges élevés à un compte de moindre privilège (remplacement de l'identifiant de session, sans autre modification), sont probablement les plus révélateurs en pratique : les défaillances de contrôle d'accès figurent en tête du classement OWASP Top 10 précisément parce qu'elles sont fréquentes, souvent faciles à exploiter (aucune compétence technique avancée n'est requise, une simple modification d'URL ou de paramètre peut suffire) et à fort impact.

**Les tests d'injection** (étape 4) doivent être systématiquement appliqués à **chaque point de saisie** de l'application, y compris ceux qui semblent a priori peu exposés (en-têtes HTTP personnalisés, paramètres de tri ou de filtre, champs de recherche, en-têtes `User-Agent` ou `Referer` parfois journalisés puis affichés sans échappement). Un exemple d'extrait de charge utile pédagogique illustrant une tentative de détection d'injection SQL par observation du comportement (méthode dite « à l'aveugle », ou *blind SQL injection*) :

```text
# Payload de détection (comportement conditionnel attendu si vulnérable)
' AND 1=1 -- 
' AND 1=2 -- 
```

Si l'application renvoie une réponse différente entre ces deux charges utiles (par exemple, un résultat affiché dans le premier cas et une page vide dans le second), cela constitue un indice fort d'injection SQL exploitable, à confirmer ensuite par une preuve de concept plus poussée dans le respect du mandat.

**Les tests côté client** (étape 5) portent sur les vulnérabilités qui s'exécutent dans le navigateur de la victime plutôt que sur le serveur : le XSS (*Cross-Site Scripting*) stocké est généralement considéré comme plus critique que le XSS réfléchi, car il affecte tous les visiteurs consultant la page compromise, sans nécessiter d'interaction spécifique de la victime (contrairement au XSS réfléchi, qui nécessite généralement que la victime clique sur un lien piégé). Le CSRF (*Cross-Site Request Forgery*) exploite la confiance qu'une application accorde à une session authentifiée, en forçant le navigateur de la victime à émettre, à son insu, une requête vers l'application ciblée.

**Les tests de configuration** (étape 6) incluent la vérification des en-têtes de sécurité HTTP :

```bash
# Vérifier les en-têtes de sécurité renvoyés par une application web
curl -I https://exemple-pedagogique.tld
```

Des en-têtes comme `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options` ou `X-Frame-Options` réduisent, sans les éliminer, certaines classes d'attaques côté client ; leur absence n'est généralement pas critique isolément mais constitue un facteur aggravant en présence d'autres vulnérabilités, et leur présence est un indicateur de la maturité générale de l'équipe de développement en matière de sécurité.

### Piège courant : tester uniquement les fonctionnalités visibles

Un piège classique consiste à limiter les tests aux fonctionnalités accessibles depuis l'interface utilisateur standard, en négligeant les points d'entrée moins visibles mais tout aussi exploitables : API exposées mais non documentées publiquement (souvent découvertes par l'inspection du trafic réseau généré par l'application), fonctionnalités d'import/export de données, interfaces d'administration accessibles sur un chemin distinct mais non protégées par un contrôle d'accès réseau. Une cartographie rigoureuse (étape 1), incluant l'analyse du trafic réseau généré par l'application dans un navigateur, réduit fortement ce risque d'angle mort.

## 5. Audit de code : exemple de grille (langage C, en lien avec le module Algorithmique)

| Point de contrôle | Risque associé |
|---|---|
| Utilisation de `strcpy`, `gets`, `sprintf` sans borne | débordement de tampon |
| Absence de vérification du retour de `malloc` | déréférencement de pointeur nul |
| `free` sans remise à `NULL` | use-after-free |
| Chaîne de format non constante passée à `printf` | vulnérabilité format string |
| Calculs de taille non vérifiés (multiplication avant `malloc`) | débordement d'entier |

Ces cinq points de contrôle, bien que spécifiques au langage C, illustrent une catégorie de vulnérabilités — les erreurs de gestion mémoire — historiquement responsable d'une part très importante des vulnérabilités critiques découvertes dans les logiciels bas niveau (systèmes d'exploitation, navigateurs, serveurs). Ils méritent d'être développés :

- **Débordement de tampon** (*buffer overflow*) : survient lorsqu'une écriture en mémoire dépasse la taille allouée à la structure de destination, pouvant écraser des données adjacentes (variables voisines, adresse de retour d'une fonction sur la pile), ce qui peut, dans le pire des cas, permettre l'exécution de code arbitraire. Les fonctions historiques comme `strcpy`, `gets` ou `sprintf` ne vérifient pas la taille de la destination et doivent être systématiquement remplacées par leurs équivalents bornés (`strncpy`, `fgets`, `snprintf`), eux-mêmes à utiliser avec attention (une troncature silencieuse mal gérée peut créer d'autres classes de bugs).
- **Déréférencement de pointeur nul** : `malloc` retourne `NULL` en cas d'échec d'allocation ; ignorer cette possibilité et déréférencer directement le pointeur retourné provoque un plantage (déni de service) et, dans certains contextes, peut être combiné à d'autres défauts pour un impact plus grave.
- **Use-after-free** : utiliser une zone mémoire après l'avoir libérée par `free` est un défaut particulièrement dangereux et difficile à détecter par relecture simple du code, car le programme peut continuer à fonctionner apparemment normalement pendant longtemps avant de manifester un comportement erratique ou exploitable ; remettre systématiquement le pointeur à `NULL` après un `free` (et vérifier cette valeur avant toute réutilisation) est une mesure de précaution simple et largement recommandée.
- **Vulnérabilité format string** : passer une donnée contrôlée par l'utilisateur directement comme chaîne de format à `printf` (au lieu de la passer comme argument d'un format constant, par exemple `printf("%s", donnee)` plutôt que `printf(donnee)`) permet potentiellement à un attaquant de lire, voire d'écrire, des zones arbitraires de la mémoire du processus.
- **Débordement d'entier** : un calcul de taille effectué avant un appel à `malloc` (par exemple, `n * taille_element`) peut déborder silencieusement si `n` est contrôlé par un attaquant et suffisamment grand, aboutissant à une allocation plus petite que prévu et donc à un débordement de tampon ultérieur lors du remplissage de la structure.

```c
/* Exemple pédagogique : correction d'un point de contrôle classique */
/* Version vulnérable */
char destination[16];
strcpy(destination, source); /* pas de vérification de taille */

/* Version corrigée */
char destination[16];
strncpy(destination, source, sizeof(destination) - 1);
destination[sizeof(destination) - 1] = '\0'; /* garantir la terminaison */
```

### Approfondissement : outils d'analyse pour le code bas niveau

Au-delà de la revue manuelle, plusieurs outils facilitent la détection de ces classes de vulnérabilités dans le code C/C++ :

- **Analyseurs statiques** : `Cppcheck`, les avertissements du compilateur activés au niveau maximal (`-Wall -Wextra` avec GCC/Clang), ou des outils plus avancés d'analyse de flux de données.
- **Outils dynamiques d'instrumentation** : `Valgrind` (détection de fuites mémoire et d'accès invalides à l'exécution), `AddressSanitizer` (instrumentation à la compilation détectant débordements et *use-after-free* avec un net gain de performance par rapport à Valgrind).
- **Fuzzing** : technique consistant à soumettre au programme un grand nombre d'entrées aléatoires ou mutées automatiquement, à la recherche de plantages révélateurs de vulnérabilités ; particulièrement efficace sur du code d'analyse de formats de fichiers ou de protocoles réseau, où la surface d'entrées possibles est vaste.

## 6. Limites

Un audit applicatif ponctuel ne couvre que l'état du code à un instant donné : toute évolution ultérieure du code peut réintroduire des vulnérabilités déjà corrigées, d'où l'intérêt d'intégrer des contrôles automatisés (SAST/SCA) en continu plutôt que de se reposer uniquement sur un audit annuel.

D'autres limites méritent d'être mentionnées pour donner une vision réaliste des attentes qu'un commanditaire peut avoir vis-à-vis d'un audit applicatif :

- **Couverture fonctionnelle incomplète** : dans le temps contraint d'une mission, il est rarement possible de tester exhaustivement toutes les combinaisons de rôles, de fonctionnalités et de données possibles ; l'auditeur doit donc prioriser les fonctionnalités les plus sensibles (paiement, gestion des accès, données personnelles) plutôt que de chercher une couverture uniforme.
- **Dépendance à l'environnement de test fourni** : un audit réalisé sur un environnement de test peut ne pas refléter parfaitement la configuration de production (données de volume différent, intégrations tierces désactivées), ce qui peut masquer certains constats ou, à l'inverse, en révéler qui n'existent pas en production.
- **Vulnérabilités liées à l'écosystème plutôt qu'au code lui-même** : une application peut être parfaitement codée mais reste dépendante de la sécurité de son infrastructure d'hébergement, de ses intégrations tierces et de la configuration de son environnement d'exécution — d'où l'intérêt de croiser les constats de ce chapitre avec ceux du chapitre 4 (audit des infrastructures) pour obtenir une vision complète du risque applicatif.

## À retenir

- L'OWASP Top 10 est la référence minimale pour structurer un audit applicatif, mais doit être complété par une analyse spécifique à la logique métier propre à chaque application.
- SAST, DAST et SCA sont complémentaires, pas substituables l'un à l'autre : chacun couvre des angles morts différents, et leur intégration continue au pipeline CI/CD (approche DevSecOps) prolonge l'effet d'un audit ponctuel dans la durée.
- Les défaillances de contrôle d'accès figurent parmi les vulnérabilités les plus fréquentes et les plus faciles à exploiter : elles méritent une attention systématique, y compris sur les fonctionnalités et API les moins visibles.
- L'audit de code bas niveau (C/C++) cible spécifiquement les erreurs de gestion mémoire et de validation des entrées, dont l'impact peut aller du simple plantage à l'exécution de code arbitraire ; des outils dynamiques (Valgrind, AddressSanitizer) et le fuzzing complètent utilement la revue manuelle.
- Un audit applicatif ponctuel reste, par construction, une photographie datée : sa valeur dans la durée dépend de son articulation avec des contrôles automatisés continus.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
