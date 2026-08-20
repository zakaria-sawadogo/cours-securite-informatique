# Chapitre 3 — Principes de conception sécurisée

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/03-principes-conception-securisee.txt){ target=_blank }

## 0. Origine : les travaux de Saltzer et Schroeder

La majorité des principes présentés dans ce chapitre trouvent leur origine dans l'article fondateur de Jerome Saltzer et Michael Schroeder, *The Protection of Information in Computer Systems* (1975), qui a formalisé pour la première fois un ensemble cohérent de principes de conception applicables à tout système informatique cherchant à protéger de l'information. Ce qui frappe, près de cinquante ans plus tard, est la robustesse de ces principes face à l'évolution radicale des technologies : conçus à l'ère des systèmes à temps partagé centralisés, ils s'appliquent sans perte de pertinence aux architectures Cloud distribuées, aux microservices et aux API modernes. Cela s'explique par leur nature : ce ne sont pas des recommandations techniques liées à une technologie donnée, mais des propriétés structurelles générales que tout système de contrôle d'accès doit respecter pour rester robuste.

## 1. Moindre privilège (least privilege)

Chaque composant, utilisateur ou processus ne doit disposer que des droits strictement nécessaires à l'accomplissement de sa tâche, ni plus. Application concrète : un service applicatif se connectant à une base de données ne doit pas utiliser un compte disposant de droits d'administration complets si seules des opérations de lecture/écriture sur des tables spécifiques sont nécessaires. Ce principe limite l'impact d'une compromission (un composant compromis ne peut nuire qu'à hauteur de ses propres privilèges).

### Illustration : compte de service en base de données

```sql
-- À éviter : le service applicatif utilise un compte disposant de tous les droits
-- GRANT ALL PRIVILEGES ON *.* TO 'service_app'@'%';

-- Application du moindre privilège : droits limités aux tables et opérations nécessaires
CREATE USER 'service_app'@'10.0.1.%' IDENTIFIED BY '***';
GRANT SELECT, INSERT, UPDATE ON gestioncours.inscriptions TO 'service_app'@'10.0.1.%';
GRANT SELECT ON gestioncours.etudiants TO 'service_app'@'10.0.1.%';
-- Aucun droit DELETE, DROP, ni accès aux autres bases du serveur
```

Ce même principe s'applique au-delà des bases de données : rôles IAM Cloud limités à un périmètre précis de ressources, jetons d'API à portée (*scope*) restreinte, comptes de service dédiés à une seule fonction plutôt qu'un compte générique partagé par plusieurs composants. Un corollaire souvent négligé du moindre privilège est sa dimension **temporelle** : un privilège élevé accordé de façon permanente pour un besoin ponctuel (par exemple, un accès administrateur donné à un développeur pour une opération de maintenance unique) devrait être révoqué une fois le besoin satisfait, plutôt que laissé actif indéfiniment « au cas où ». C'est l'idée d'accès *juste-à-temps* (just-in-time), reprise et approfondie au chapitre 4 dans le contexte du zero trust.

## 2. Défense en profondeur (defense in depth)

Superposer plusieurs couches de contrôles indépendantes, de sorte que la défaillance d'une seule couche ne compromette pas l'ensemble du système. Exemple pour une application web : pare-feu réseau, pare-feu applicatif (WAF), validation des entrées côté serveur, requêtes préparées contre l'injection SQL, chiffrement des données sensibles au repos, journalisation et détection d'anomalies. Aucune couche seule n'est infaillible ; leur combinaison réduit fortement la probabilité qu'une seule faille suffise à compromettre le système entier.

Une condition essentielle pour que la défense en profondeur soit réellement efficace est l'**indépendance** des couches : si toutes les couches de protection reposent sur la même hypothèse ou la même technologie sous-jacente, une faille unique dans cette hypothèse commune peut les neutraliser simultanément. Par exemple, si le pare-feu réseau, le pare-feu applicatif et la validation des entrées reposent tous implicitement sur la même liste blanche de caractères autorisés mal conçue, une technique de contournement de cette liste peut traverser les trois couches d'un coup. Une défense en profondeur robuste combine des mécanismes de nature différente (réseau, applicatif, cryptographique, organisationnel/procédural) plutôt que la simple répétition du même type de contrôle.

## 3. Échec sécurisé (fail-safe / fail-secure)

En cas d'erreur ou de défaillance, un système doit basculer vers l'état le plus sûr, pas vers un état permissif. Exemple : si un mécanisme de vérification d'autorisation échoue techniquement (exception, timeout), le comportement par défaut doit être de **refuser** l'accès, jamais de l'accorder par défaut. Une erreur de logique inversant ce principe (`if erreur: autoriser_acces()` au lieu de refuser) est une classe de bug de sécurité récurrente.

```python
# Anti-pattern à éviter : en cas d'erreur, l'accès est accordé par défaut
def verifier_acces(utilisateur, ressource):
    try:
        return service_autorisation.est_autorise(utilisateur, ressource)
    except ErreurService:
        return True  # dangereux : une panne du service d'autorisation ouvre l'accès

# Application du principe d'échec sécurisé : en cas d'erreur, l'accès est refusé
def verifier_acces(utilisateur, ressource):
    try:
        return service_autorisation.est_autorise(utilisateur, ressource)
    except ErreurService:
        journaliser_incident("service d'autorisation indisponible")
        return False  # refus par défaut, même au prix d'une indisponibilité fonctionnelle
```

Ce principe illustre un arbitrage fréquent en conception : privilégier la sécurité (refus par défaut) peut dégrader temporairement la disponibilité fonctionnelle du service en cas de panne d'un composant tiers. C'est un compromis assumé : une indisponibilité temporaire, bien que gênante, est presque toujours préférable à un accès non autorisé accordé par erreur, dont les conséquences peuvent être bien plus difficiles à corriger a posteriori (fuite de données, action malveillante déjà réalisée).

## 4. Médiation complète (complete mediation)

Chaque accès à une ressource protégée doit être vérifié systématiquement, à chaque tentative, sans mise en cache non maîtrisée d'une autorisation antérieure qui pourrait être devenue caduque (ex. droit révoqué entre-temps). Une vérification effectuée une seule fois « à l'entrée » puis considérée valide pour toute la suite d'une session peut devenir obsolète si les droits changent en cours de session.

Un cas fréquent de violation de ce principe se rencontre dans des applications qui vérifient les droits d'un utilisateur au moment de l'affichage d'une page ou d'un menu (par exemple, en masquant un bouton « supprimer » pour un utilisateur non autorisé), mais omettent de revérifier ces mêmes droits côté serveur au moment où l'action est effectivement exécutée. Un attaquant peut alors contourner la vérification côté affichage (interface utilisateur) et invoquer directement l'action côté serveur (appel API direct), en l'absence de médiation complète effective à ce niveau. La règle de conception qui en découle est simple à énoncer mais souvent négligée en pratique : **toute vérification d'autorisation effectuée uniquement côté client (navigateur) doit être considérée comme un confort d'expérience utilisateur, jamais comme un contrôle de sécurité** — le contrôle de sécurité réel doit systématiquement être répété côté serveur.

## 5. Séparation des tâches (separation of duties)

Répartir des opérations sensibles entre plusieurs personnes ou composants distincts, de sorte qu'aucune entité unique ne puisse, seule, réaliser une action critique de bout en bout sans contrôle croisé. Exemple organisationnel : la personne qui valide un paiement ne doit pas être la même que celle qui l'initie. Exemple technique : séparer les clés de signature de code des systèmes de build automatisés accessibles à de nombreux développeurs.

Dans un pipeline de développement logiciel moderne, ce principe se traduit par exemple par la séparation entre les droits de proposer une modification de code (ouverture d'une *pull request*) et les droits de l'approuver et de la fusionner dans la branche principale (revue de code obligatoire par une personne distincte de l'auteur), ou par la séparation entre les droits de développement d'une application et les droits d'accès direct à l'environnement de production. Cette séparation ne vise pas seulement à se prémunir contre des actions délibérément malveillantes d'un employé (scénario rare), mais surtout à réduire le risque d'erreur humaine non intentionnelle : un second regard indépendant détecte plus souvent une erreur de configuration ou de logique qu'un contrôle purement automatisé.

## 6. Simplicité de conception (economy of mechanism)

Un système simple est plus facile à analyser, auditer et prouver correct qu'un système complexe. La complexité est l'ennemie de la sécurité : elle multiplie les cas limites, les interactions imprévues entre composants et la surface d'attaque effective. Ce principe justifie de préférer des architectures et des dépendances minimales à des solutions sur-conçues « au cas où ».

Ce principe entre parfois en tension avec d'autres objectifs de conception légitimes (généricité, extensibilité future, performance) : un mécanisme d'autorisation très générique et configurable, capable de gérer un grand nombre de cas particuliers actuels et futurs, est souvent plus difficile à auditer complètement qu'un mécanisme volontairement plus rigide mais dont chaque cas possible peut être énuméré et vérifié. Une conception mature ne maximise pas la généricité par principe, mais recherche le niveau de complexité minimal suffisant pour couvrir les besoins réels et raisonnablement anticipés du système, en résistant à la tentation d'ajouter des options ou des cas particuliers « au cas où un besoin futur hypothétique se présenterait ».

## 7. Conception ouverte (open design) — rappel du principe de Kerckhoffs

Rappel direct du module *Cryptographie* : la sécurité d'un système ne doit pas reposer sur le secret de sa conception, mais sur des mécanismes robustes même connus de l'attaquant (clés, secrets d'authentification), le design lui-même pouvant être examiné publiquement.

Ce principe s'oppose à l'approche dite de « sécurité par l'obscurité » (*security through obscurity*), qui consiste à compter sur le fait qu'un attaquant ignore les détails de fonctionnement interne d'un système pour se protéger (par exemple, un algorithme de chiffrement maison non documenté, ou un format de jeton de session propriétaire supposé difficile à deviner). L'expérience répétée du domaine montre que ce type de protection s'effondre dès qu'un attaquant déterminé investit le temps nécessaire pour analyser le système (rétro-ingénierie), alors qu'un mécanisme conçu pour résister même en connaissance complète de son fonctionnement (algorithmes cryptographiques publics et éprouvés, protocoles standardisés et largement examinés par la communauté) offre une garantie de sécurité bien plus fiable et durable. Cela ne signifie pas qu'il faille publier inutilement des détails d'implémentation sensibles (configuration, adresses internes) : la conception ouverte porte sur les *mécanismes* de sécurité eux-mêmes, pas sur l'ensemble des informations opérationnelles d'un système.

## 8. Minimisation de la surface d'attaque

Réduire le nombre de points d'entrée exposés (fonctionnalités, ports réseau, API, dépendances tierces) au strict nécessaire. Chaque fonctionnalité, endpoint ou dépendance supplémentaire est une surface d'attaque potentielle additionnelle à sécuriser et maintenir — un principe qui rejoint la minimisation de complexité du point précédent.

En pratique, minimiser la surface d'attaque se traduit par des actions concrètes et souvent simples à mettre en œuvre : désactiver les modules ou services non utilisés d'un serveur, fermer les ports réseau non strictement nécessaires, retirer le code mort ou les fonctionnalités de débogage laissées actives en production (endpoints de diagnostic, interfaces d'administration accessibles sans authentification renforcée), et auditer régulièrement les dépendances tierces effectivement utilisées par rapport à celles déclarées. Une pratique de revue périodique de la surface d'attaque exposée (inventaire des points d'entrée réellement accessibles depuis l'extérieur) permet de détecter la dérive progressive d'un système au fil de ses évolutions, phénomène fréquent lorsque des fonctionnalités temporaires ou expérimentales ne sont jamais retirées après leur usage initial.

## 9. Valeurs par défaut sécurisées (secure by default)

Un système livré ou installé doit être sécurisé « dès la sortie de la boîte », sans que l'utilisateur ait à activer explicitement les protections de base (ex. chiffrement activé par défaut, comptes par défaut désactivés, ports non essentiels fermés par défaut). Rappel du chapitre 1 : la sécurité facile et par défaut est plus efficace en pratique que la sécurité techniquement supérieure mais optionnelle.

## 10. Synthèse : comment ces principes se combinent

Ces principes ne s'appliquent pas de façon isolée : ils interagissent et se renforcent mutuellement dans une conception cohérente. Le tableau suivant illustre quelques interactions typiques entre principes :

| Situation de conception | Principes combinés |
|---|---|
| Un service compromis ne peut accéder qu'à un sous-ensemble limité de données, et toute tentative d'accès hors de ce périmètre est bloquée et journalisée | moindre privilège + médiation complète + défense en profondeur |
| Une panne du service d'authentification entraîne un refus d'accès généralisé plutôt qu'un accès non contrôlé, limité aux seules fonctions strictement nécessaires du système en mode dégradé | échec sécurisé + minimisation de la surface d'attaque |
| Une modification de code sensible ne peut être déployée en production qu'après revue par une personne distincte de l'auteur, elle-même sans droit de déploiement direct | séparation des tâches + moindre privilège |

## À retenir

- Ces principes (Saltzer et Schroeder, et compléments ultérieurs) forment un socle de conception réutilisable indépendamment de la technologie employée, car ils décrivent des propriétés structurelles générales plutôt que des recommandations techniques ponctuelles.
- Ils se combinent : la défense en profondeur suppose l'application cohérente du moindre privilège à chaque couche ; l'échec sécurisé suppose une médiation complète ; la séparation des tâches renforce le moindre privilège à l'échelle organisationnelle.
- La médiation complète impose de ne jamais considérer une vérification côté client comme un contrôle de sécurité suffisant : le contrôle réel doit toujours être répété côté serveur.
- La simplicité de conception (economy of mechanism) entre parfois en tension avec la généricité ou l'extensibilité : une conception mature recherche le niveau de complexité minimal suffisant, pas la généricité maximale.
- La conception ouverte rejette la sécurité par l'obscurité au profit de mécanismes robustes même connus de l'attaquant, tout en distinguant cela de la publication inutile d'informations opérationnelles sensibles.
- Aucun principe pris isolément ne suffit ; c'est leur application systématique et cohérente à l'échelle d'un système qui constitue une véritable démarche *security by design*.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
