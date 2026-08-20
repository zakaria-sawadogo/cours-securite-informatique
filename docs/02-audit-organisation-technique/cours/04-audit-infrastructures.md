# Chapitre 4 — Audit des infrastructures réseau et systèmes

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/04-audit-infrastructures.txt){ target=_blank }

## 1. Audit de l'architecture réseau

- **Segmentation** : vérifier l'existence de zones réseau distinctes (DMZ, réseau interne, réseau d'administration) et que le filtrage entre zones suit le principe du moindre privilège.
- **Flux autorisés** : revue des règles de pare-feu (une règle `ANY-ANY` en sortie ou entrée est presque toujours une non-conformité à documenter).
- **Points d'entrée exposés** : recensement de tout ce qui est accessible depuis Internet (VPN, portails, services mal exposés par erreur).

La segmentation réseau est souvent présentée comme une simple bonne pratique, mais elle joue en réalité un rôle de **limitation de la propagation** (*containment*) qui conditionne fortement l'impact réel d'une compromission. Un attaquant qui parvient à compromettre un poste utilisateur dans un réseau plat (sans segmentation) peut potentiellement atteindre directement les serveurs critiques ; dans une architecture correctement segmentée, ce même attaquant doit franchir plusieurs zones de filtrage successives, ce qui augmente le temps et la difficulté de l'attaque, et donc les chances de détection.

Un principe important à vérifier lors d'un audit de segmentation est la **zone d'administration dédiée** : les interfaces d'administration des équipements (pare-feu, serveurs, hyperviseurs) ne devraient jamais être accessibles depuis le réseau utilisateur standard, mais uniquement depuis un réseau d'administration restreint, idéalement avec un accès filtré par une passerelle d'administration (*bastion* ou *jump host*) journalisant chaque connexion.

### Revue des règles de pare-feu : une méthode pas à pas

Une revue de règles de pare-feu suit généralement cette logique :

1. Extraire la configuration complète (export de règles) plutôt que de se fier à une description orale de l'architecture.
2. Identifier les règles trop permissives (`ANY` en source, destination ou port), en particulier celles ouvertes en sortie vers Internet, souvent négligées alors qu'elles facilitent l'exfiltration de données en cas de compromission.
3. Identifier les règles **obsolètes** : flux autorisés pour un projet ou un prestataire qui n'existe plus, souvent jamais nettoyés faute de processus de revue périodique.
4. Vérifier la cohérence entre la documentation d'architecture attendue et les règles réellement appliquées (un écart entre les deux est un indicateur de dérive de configuration, ou *configuration drift*).
5. Tester, lorsque le mandat le permet, l'efficacité réelle du filtrage depuis différentes zones (un scan de ports lancé depuis le réseau utilisateur vers le réseau d'administration doit ne rien retourner s'il est correctement filtré).

```bash
# Vérification empirique du filtrage entre deux zones réseau
nmap -Pn -p 22,3389,443,161 10.10.5.0/24
```

### Piège courant : la confiance excessive dans le pare-feu périmétrique

Une organisation qui a investi dans un pare-feu périmétrique performant peut avoir tendance à négliger la sécurité interne, en considérant que « tout ce qui est à l'intérieur est de confiance ». Cette hypothèse, héritée d'une vision « château fort » de la sécurité, est de moins en moins tenable : la majorité des incidents graves impliquent un point d'entrée initial limité (poste compromis par hameçonnage, identifiants volés) suivi d'un mouvement latéral à l'intérieur du réseau. C'est le fondement du modèle de sécurité **Zero Trust**, qui recommande de ne faire confiance à aucun flux par défaut, y compris interne, et de vérifier systématiquement l'identité et le contexte de chaque connexion — un point qu'un auditeur peut utilement évoquer dans ses recommandations, même s'il dépasse le strict périmètre technique de l'audit.

## 2. Audit des équipements réseau

