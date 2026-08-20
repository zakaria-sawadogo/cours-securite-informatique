# Chapitre 3 — Audit technique : méthodologie des tests d'intrusion

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/03-audit-technique-methodologie.txt){ target=_blank }

## 1. Le PTES (Penetration Testing Execution Standard)

Le PTES structure un test d'intrusion en sept phases :

1. **Pre-engagement** : cadrage, périmètre, mandat, règles d'engagement.
2. **Intelligence gathering** (reconnaissance) : collecte passive et active d'informations.
3. **Threat modeling** : identification des actifs critiques et des scénarios d'attaque plausibles.
4. **Vulnerability analysis** : identification des failles (scan, analyse manuelle).
5. **Exploitation** : tentative d'exploitation contrôlée des vulnérabilités identifiées.
6. **Post-exploitation** : évaluation de l'impact réel (élévation de privilèges, pivot, exfiltration simulée).
7. **Reporting** : rédaction du rapport.

Le PTES n'est pas la seule méthodologie de référence en test d'intrusion, mais c'est l'une des plus complètes et des plus pédagogiques pour un cours d'introduction, car elle explicite des phases souvent négligées par les praticiens pressés — en particulier le *threat modeling* et le *pre-engagement*. D'autres cadres méthodologiques existent et sont utiles à connaître :

- **OSSTMM** (Open Source Security Testing Methodology Manual) : très orienté mesure quantitative de la sécurité (notion de *RAV*, *Risk Assessment Value*).
- **OWASP Testing Guide** : spécifique aux applications web, largement mobilisé au chapitre 5 de ce module.
- **NIST SP 800-115** : guide technique publié par le NIST, souvent cité en référence dans les cahiers des charges d'audit publics.

Ces méthodologies partagent une structure globale similaire (cadrage, reconnaissance, analyse, exploitation, restitution) : l'essentiel n'est pas de mémoriser un vocabulaire propre à chacune, mais de comprendre la logique commune qui les sous-tend et de savoir l'adapter au contexte de la mission.

### Le threat modeling, une phase souvent négligée

La modélisation des menaces (phase 3) consiste à se poser, avant même de lancer le premier scan, la question : « si j'étais un attaquant motivé, que chercherais-je à atteindre dans ce système, et par quel chemin le plus probable ? » Cette réflexion évite à l'auditeur de disperser son temps limité sur des cibles secondaires et l'aide à prioriser : un serveur de test isolé du réseau de production mérite moins d'attention qu'un serveur d'authentification centralisé, même si le second présente, en apparence, moins de vulnérabilités techniques immédiates. Des approches structurées comme **STRIDE** (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege) peuvent être mobilisées pour systématiser cette réflexion, notamment lors d'un audit d'architecture ou de conception.

## 2. Reconnaissance passive vs active

| Type | Exemples de techniques | Détectable par la cible ? |
|---|---|---|
| Passive | OSINT (WHOIS, réseaux sociaux, moteurs de recherche, `theHarvester`), consultation de code source public | non |
| Active | scan de ports, résolution DNS directe, requêtes HTTP vers la cible | oui, potentiellement journalisé |

La reconnaissance passive mérite d'être développée car elle est souvent sous-estimée par les débutants, pressés d'en venir au scan technique. Une bonne reconnaissance passive peut révéler, sans jamais émettre le moindre paquet vers la cible :

- l'architecture technique probable (offres d'emploi mentionnant des technologies précises, dépôts de code publics laissés ouverts par erreur) ;
- des informations sur les collaborateurs (utile pour évaluer le risque d'ingénierie sociale et pour construire des listes de noms d'utilisateurs plausibles) ;
- des sous-domaines et adresses IP historiques (certificats TLS archivés dans des journaux de transparence, archives de pages web) ;
- des fuites de données antérieures potentiellement réutilisables (mots de passe, adresses e-mail).

```bash
# Exemples d'outils et de commandes de reconnaissance passive
whois exemple-pedagogique.tld
theHarvester -d exemple-pedagogique.tld -b google,bing
dig exemple-pedagogique.tld ANY
```

