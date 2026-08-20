# Chapitre 5 — Sécurité et vulnérabilités des smart contracts

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/05-securite-smart-contracts.txt){ target=_blank }

## 1. Pourquoi la sécurité des smart contracts est particulièrement critique

Un smart contract déployé sur une blockchain publique est, par construction, **immuable et son code est public** : contrairement à une application classique, il n'est généralement pas possible de corriger une faille par un simple correctif déployé discrètement — le code vulnérable reste actif et visible de tous jusqu'à une migration complexe. De plus, les contrats manipulent souvent directement des fonds réels, ce qui en fait des cibles à forte valeur immédiate pour un attaquant.

Cette combinaison de facteurs — code public, immutabilité, valeur financière directe, exécution automatique sans intervention humaine possible pour interrompre une transaction en cours — distingue nettement le modèle de menace des smart contracts de celui d'une application web classique étudiée dans les modules précédents de ce cours. Il est utile de comparer explicitement ces deux contextes :

| Critère | Application web classique | Smart contract déployé |
|---|---|---|
| Visibilité du code source | généralement non public (code compilé côté serveur) | bytecode public par construction ; code source souvent publié volontairement pour la transparence |
| Correction après découverte d'une faille | correctif déployé rapidement, souvent de façon transparente pour l'utilisateur | migration complexe, souvent impossible sans mécanisme d'évolutivité prévu à l'avance |
| Réversibilité d'une transaction frauduleuse | souvent possible (annulation, remboursement, intervention d'un administrateur) | généralement impossible une fois la transaction confirmée (chapitre 2) |
| Fenêtre d'exploitation | limitée par les capacités de détection et réaction de l'équipe opérationnelle | potentiellement illimitée tant que le contrat reste actif et non corrigé |

Cette comparaison justifie pourquoi la démarche d'audit **avant** déploiement (section 7 de ce chapitre) occupe une place centrale dans la sécurité des smart contracts, alors qu'une application web classique peut davantage s'appuyer sur une détection et une réaction rapides après incident (modules *Détection d'intrusion* et *Réponse aux incidents*).

### Mécanismes d'évolutivité et leurs propres risques

Face à la rigidité de l'immutabilité, plusieurs patrons de conception permettent de prévoir, dès le déploiement initial, une possibilité de mise à jour ultérieure du code — au prix d'une complexité et d'une surface d'attaque supplémentaires :

- **contrats proxy (délégation)** : un contrat « proxy », dont l'adresse est celle communiquée aux utilisateurs, délègue l'exécution de sa logique à un second contrat d'implémentation dont l'adresse peut être changée par un administrateur autorisé ; cela permet de faire évoluer la logique métier sans changer l'adresse publique du contrat, mais concentre un pouvoir important (et donc un risque) entre les mains de qui contrôle le changement d'implémentation ;
- **pause d'urgence (circuit breaker)** : une fonction, protégée par un contrôle d'accès strict, permettant de suspendre temporairement certaines opérations sensibles du contrat en cas de détection d'une attaque en cours, limitant les pertes le temps d'une analyse approfondie ;
- **limites de montant et listes blanches** : contraindre dès la conception les montants manipulables ou les adresses autorisées à interagir avec les fonctions les plus sensibles, réduisant l'impact maximal d'une faille non encore découverte.

Chacun de ces mécanismes introduit lui-même une nouvelle surface d'attaque potentielle (par exemple, un contrôle d'accès défaillant sur la fonction de changement d'implémentation d'un proxy annule tout l'intérêt du mécanisme) — un rappel que la sécurité d'un système complexe se raisonne toujours de façon globale, jamais mécanisme par mécanisme isolément.

## 2. Réentrance (reentrancy)

Vulnérabilité historique majeure (à l'origine du piratage « The DAO » en 2016, environ 60 millions de dollars de l'époque). Principe : un contrat appelle un contrat externe (ex. pour transférer des fonds) **avant** d'avoir mis à jour son propre état interne ; le contrat externe malveillant peut alors rappeler la fonction d'origine de façon récursive, exploitant le fait que l'état (le solde) n'a pas encore été mis à jour.