- Mots de passe par défaut non changés (routeurs, switchs, imprimantes réseau, caméras IP).
- Protocoles non chiffrés encore actifs (Telnet, HTTP d'administration, SNMP v1/v2 avec communauté `public`).
- Firmware obsolète et absence de politique de mise à jour.

Ces trois catégories de constats reviennent avec une régularité frappante dans les audits d'infrastructure, ce qui en fait une checklist minimale à ne jamais omettre :

- **Mots de passe par défaut** : de nombreux équipements réseau et IoT (caméras, imprimantes, objets connectés de supervision) sont livrés avec des identifiants par défaut documentés publiquement par le fabricant. Un audit sérieux inclut systématiquement une tentative de connexion avec les identifiants par défaut connus du modèle identifié (fingerprinting préalable), dans le respect du mandat.
- **Protocoles non chiffrés** : Telnet et les interfaces HTTP d'administration transmettent les identifiants en clair, interceptables par tout attaquant en position d'écoute sur le même segment réseau (attaque de type *ARP spoofing* suivie d'une capture de trafic). SNMP en version 1 ou 2c repose sur une « communauté » (chaîne d'authentification) souvent laissée à la valeur par défaut `public` en lecture et parfois `private` en écriture — cette dernière permettant potentiellement de modifier la configuration de l'équipement.
- **Firmware obsolète** : contrairement aux systèmes d'exploitation serveurs, les équipements réseau et IoT font souvent l'objet d'une politique de mise à jour beaucoup moins rigoureuse, alors qu'ils sont directement exposés au trafic réseau. L'auditeur doit vérifier l'existence d'un inventaire des équipements avec leur version de firmware et la fréquence de vérification des mises à jour de sécurité disponibles.

```bash
# Identification de version SNMP et test de la communauté par défaut (avec autorisation explicite)
snmpwalk -v2c -c public 192.168.1.1 system
```

### Bonne pratique : l'inventaire des actifs comme prérequis

Un audit d'infrastructure ne peut être exhaustif que si l'organisation dispose d'un **inventaire à jour** de ses équipements réseau. En son absence, l'auditeur doit reconstruire cet inventaire par la découverte active (scan réseau), ce qui prend du temps et ne garantit jamais une couverture à 100 % (un équipement éteint pendant l'audit, ou volontairement dissimulé du fait d'un usage non autorisé — phénomène de « shadow IT » — peut échapper à la découverte). Recommander la mise en place et la maintenance d'un inventaire d'actifs figure généralement, à juste titre, parmi les toutes premières recommandations d'un rapport d'audit d'infrastructure, car cet inventaire conditionne l'efficacité de tous les contrôles ultérieurs (gestion des correctifs, détection d'intrusion, réponse à incident).

## 3. Audit des systèmes d'exploitation (hardening)

Points de contrôle typiques, alignés sur les référentiels de durcissement (CIS Benchmarks) :

| Domaine | Points de contrôle |
|---|---|
| Comptes et authentification | comptes par défaut désactivés, politique de mots de passe, MFA pour les comptes à privilèges |
| Services | services inutiles désactivés (réduction de la surface d'attaque) |
| Correctifs | politique de patch management, délai moyen d'application des correctifs critiques |
| Journalisation | logs activés, centralisés, conservés une durée suffisante |
| Droits | principe du moindre privilège, séparation des comptes admin/utilisateur |

Les **CIS Benchmarks** (publiés par le Center for Internet Security) sont des guides de configuration très détaillés, déclinés par système d'exploitation et par version (Windows Server, distributions Linux courantes, équipements réseau, bases de données, conteneurs). Chaque point de contrôle y est présenté avec une justification technique, une procédure de vérification et une procédure de remédiation, ce qui en fait un outil particulièrement utile pour un auditeur : plutôt que de réinventer une checklist, il est possible de s'appuyer directement sur ces benchmarks et, pour les systèmes les plus courants, sur des outils d'évaluation automatisée qui en vérifient l'application (scanners de conformité de configuration).

Quelques points d'attention à approfondir par domaine :

- **Comptes et authentification** : au-delà de la simple existence d'une politique de mots de passe, l'auditeur doit vérifier sa robustesse réelle (longueur minimale, complexité, historique empêchant la réutilisation) et surtout l'application effective du MFA (authentification multifacteur) sur les comptes à privilèges — un point de contrôle devenu quasi incontournable dans les référentiels récents, tant les identifiants volés ou devinés restent un vecteur d'attaque initial fréquent.
- **Services** : chaque service actif est une surface d'attaque potentielle. La désactivation des services inutiles (principe de réduction de la surface d'attaque, ou *attack surface reduction*) est une mesure simple, peu coûteuse et à fort impact, souvent négligée par manque de temps lors du déploiement initial des systèmes.
- **Correctifs** : au-delà de l'existence d'une politique, l'indicateur le plus révélateur est le **délai moyen d'application** des correctifs critiques (*Mean Time To Patch*) ; un audit peut utilement échantillonner un ensemble de serveurs et comparer leur niveau de correctifs à la date de publication des correctifs de sécurité correspondants.
- **Journalisation** : des journaux existent souvent localement, mais leur valeur pour la détection et l'investigation dépend de leur **centralisation** (SIEM ou solution équivalente) et de leur durée de conservation, qui doit être compatible avec le délai moyen de détection d'une intrusion (souvent de plusieurs semaines à plusieurs mois lorsque l'attaquant reste discret) et avec les obligations réglementaires applicables.
- **Droits** : le principe du moindre privilège s'applique autant aux comptes humains qu'aux comptes de service utilisés par les applications ; un compte de service disposant de droits d'administrateur du domaine pour exécuter une tâche planifiée simple est un constat classique et à fort impact.

### Exemple de commande d'audit local (Linux)

```bash
# Lister les comptes disposant d'un shell interactif (candidats à vérifier)
awk -F: '$7 ~ /(bash|sh)$/ {print $1}' /etc/passwd

# Vérifier les services en écoute sur le système
ss -tulnp

# Vérifier la configuration SSH (points fréquemment non conformes)
grep -Ei 'PermitRootLogin|PasswordAuthentication|Protocol' /etc/ssh/sshd_config
```

Ces trois commandes illustrent une démarche simple d'audit local : identifier les comptes exploitables interactivement, cartographier les services réellement exposés, et vérifier des paramètres de configuration critiques (connexion root directe en SSH, authentification par mot de passe plutôt que par clé) qui figurent systématiquement dans les CIS Benchmarks correspondants.

## 4. Cas particulier : audit d'un domaine Active Directory

L'annuaire Active Directory est une cible privilégiée car il centralise l'authentification. Points d'audit fréquents :

- comptes avec mot de passe n'expirant jamais ou marqués `PASSWD_NOTREQD` ;
- délégation Kerberos non contrainte mal maîtrisée (risque d'usurpation) ;
- groupes à privilèges (`Domain Admins`) surdimensionnés ;
- absence de séparation entre comptes d'administration et comptes utilisateurs quotidiens ;
- outils dédiés d'audit : `BloodHound` (cartographie des chemins d'attaque), `PingCastle` (score de maturité AD).

Active Directory mérite ce traitement particulier car il concentre, dans la quasi-totalité des environnements d'entreprise sous Windows, un rôle d'infrastructure critique disproportionné par rapport à l'attention qu'il reçoit souvent lors des audits généralistes. Une poignée de mauvaises configurations, prises isolément anodines, peuvent se combiner en un **chemin d'attaque** menant directement à la compromission de l'ensemble du domaine (*Domain Admin*) — c'est précisément ce que des outils comme `BloodHound` permettent de visualiser, en modélisant l'annuaire sous forme de graphe de relations (appartenance à des groupes, droits délégués, sessions actives) et en identifiant automatiquement les chemins les plus courts vers les comptes à privilèges les plus élevés.

Quelques constats classiques à approfondir :

- **Kerberoasting** : tout compte de service disposant d'un *Service Principal Name* (SPN) est susceptible de voir son ticket Kerberos extrait par un utilisateur authentifié standard, puis soumis à une attaque hors ligne par force brute si son mot de passe est faible — d'où l'importance de mots de passe longs et aléatoires pour les comptes de service, idéalement gérés par un coffre-fort de secrets.
- **Délégation Kerberos non contrainte** : un compte configuré avec ce type de délégation peut, dans certaines conditions, capturer et réutiliser les tickets d'authentification d'autres utilisateurs se connectant à lui, ce qui en fait une cible de choix si un attaquant en obtient le contrôle.
- **Groupes à privilèges surdimensionnés** : le groupe `Domain Admins` ne devrait contenir qu'un nombre restreint de comptes dédiés à l'administration, utilisés ponctuellement, et jamais de comptes utilisateurs du quotidien (consultation de messagerie, navigation web) — la pratique consistant à donner un compte unique « tout puissant » à chaque administrateur reste malheureusement fréquente et constitue un facteur aggravant majeur en cas de compromission d'un seul poste de travail.

### Bonne pratique : le modèle en tiers (tiering)

Microsoft recommande un modèle d'administration en **tiers** (*tier model*), qui sépare strictement les comptes et postes d'administration selon le niveau de criticité des ressources administrées (contrôleurs de domaine en tier 0, serveurs en tier 1, postes utilisateurs en tier 2), avec l'interdiction pour un compte d'un tier donné de s'authentifier sur une ressource d'un tier plus sensible. Vérifier l'existence, même partielle, d'une telle séparation est un point de contrôle avancé mais particulièrement révélateur de la maturité d'une organisation en matière de gestion des identités et des accès.

## 5. Audit du Cloud et des environnements virtualisés

- Vérification des configurations de stockage (buckets S3 ou équivalents en accès public non justifié).
- Gestion des identités et accès (IAM) : clés d'API non tournées, permissions trop larges.
- Isolation entre locataires (tenants) dans un environnement mutualisé.
- Journalisation et alerte (CloudTrail ou équivalent) activées et supervisées.

L'audit d'environnements Cloud introduit une spécificité importante : le **modèle de responsabilité partagée**. Selon le niveau de service considéré (IaaS, PaaS, SaaS), la répartition des responsabilités de sécurité entre le fournisseur Cloud et le client change fortement. Sur une infrastructure IaaS, le fournisseur sécurise la couche physique et la virtualisation, mais la configuration du système d'exploitation, du réseau virtuel et des applications reste de la responsabilité du client ; sur un service SaaS, le fournisseur gère l'essentiel de la pile technique, et la responsabilité du client se concentre sur la configuration des accès et des paramètres applicatifs. Un audit Cloud doit toujours commencer par clarifier ce partage de responsabilité, faute de quoi l'auditeur risque soit d'auditer des éléments hors du contrôle du client, soit d'en oublier qui sont bien de sa responsabilité.

Quelques approfondissements sur les points de contrôle listés ci-dessus :

- **Stockage en accès public non justifié** : de nombreuses fuites de données documentées publiquement dans l'industrie proviennent d'espaces de stockage objet (type S3) configurés par erreur en accès public, souvent lors d'un déploiement rapide ou d'un test jamais reconfiguré ensuite. La vérification de ce point ne nécessite généralement aucune exploitation active : une simple tentative de lecture non authentifiée suffit à démontrer le constat.
- **Gestion des identités et accès (IAM)** : le Cloud facilite la création rapide de clés d'API et de rôles, mais aussi leur prolifération incontrôlée. Un audit IAM vérifie notamment l'absence de clés d'accès à privilèges larges (`*` en action et en ressource) attribuées à des services qui n'en ont pas besoin, la rotation régulière des clés statiques, et la préférence donnée aux rôles temporaires plutôt qu'aux identifiants permanents.
- **Isolation entre locataires** : dans un environnement mutualisé, un défaut d'isolation peut permettre à un client d'accéder, même involontairement, aux ressources d'un autre client du même fournisseur — un point difficile à auditer directement sans la coopération du fournisseur, mais qui doit au minimum faire l'objet d'une vérification contractuelle et documentaire (certifications du fournisseur, rapports d'audit tiers type SOC 2).
- **Journalisation et alerte** : contrairement à une infrastructure classique où les journaux doivent être configurés service par service, les fournisseurs Cloud proposent généralement un service de journalisation centralisé des actions d'administration (traçant qui a fait quoi, quand, depuis où) ; un constat fréquent est que ce service, bien que disponible, n'est simplement pas activé, ou activé sans alerte associée en cas d'action sensible (suppression d'un journal, modification d'une politique de sécurité).

## 6. Restitution d'un audit d'infrastructure

Chaque constat technique doit être formulé selon la structure : **observation** (fait constaté et preuve), **risque** (ce que cela permet à un attaquant), **recommandation** (action corrective priorisée), **référence** (CIS Benchmark, ISO 27002, CVE le cas échéant).

Cette structure en quatre temps mérite d'être illustrée par un exemple complet, transposable à n'importe quel constat de ce chapitre :

> **Observation** : le service SNMP du commutateur cœur de réseau (adresse 192.168.1.1) répond en version 2c avec la communauté par défaut `public` en lecture, comme démontré par la commande `snmpwalk` exécutée le [date].
> **Risque** : un attaquant en position d'accès au réseau interne peut, sans authentification robuste, extraire des informations de configuration détaillées de l'équipement (table de routage, interfaces, parfois mots de passe faiblement protégés selon le modèle), facilitant la cartographie de l'infrastructure en vue d'une attaque ultérieure.
> **Recommandation** : désactiver SNMP v1/v2c au profit de SNMP v3 (authentification et chiffrement), ou à défaut remplacer la communauté par défaut par une valeur forte et restreindre l'accès SNMP par liste de contrôle d'accès (ACL) au seul poste de supervision légitime.
> **Référence** : CIS Benchmark pour équipements réseau ; ISO/IEC 27002, thème technologique — sécurité des réseaux.

Cette structure systématique facilite non seulement la lecture du rapport, mais aussi son exploitation ultérieure par les équipes techniques chargées de la remédiation, qui peuvent directement transformer chaque constat en tâche opérationnelle (voir chapitre 6).

## À retenir

- L'audit d'infrastructure combine architecture réseau, configuration système et, de plus en plus, environnements Cloud/AD ; la segmentation réseau reste un mécanisme fondamental de limitation de la propagation en cas de compromission.
- Les référentiels de durcissement (CIS Benchmarks) fournissent des points de contrôle réutilisables et mesurables, applicables aussi bien aux systèmes d'exploitation qu'aux équipements réseau, bases de données ou conteneurs.
- Active Directory concentre un rôle critique disproportionné : un audit dédié, appuyé sur des outils comme BloodHound ou PingCastle, permet d'identifier des chemins d'attaque invisibles au niveau de constats isolés.
- L'audit Cloud impose de clarifier au préalable le modèle de responsabilité partagée, faute de quoi l'auditeur risque d'évaluer des éléments hors du contrôle réel du client ou d'en oublier qui restent de sa responsabilité.
- Chaque constat doit relier un fait technique à un risque métier compréhensible par la direction, en suivant systématiquement la structure observation / risque / recommandation / référence.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