La reconnaissance active, à l'inverse, laisse des traces dans les journaux de la cible (pare-feu, IDS/IPS, journaux applicatifs) : c'est un point important à documenter dans le rapport (les événements générés par l'audit peuvent ensuite être comparés aux alertes effectivement déclenchées côté client, ce qui donne une indication indirecte sur les capacités de détection de l'organisation — même si un test d'intrusion n'est pas conçu, en soi, pour évaluer la détection de façon rigoureuse, contrairement à un exercice Red Team).

### Piège courant : négliger la reconnaissance pour « aller vite »

Un auditeur pressé qui saute directement au scan de vulnérabilités automatisé prend le risque de manquer des informations qu'aucun scanner ne trouvera seul (identifiants exposés par erreur dans un dépôt public, sous-domaine oublié hébergeant une ancienne version vulnérable de l'application). L'expérience montre qu'une reconnaissance soignée, même chronophage, est souvent l'étape la plus rentable en termes de vulnérabilités critiques découvertes par heure investie.

## 3. Scan et énumération

- **Scan de ports** (`nmap`) : identifier les services exposés (ports ouverts, fermés, filtrés).
- **Fingerprinting** : identifier la version des services et systèmes (`nmap -sV -O`).
- **Énumération** : lister les ressources exposées (partages réseau, utilisateurs, répertoires web).

```bash
nmap -sV -sC -p- 192.168.1.10
```

Cette commande mérite d'être décomposée, car chaque option a un rôle précis dans une démarche d'audit :

- `-sV` : détection de version des services (utile pour ensuite confronter chaque version à une base de CVE) ;
- `-sC` : exécution des scripts par défaut de la bibliothèque NSE (*Nmap Scripting Engine*), qui réalisent des vérifications complémentaires (bannières, certificats TLS, configurations faibles connues) ;
- `-p-` : balaie l'intégralité des 65 535 ports TCP, plutôt que la liste restreinte des ports les plus courants scannée par défaut — indispensable en audit, car un service critique mal exposé sur un port non standard est un constat fréquent et significatif.

D'autres options sont fréquemment utiles selon le contexte de la mission :

```bash
# Scan UDP (souvent négligé, plus lent, mais révèle des services comme SNMP, DNS, NTP)
nmap -sU --top-ports 100 192.168.1.10

# Scan discret (moins susceptible de déclencher une alerte, mais plus lent)
nmap -sS -T2 192.168.1.10

# Scan de vulnérabilités via scripts NSE dédiés
nmap --script vuln 192.168.1.10
```

L'énumération, une fois les services identifiés, cherche à en extraire un maximum d'informations exploitables : liste des partages réseau accessibles (SMB), utilisateurs valides sur un service d'authentification, répertoires et fichiers accessibles sur un serveur web (`gobuster`, `ffuf`, `dirb`). Une bonne pratique méthodologique consiste à documenter systématiquement, service par service, ce qui a été identifié — même en l'absence de vulnérabilité immédiate — car cette cartographie servira de base à l'ensemble des phases suivantes et devra figurer, sous une forme synthétique, dans les annexes du rapport final.

## 4. Analyse de vulnérabilités

Outils automatisés (scanners) confrontent les versions de services identifiées à des bases de vulnérabilités connues (CVE) :

- **Nessus**, **OpenVAS** : scanners généralistes réseau/systèmes.
- **Nikto**, **OWASP ZAP**, **Burp Suite** : scanners orientés applications web.

Un scan automatisé produit des **faux positifs** (vulnérabilité signalée mais non exploitable en pratique) et peut manquer des **faux négatifs** (vulnérabilités logiques non détectables par signature) : il ne remplace jamais l'analyse manuelle d'un auditeur.

Comprendre l'origine de ces faux positifs et faux négatifs aide à calibrer la confiance à accorder à un rapport de scan brut :

- Un **faux positif** survient typiquement lorsque le scanner identifie une version de logiciel signalée comme vulnérable dans sa base de signatures, sans tenir compte d'un correctif appliqué manuellement (rétroportage de correctif, *backport*) qui ne modifie pas le numéro de version affiché — pratique courante sur certaines distributions Linux d'entreprise.
- Un **faux négatif** peut résulter d'une vulnérabilité logique propre à l'application (par exemple, un contrôle d'accès mal conçu permettant à un utilisateur d'accéder aux données d'un autre en modifiant un identifiant dans l'URL) : ce type de faille ne correspond à aucune signature connue et ne peut être détecté que par une analyse manuelle ou une revue de code.