```solidity
// Version vulnérable
function retirer(uint256 montant) external {
    require(soldes[msg.sender] >= montant, "Solde insuffisant");
    payable(msg.sender).transfer(montant); // appel externe AVANT la mise à jour de l'état
    soldes[msg.sender] -= montant;          // trop tard : un contrat malveillant a pu rappeler retirer() entre-temps
}
```

**Contre-mesure** : appliquer le patron **checks-effects-interactions** (vérifications, puis mise à jour de l'état interne, puis seulement interaction externe) — c'est exactement l'ordre correct déjà utilisé dans l'exemple du chapitre 4.

```solidity
// Version corrigée, appliquant le patron checks-effects-interactions
function retirer(uint256 montant) external {
    require(soldes[msg.sender] >= montant, "Solde insuffisant"); // 1. vérifications (checks)
    soldes[msg.sender] -= montant;                                // 2. mise à jour de l'état (effects)
    payable(msg.sender).transfer(montant);                        // 3. interaction externe, en dernier (interactions)
}
```

### Pourquoi `transfer` ne suffit pas toujours : le rôle du verrou de réentrance

Historiquement, une contre-mesure complémentaire consistait à utiliser la fonction `transfer` (plutôt que `call`) pour envoyer des fonds, car elle limite le gas transmis à l'appel externe (2300 unités à l'époque), rendant impossible l'exécution d'une logique complexe — dont un rappel récursif — dans le contrat destinataire. Cette contre-mesure est aujourd'hui considérée comme fragile et insuffisante à elle seule (le coût de certaines opérations EVM a évolué, et de nombreux contrats destinataires légitimes, comme les portefeuilles multi-signatures, ont eux-mêmes besoin de plus de gas que cette limite pour fonctionner correctement) ; le patron checks-effects-interactions reste la contre-mesure de référence, indépendante des variations de coût du gas. Une seconde contre-mesure complémentaire, souvent utilisée en défense en profondeur, consiste à ajouter un **verrou de réentrance** (*reentrancy guard*) explicite :

```solidity
contract CompteProtege {
    mapping(address => uint256) public soldes;
    bool private enCoursDExecution;   // verrou de réentrance

    modifier nonReentrant() {
        require(!enCoursDExecution, "Appel reentrant detecte");
        enCoursDExecution = true;
        _;
        enCoursDExecution = false;
    }

    function retirer(uint256 montant) external nonReentrant {
        require(soldes[msg.sender] >= montant, "Solde insuffisant");
        soldes[msg.sender] -= montant;
        payable(msg.sender).transfer(montant);
    }
}
```

Ce modificateur `nonReentrant` empêche explicitement qu'un même appel « en cours » soit rappelé récursivement, quelle que soit la fonction externe utilisée pour transférer les fonds — une défense indépendante et complémentaire au patron checks-effects-interactions, la combinaison des deux étant considérée comme une bonne pratique robuste.

### Réentrance inter-fonctions et réentrance en lecture seule

Un piège pédagogique fréquent consiste à croire qu'un verrou de réentrance appliqué à une seule fonction protège l'ensemble du contrat. En réalité, si deux fonctions distinctes partagent un état commun et qu'une seule des deux est protégée par le patron checks-effects-interactions ou par un verrou de réentrance, un attaquant peut parfois exploiter la fonction non protégée pendant l'exécution suspendue de la première — une variante dite **réentrance inter-fonctions**. Une variante plus subtile encore, la **réentrance en lecture seule** (*read-only reentrancy*), exploite le fait qu'une fonction externe rappelée pendant un appel en cours peut lire un état temporairement incohérent (par exemple, un solde déjà débité mais un transfert pas encore confirmé) via une fonction de lecture (`view`) d'un tout autre contrat qui se fie à cet état pour son propre calcul — illustrant que le raisonnement de sécurité doit porter sur l'ensemble des contrats en interaction, pas uniquement sur celui contenant la fonction vulnérable la plus évidente.

## 3. Débordement/soupassement d'entier (overflow/underflow)

Historiquement, un compteur non signé en Solidity pouvait « déborder » silencieusement (revenir à zéro après le maximum, ou à une valeur maximale après une soustraction sous zéro), exploité pour créer des soldes ou des quantités de jetons artificiellement énormes. Depuis Solidity 0.8, ces vérifications sont effectuées automatiquement par défaut (le compilateur annule la transaction en cas de dépassement), mais le risque reste réel pour du code utilisant des versions antérieures ou des blocs `unchecked` mal justifiés.

```solidity
// Exemple pédagogique illustrant le risque historique (avant Solidity 0.8, ou dans un bloc unchecked)
uint8 compteur = 255;       // uint8 : valeur maximale représentable = 255
unchecked {
    compteur += 1;           // débordement silencieux : compteur devient 0, sans erreur levée
}

// Depuis Solidity >= 0.8, sans le mot-clé `unchecked`, la même opération
// provoquerait automatiquement l'annulation de la transaction (revert).
```

Le mot-clé `unchecked` reste disponible depuis Solidity 0.8 pour des raisons d'optimisation du coût en gas dans des cas où le développeur peut prouver, par la logique du programme, qu'un dépassement est impossible (par exemple un compteur de boucle borné par une valeur elle-même petite) ; son usage doit néanmoins être systématiquement justifié et documenté lors d'un audit, car il réintroduit délibérément le risque historique dans le seul segment de code qu'il englobe.

## 4. Contrôle d'accès défaillant

Une fonction sensible (ex. modification du propriétaire du contrat, retrait de fonds réservés à l'administrateur) sans vérification appropriée des droits de l'appelant (`require(msg.sender == proprietaire)`) permet à n'importe quel utilisateur de l'exécuter. C'est l'équivalent, dans le monde des smart contracts, du contrôle d'accès défaillant classé numéro un de l'OWASP Top 10 (module *Audit organisation et technique*).

```solidity
// Version vulnérable : n'importe qui peut devenir propriétaire du contrat
contract GestionVulnerable {
    address public proprietaire;

    constructor() {
        proprietaire = msg.sender;
    }

    function changerProprietaire(address nouveauProprietaire) external {
        proprietaire = nouveauProprietaire;   // aucune vérification de l'appelant !
    }
}

// Version corrigée
contract GestionSecurisee {
    address public proprietaire;

    constructor() {
        proprietaire = msg.sender;
    }

    modifier seulementProprietaire() {
        require(msg.sender == proprietaire, "Reserve au proprietaire");
        _;
    }

    function changerProprietaire(address nouveauProprietaire) external seulementProprietaire {
        require(nouveauProprietaire != address(0), "Adresse invalide");
        proprietaire = nouveauProprietaire;
    }
}
```

Cet exemple illustre également une bonne pratique complémentaire : vérifier que l'adresse fournie n'est pas l'adresse nulle (`address(0)`), une erreur de saisie ou de logique qui, dans un contrat mal protégé, peut rendre définitivement inaccessibles des fonctions administratives (aucune clé privée ne correspond à l'adresse nulle).

### Fonctions non initialisées et visibilité par défaut

Une variante historique de cette classe de vulnérabilité, moins fréquente avec les versions récentes du compilateur qui imposent une visibilité explicite, mérite d'être connue à titre pédagogique : dans d'anciennes versions de Solidity, une fonction sans mot-clé de visibilité explicite était `public` par défaut, ce qui pouvait rendre accessible depuis l'extérieur une fonction destinée à un usage strictement interne (par exemple une fonction d'initialisation censée n'être appelée qu'une seule fois par le déploiement lui-même). Ce type d'erreur illustre un principe de sécurité plus général, déjà rencontré dans d'autres modules de ce cours : préférer systématiquement une **politique de refus par défaut** (*deny by default*, restreindre explicitement l'accès et n'ouvrir que ce qui est nécessaire) à une politique d'autorisation par défaut.

## 5. Attaques par manipulation d'oracle de prix