Il est donc essentiel de **trier manuellement** les résultats bruts d'un scanner avant de les inclure dans un rapport : présenter à un client une liste de 200 alertes automatisées non vérifiées, dont une part importante de faux positifs, nuit à la crédibilité de l'auditeur et dilue les constats réellement critiques.

### Exemple d'utilisation d'OpenSSL pour vérifier une configuration TLS

Au-delà des scanners dédiés, des outils standards permettent de vérifier manuellement des points de configuration précis, ce qui est utile pour confirmer ou infirmer une alerte automatisée :

```bash
# Vérifier les protocoles et suites de chiffrement supportés par un serveur
openssl s_client -connect exemple-pedagogique.tld:443 -tls1_2

# Afficher les informations du certificat présenté (validité, émetteur, algorithme)
echo | openssl s_client -connect exemple-pedagogique.tld:443 2>/dev/null | openssl x509 -noout -dates -issuer -subject
```

Un constat classique d'audit technique consiste, par exemple, à identifier la persistance de protocoles obsolètes comme SSLv3 ou TLS 1.0 encore acceptés par un serveur, ou un certificat expiré ou auto-signé sur un service exposé publiquement — deux non-conformités simples à démontrer avec ce type de commande, et faciles à faire comprendre dans le rapport, y compris à un lecteur non technique.

## 5. Exploitation contrôlée

L'exploitation vise à démontrer l'impact réel d'une vulnérabilité (preuve de concept), sans dégrader le service ni les données de production. Principes :

- privilégier les preuves non destructives (ex. lecture d'un fichier témoin plutôt que suppression) ;
- documenter précisément chaque action entreprise (horodatage, commande, résultat) pour la traçabilité et le rapport ;
- s'arrêter au périmètre autorisé par le mandat, même si une extension semble techniquement possible.

L'exploitation est souvent la phase la plus valorisée par les apprenants, mais elle doit rester subordonnée aux règles d'engagement définies en amont. Quelques précisions pratiques :

- **Preuve minimale suffisante** : pour une injection SQL, par exemple, il suffit généralement de démontrer l'extraction d'une donnée non sensible (numéro de version de la base de données, nom d'une table) plutôt que d'extraire l'intégralité d'une table contenant des données à caractère personnel — le principe de minimisation s'applique aussi à l'auditeur.
- **Environnement de test dédié** : lorsque le mandat le permet, privilégier un environnement de pré-production ou de test plutôt que la production pour les tentatives d'exploitation les plus risquées (élévation de privilèges, déni de service partiel).
- **Journal d'exploitation (log book)** : tenir, tout au long de la mission, un journal chronologique précis de chaque action technique (commande exécutée, horodatage, résultat, capture d'écran) : ce journal est à la fois une preuve pour le rapport final et une protection pour l'auditeur en cas de contestation ultérieure sur le déroulement de la mission.
- **Communication en cas d'incident imprévu** : si une action, même prudente, provoque un effet inattendu sur la production (service dégradé, alerte de sécurité déclenchée chez le client), l'auditeur doit immédiatement contacter le point de contact prévu au mandat plutôt que de tenter de dissimuler ou corriger seul l'incident.

### Piège courant : la surenchère d'exploitation

Un piège fréquent chez les auditeurs juniors est de chercher à « aller le plus loin possible » pour impressionner le commanditaire, au risque de dépasser le périmètre autorisé ou de causer un incident évitable. La valeur d'un audit ne se mesure pas au nombre de systèmes compromis, mais à la pertinence, la clarté et l'actionnabilité des constats produits. Une preuve de concept élégante et non destructive vaut toujours mieux qu'une compromission complète non maîtrisée.

## 6. Notation de la criticité (CVSS)

Le score **CVSS** (Common Vulnerability Scoring System) permet de qualifier objectivement la gravité d'une vulnérabilité à partir de métriques (vecteur d'attaque, complexité, privilèges requis, impact sur confidentialité/intégrité/disponibilité). Un score CVSS de 9.8 sur 10 (ex. exécution de code à distance sans authentification) impose une remédiation immédiate ; un score de 3.1 peut être traité dans un cycle de correctifs standard.