Un contrat DeFi qui détermine un prix d'actif à partir d'une source manipulable (ex. la liquidité instantanée d'un seul échange décentralisé) peut être trompé par un attaquant qui manipule temporairement ce prix (souvent via un **prêt flash**, un emprunt sans garantie remboursé dans la même transaction) pour déclencher des conditions favorables artificielles dans le contrat cible.

### Anatomie d'un scénario pédagogique de manipulation par prêt flash

Afin de rendre ce mécanisme concret, considérons un scénario entièrement fictif, à but pédagogique uniquement (les montants et le nom du protocole sont inventés) :

1. Un protocole fictif « PrêtDeFi » permet d'emprunter un actif en garantissant un autre actif, en évaluant la valeur de la garantie à partir du prix instantané observé sur un unique échange décentralisé de faible liquidité.
2. Un attaquant contracte un prêt flash important dans un autre actif, sans garantie, remboursable obligatoirement avant la fin de la même transaction.
3. Il utilise ces fonds empruntés pour acheter massivement, en une seule opération, l'actif utilisé comme garantie sur ce petit échange décentralisé, en faisant artificiellement grimper son prix instantané sur cet échange précis.
4. Toujours dans la même transaction, il dépose ce même actif (dont le prix affiché est maintenant artificiellement élevé) en garantie sur le protocole « PrêtDeFi », et emprunte un montant excessif, disproportionné par rapport à la valeur réelle de sa garantie.
5. Il rembourse le prêt flash initial (obligatoire dans la même transaction), et conserve la différence empruntée en excès — une perte nette pour les autres utilisateurs du protocole « PrêtDeFi ».

Ce scénario, bien que fictif dans ses détails numériques, correspond à une classe d'attaques réellement documentée dans l'écosystème DeFi et illustre pourquoi la conception d'un oracle de prix robuste (agrégation de plusieurs sources, moyennes temporelles, cf. chapitre 4) est une décision de sécurité de premier ordre, et non un détail d'implémentation secondaire, pour tout contrat manipulant des montants financiers déterminés par un prix externe.

## 6. Attaques par déni de service et boucles coûteuses en gas

Une fonction itérant sur une structure de données dont la taille peut être manipulée par un attaquant (ex. une liste d'utilisateurs qu'il peut faire croître) peut consommer plus de gas que le maximum autorisé par bloc, rendant la fonction définitivement inexécutable — un déni de service permanent et non corrigible sans migration du contrat.

```solidity
// Version vulnérable : la boucle grandit avec le nombre d'inscrits, sans limite
contract DistributionVulnerable {
    address[] public inscrits;

    function sInscrire() external {
        inscrits.push(msg.sender);
    }

    function distribuerRecompense() external {
        for (uint256 i = 0; i < inscrits.length; i++) {
            payable(inscrits[i]).transfer(1 ether); // coûte de plus en plus de gas à mesure que la liste grandit
        }
        // Au-delà d'un certain nombre d'inscrits, cette boucle dépasse la limite de gas
        // autorisée par bloc et la fonction devient définitivement inexécutable.
    }
}

// Version corrigée : modèle "pull" plutôt que "push" — chaque utilisateur retire lui-même sa part
contract DistributionCorrigee {
    mapping(address => bool) public inscrits;
    mapping(address => uint256) public recompensesDisponibles;

    function sInscrire() external {
        inscrits[msg.sender] = true;
        recompensesDisponibles[msg.sender] = 1 ether;
    }

    function retirerRecompense() external {
        uint256 montant = recompensesDisponibles[msg.sender];
        require(montant > 0, "Aucune recompense disponible");
        recompensesDisponibles[msg.sender] = 0;   // état mis à jour avant l'interaction externe (checks-effects-interactions)
        payable(msg.sender).transfer(montant);
    }
}
```

Ce couple d'exemples illustre un principe de conception plus général, connu sous le nom de **patron pull-over-push** : plutôt que pour un contrat de pousser proactivement des fonds vers une liste potentiellement non bornée de destinataires (coût cumulé porté par une seule transaction, donc un seul appelant), il est préférable de laisser chaque bénéficiaire retirer lui-même sa part dans sa propre transaction (coût réparti, chaque appelant ne payant que pour sa propre opération). Ce principe réapparaît fréquemment dans la conception de contrats DeFi robustes.

## 7. Démarche d'audit d'un smart contract

1. Revue manuelle du code, en confrontant systématiquement chaque fonction sensible aux classes de vulnérabilités ci-dessus.
2. Analyse statique automatisée (outils dédiés type Slither, MythX).
3. Tests unitaires et de fuzzing (génération automatique d'entrées pour détecter des comportements inattendus).
4. Vérification formelle pour les contrats à très haute valeur (preuve mathématique de propriétés de sécurité).
5. Programme de récompense pour la découverte de vulnérabilités (*bug bounty*) avant et après déploiement.

### Les limites propres à chaque outil d'audit

Il est pédagogiquement important de ne présenter aucun de ces outils comme une garantie absolue, mais bien comme des instruments complémentaires, chacun avec ses angles morts caractéristiques :

- l'**analyse statique automatisée** détecte efficacement les classes de vulnérabilités déjà connues et « signées » dans ses règles (réentrance simple, visibilité incorrecte, usages dangereux de primitives connues), mais peut manquer des failles logiques propres au métier du contrat (par exemple une condition d'éligibilité mal pensée sur le plan économique) et génère parfois de faux positifs à trier manuellement ;
- le **fuzzing** (génération automatique d'entrées, y compris de séquences d'appels de fonctions) est efficace pour découvrir des combinaisons d'entrées non anticipées par les développeurs, mais son efficacité dépend directement de la qualité des invariants définis par l'auditeur (les propriétés que le fuzzer doit essayer de violer) ;
- la **vérification formelle** offre la garantie la plus forte (une preuve mathématique qu'une propriété précisément spécifiée est respectée dans tous les cas), mais est coûteuse en temps d'ingénierie, ne couvre que les propriétés explicitement formalisées (une propriété oubliée dans la spécification reste un angle mort), et reste réservée en pratique aux contrats à très haute valeur ou à très fort risque systémique ;
- la **revue manuelle** reste irremplaçable pour évaluer la cohérence globale de la logique métier et repérer des vulnérabilités inédites, mais dépend fortement de l'expertise et de la disponibilité en temps de l'auditeur, et ne passe pas à l'échelle sur de très gros codebases sans être combinée aux approches automatisées.

Un audit rigoureux combine typiquement plusieurs de ces approches plutôt que de s'appuyer sur une seule, exactement comme un test de pénétration classique (module *Sécurité offensive*) combine différentes techniques complémentaires plutôt qu'un unique outil automatisé.

### Un rappel historique : l'attaque de The DAO

L'attaque de The DAO (2016), déjà mentionnée en introduction de ce chapitre comme illustration de la vulnérabilité de réentrance, mérite un rappel de son importance historique pour le domaine : au-delà de la perte financière directe, cet incident a provoqué un débat de gouvernance majeur au sein de la communauté Ethereum sur la conduite à tenir face à une exploitation manifeste d'une faille de code — fallait-il considérer que « le code fait loi » (*code is law*) et laisser les fonds détournés à l'attaquant, puisque son action respectait strictement les règles du code déployé, ou bien intervenir pour annuler les conséquences de l'attaque au nom de l'intention originelle des utilisateurs lésés ? Ce débat a conduit à un hard fork du réseau (chapitre 3) et à la coexistence, depuis lors, de deux chaînes distinctes issues de ce désaccord. Cet épisode reste, encore aujourd'hui, une référence pédagogique incontournable pour illustrer à la fois la criticité technique de la réentrance et la dimension de gouvernance propre aux blockchains publiques, qui dépasse la seule question technique de la sécurité du code.

## À retenir

- La réentrance, le contrôle d'accès défaillant, la manipulation d'oracle de prix et les attaques par déni de service via consommation excessive de gas sont des classes de vulnérabilités récurrentes, à l'origine de pertes financières réelles considérables dans l'écosystème des smart contracts.
- L'immutabilité des smart contracts déployés rend la prévention (audit avant déploiement) bien plus critique que pour une application web classique corrigible a posteriori ; des mécanismes d'évolutivité (proxy, pause d'urgence) existent mais introduisent leur propre surface d'attaque.
- Le patron checks-effects-interactions, éventuellement complété par un verrou de réentrance explicite, est la contre-mesure de référence contre la réentrance (y compris ses variantes inter-fonctions et en lecture seule), illustrant l'importance de l'ordre des opérations dans le code.
- Le patron pull-over-push (laisser chaque bénéficiaire retirer ses fonds plutôt que de les distribuer en boucle) prévient les attaques par déni de service liées à une consommation de gas non bornée.
- Aucun outil d'audit (analyse statique, fuzzing, vérification formelle, revue manuelle) n'est suffisant isolément : une démarche d'audit rigoureuse combine plusieurs approches complémentaires, exactement comme un test de pénétration classique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