Le CVSS (dans ses versions 3.1 et 4.0, la version 4.0 affinant notamment la prise en compte du contexte d'exploitation) repose principalement sur un jeu de métriques dites de base (*Base metrics*), calculées à partir de caractéristiques intrinsèques de la vulnérabilité :

| Métrique | Ce qu'elle mesure |
|---|---|
| Vecteur d'attaque (AV) | Depuis où l'attaque est-elle possible : réseau, adjacent, local, physique ? |
| Complexité d'attaque (AC) | Le succès de l'attaque dépend-il de conditions difficiles à réunir ? |
| Privilèges requis (PR) | Faut-il déjà disposer d'un compte, et avec quel niveau de privilège ? |
| Interaction utilisateur (UI) | Une action de la victime est-elle nécessaire (ouvrir un lien, un fichier) ? |
| Impact sur la confidentialité (C) | Fuite de données possible et étendue |
| Impact sur l'intégrité (I) | Modification non autorisée de données ou de systèmes |
| Impact sur la disponibilité (A) | Déni de service possible et étendue |

Ces métriques de base ne racontent toutefois pas toute l'histoire : le CVSS prévoit également des métriques temporelles (existence d'un exploit public, disponibilité d'un correctif) et environnementales (contexte spécifique du système audité), trop rarement renseignées en pratique alors qu'elles sont précisément ce qui permet d'adapter le score générique à la réalité de l'organisation auditée. C'est un point que le chapitre 6 développera : un score CVSS brut ne doit jamais être utilisé seul pour prioriser une remédiation sans contextualisation métier.

## 7. Limites et complémentarité avec l'audit organisationnel

Un test d'intrusion technique donne une photographie à un instant *t* d'un périmètre limité. Il ne dit rien sur la capacité de l'organisation à détecter et répondre à une attaque réelle (à ne pas confondre avec un exercice de type *Red Team*, plus large et plus long, qui teste aussi la détection).

Cette limite mérite d'être explicitée davantage, car elle conditionne le choix du bon type de mission selon l'objectif recherché :

- Un **test d'intrusion classique** vise l'exhaustivité de la découverte de vulnérabilités sur un périmètre donné, dans un temps contraint, généralement avec la connaissance (au moins partielle) des équipes de défense.
- Un exercice **Red Team** vise à tester, sur une durée plus longue, la capacité réelle de détection et de réaction des équipes de défense (*Blue Team*), sans que celles-ci soient nécessairement informées à l'avance ; l'objectif n'est plus de trouver un maximum de vulnérabilités mais d'atteindre un objectif précis (exfiltration simulée d'une donnée cible, par exemple) par le chemin le plus discret possible.
- Un exercice **Purple Team** organise une collaboration active entre attaquants et défenseurs pendant le test, dans un objectif pédagogique et d'amélioration continue plutôt que d'évaluation à l'aveugle.

Par ailleurs, un test d'intrusion technique ne remplace pas un audit organisationnel : une organisation peut réussir un test d'intrusion (aucune vulnérabilité critique exploitable trouvée sur le périmètre testé) tout en présentant des lacunes organisationnelles majeures (absence de plan de réponse à incident, absence de sensibilisation du personnel) qui la rendraient vulnérable à un vecteur non couvert par le test, en particulier l'ingénierie sociale. C'est pourquoi les deux chapitres précédents et les suivants de ce module doivent être lus comme complémentaires plutôt que substituables.

## À retenir

- Le PTES structure la démarche en sept phases, de la reconnaissance au rapport ; d'autres méthodologies (OSSTMM, OWASP Testing Guide, NIST SP 800-115) partagent une logique globale similaire.
- La reconnaissance, souvent négligée, est fréquemment l'étape la plus rentable en vulnérabilités critiques découvertes par heure investie ; elle doit toujours précéder le scan automatisé.
- L'automatisation (scanners) accélère l'analyse mais ne remplace pas le jugement de l'auditeur : chaque alerte automatisée doit être vérifiée manuellement avant d'être incluse dans un rapport.
- L'exploitation doit rester strictement subordonnée au mandat et privilégier des preuves non destructives, documentées avec précision (journal d'exploitation).
- Le score CVSS objective la priorisation des vulnérabilités identifiées, mais ses métriques temporelles et environnementales, souvent négligées, sont indispensables pour l'adapter au contexte réel de l'organisation.
- Un test d'intrusion classique, un exercice Red Team et un audit organisationnel répondent à des objectifs différents et complémentaires, à ne jamais confondre.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
