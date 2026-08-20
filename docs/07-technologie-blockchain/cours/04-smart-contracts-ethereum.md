# Chapitre 4 — Smart contracts et Ethereum

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/04-smart-contracts-ethereum.txt){ target=_blank }

## 1. Qu'est-ce qu'un smart contract ?

Un smart contract (« contrat intelligent ») est un programme informatique déployé sur une blockchain, dont le code et l'état s'exécutent de façon déterministe et vérifiable par l'ensemble des nœuds du réseau, sans possibilité de modification unilatérale une fois déployé (sauf mécanisme d'évolutivité explicitement prévu). Le terme « contrat » est trompeur au sens juridique : il s'agit avant tout d'un **programme auto-exécutable**, dont la portée juridique effective dépend du contexte réglementaire.

### Origine du concept

L'idée d'un « contrat intelligent » précède de plusieurs décennies son implémentation sur blockchain : le terme est attribué au chercheur Nick Szabo, qui décrivait dans les années 1990 l'idée d'encoder les clauses d'un contrat dans un logiciel capable de les exécuter automatiquement, avant même l'existence d'une infrastructure décentralisée capable de l'exécuter de façon fiable et vérifiable par des parties mutuellement méfiantes. Bitcoin dispose bien d'un langage de script rudimentaire (permettant, par exemple, d'exiger plusieurs signatures pour dépenser une transaction — un mécanisme dit *multisig*), mais ce langage est volontairement limité et non Turing-complet (pas de boucles, notamment), ce qui empêche l'exécution de programmes arbitrairement complexes. C'est précisément cette limitation que le concepteur d'Ethereum, Vitalik Buterin, a proposé de dépasser en 2013-2014, en concevant une blockchain dont le langage de script serait Turing-complet.

### Déterminisme : une contrainte de conception fondamentale

Une propriété essentielle, souvent sous-estimée par les étudiants découvrant le sujet, est que le code d'un smart contract doit être **strictement déterministe** : exécuté par des milliers de nœuds indépendants dans le monde, à des instants différents, sur des machines différentes, il doit systématiquement produire exactement le même résultat, faute de quoi les nœuds ne pourraient jamais s'accorder sur l'état résultant (rompant la propriété de consensus étudiée au chapitre 3). Cette contrainte a des conséquences concrètes sur ce qu'un smart contract peut ou ne peut pas faire nativement : pas d'accès direct à une horloge « réelle » précise, pas de génération de nombres véritablement aléatoires (une source de vulnérabilité classique, abordée au chapitre 5), pas d'appel réseau vers une ressource externe (d'où le problème de l'oracle, développé en section 6 de ce chapitre).

## 2. Ethereum : une blockchain programmable

Ethereum (2015) a étendu le modèle de Bitcoin en introduisant une **machine virtuelle Turing-complète** (l'EVM — Ethereum Virtual Machine), permettant d'exécuter des programmes arbitrairement complexes sur la blockchain, et non plus seulement des transactions de transfert de valeur.

### Notion de « gas »

Chaque opération exécutée par l'EVM consomme du « gas », payé par l'émetteur de la transaction dans la cryptomonnaie native (Ether). Ce mécanisme a un double rôle : rémunérer les validateurs pour les ressources de calcul consommées, et **empêcher les boucles infinies ou les attaques par déni de service** (un programme épuise son budget de gas avant de pouvoir consommer des ressources indéfiniment — solution pratique au problème de l'arrêt appliqué à un contexte adversarial).

### Le problème de l'arrêt, revisité

Le lien avec le problème de l'arrêt (informatique théorique) mérite d'être explicité : il est démontré qu'il n'existe, en général, aucun algorithme capable de déterminer, pour un programme et une entrée arbitraires, si son exécution se terminera ou bouclera indéfiniment. Une blockchain à smart contracts Turing-complets fait face à une version pratique et adversariale de ce problème : un validateur ne peut pas, avant exécution, garantir qu'un contrat soumis par un utilisateur quelconque terminera son exécution en un temps raisonnable — et un attaquant pourrait délibérément soumettre un programme conçu pour boucler indéfiniment, dans le but de paralyser le réseau (attaque par déni de service). Le mécanisme du gas contourne élégamment ce problème indécidable en le rendant **non pertinent** : peu importe qu'un programme termine ou non en théorie, il est de toute façon interrompu de force dès que son budget de gas est épuisé, la transaction étant alors annulée (mais le gas déjà consommé n'est pas remboursé, ce qui dissuade les tentatives malveillantes ou les erreurs de programmation coûteuses).

### Chaque opcode a un coût précis

Chaque instruction élémentaire de l'EVM (un « opcode », par exemple additionner deux nombres, lire une variable en mémoire, écrire une variable en stockage persistant) se voit attribuer un coût en gas fixé par le protocole, reflétant approximativement le coût réel de la ressource consommée par les validateurs pour l'exécuter. Un point pédagogique important : l'écriture d'une donnée en **stockage persistant** (`storage`, la partie de l'état d'un contrat qui reste enregistrée sur la blockchain entre deux transactions) coûte typiquement beaucoup plus cher en gas que la manipulation d'une variable en **mémoire temporaire** (`memory`, effacée à la fin de l'exécution de la transaction) — une différence de coût qui a des conséquences directes sur la façon dont un développeur Solidity doit structurer son code pour rester économique, et qui explique pourquoi les boucles itérant sur des structures de données stockées on-chain sont un point d'attention de sécurité particulier (chapitre 5, attaques par déni de service via consommation excessive de gas).

## 3. Langages et environnement de développement

- **Solidity** : langage le plus utilisé pour écrire des smart contracts sur Ethereum, syntaxe proche du JavaScript/C++.
- **Environnements de développement/test** : Remix (IDE en ligne), Hardhat, Foundry, permettant de développer, tester et déployer sur des réseaux de test avant tout déploiement sur le réseau principal.
- **Réseaux de test (testnets)** : répliques du réseau principal utilisant une monnaie sans valeur réelle, indispensables pour tester un contrat avant déploiement (utilisés en TP3/TP4).

### Cycle de vie du développement d'un smart contract

À titre pédagogique, le développement d'un smart contract destiné à un déploiement en production suit typiquement les étapes suivantes, chacune ayant un rôle distinct dans la réduction du risque (les risques spécifiques étant détaillés au chapitre 5) :

1. **Écriture du code** (Solidity ou un langage équivalent) et compilation vers le bytecode EVM.
2. **Tests unitaires locaux**, exécutés sur un nœud de développement local simulant l'EVM (rapide, sans coût réel), vérifiant le comportement attendu de chaque fonction, y compris les cas limites et les tentatives volontaires de faire échouer le contrat.
3. **Déploiement sur un ou plusieurs réseaux de test (testnets)**, avec une monnaie sans valeur réelle obtenue via un « robinet » (*faucet*), permettant de vérifier le comportement du contrat dans des conditions proches du réseau principal (latence réelle, coûts de gas réels bien que la monnaie soit fictive).
4. **Audit de sécurité** (interne et/ou par un tiers spécialisé), combinant revue manuelle et outils automatisés (chapitre 5).
5. **Déploiement sur le réseau principal**, généralement précédé d'un déploiement progressif ou d'une limite de fonds autorisés, le temps de gagner en confiance sur le comportement du contrat en conditions réelles.

Cette prudence méthodique s'explique directement par une propriété déjà mentionnée : un smart contract déployé est, par défaut, immuable — contrairement à une application web classique, il n'existe généralement pas de correctif silencieux possible après coup (chapitre 5 revient en détail sur cette contrainte et sur les mécanismes d'évolutivité qui tentent, avec leurs propres risques, d'y répondre).

## 4. Exemple simplifié de structure d'un smart contract (Solidity)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CompteSimple {
    mapping(address => uint256) public soldes;

    function deposer() external payable {
        soldes[msg.sender] += msg.value;
    }

    function retirer(uint256 montant) external {
        require(soldes[msg.sender] >= montant, "Solde insuffisant");
        soldes[msg.sender] -= montant;
        payable(msg.sender).transfer(montant);
    }
}
```

Ce court exemple illustre déjà des notions clés : état persistant sur la blockchain (`mapping`), vérifications explicites (`require`), et manipulation de valeur (Ether) — et prépare directement le chapitre suivant sur la sécurité (l'ordre des opérations dans `retirer` a une importance de sécurité directe).

### Éléments de vocabulaire Solidity à connaître

Le tableau suivant résume, à titre de référence, quelques éléments syntaxiques et sémantiques essentiels de Solidity utilisés dans ce chapitre et le suivant :

| Élément | Rôle |
|---|---|
| `mapping(clé => valeur)` | structure de données associative persistante, analogue à un dictionnaire, mais dont chaque emplacement non initialisé retourne une valeur par défaut (`0` pour un entier) plutôt qu'une erreur |
| `external` / `public` / `internal` / `private` | visibilité d'une fonction ou variable — `external` n'est appelable que depuis l'extérieur du contrat, `internal`/`private` restreignent l'accès au contrat (et ses héritiers pour `internal`) |
| `payable` | marque une fonction ou une adresse comme capable de recevoir de l'Ether |
| `msg.sender` | adresse ayant initié l'appel de la fonction courante (variable globale fournie par l'EVM) |
| `msg.value` | montant d'Ether envoyé avec l'appel courant |
| `require(condition, message)` | interrompt l'exécution et annule toutes les modifications d'état effectuées si la condition est fausse, en remboursant le gas non consommé |
| `view` / `pure` | marquent une fonction qui ne modifie pas l'état (`view`) ou qui ne le lit même pas (`pure`) — utile pour la lisibilité et pour permettre des appels gratuits en lecture seule |

### Exemple pédagogique complet : un contrat de vote simplifié

Afin d'illustrer un cas d'usage au-delà du simple transfert de fonds, voici un contrat pédagogique simplifié de vote, dont la logique met en jeu davantage de structures de données et d'états :

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VoteSimple {
    address public organisateur;
    mapping(address => bool) public aVote;
    mapping(string => uint256) public voix;
    bool public voteOuvert;

    constructor() {
        organisateur = msg.sender;   // le déployeur du contrat devient l'organisateur
        voteOuvert = true;
    }

    modifier seulementOrganisateur() {
        require(msg.sender == organisateur, "Reserve a l'organisateur");
        _;
    }

    function voter(string calldata candidat) external {
        require(voteOuvert, "Le vote est clos");
        require(!aVote[msg.sender], "Vous avez deja vote");
        aVote[msg.sender] = true;
        voix[candidat] += 1;
    }

    function cloturerVote() external seulementOrganisateur {
        voteOuvert = false;
    }
}
```

Cet exemple introduit la notion de **modificateur** (`modifier`), un mécanisme Solidity permettant de factoriser une condition de contrôle d'accès réutilisable sur plusieurs fonctions — directement pertinent pour la classe de vulnérabilité « contrôle d'accès défaillant » étudiée au chapitre 5. Il illustre également, de façon volontairement simplifiée à des fins pédagogiques, les limites d'un vote purement on-chain : l'anonymat de l'électeur n'est pas assuré (chaque vote est publiquement associé à l'adresse de l'électeur), un point directement repris dans la discussion sur le vote électronique au chapitre 6.

## 5. Cas d'usage des smart contracts

- **Jetons (tokens)** : représentation d'actifs numériques selon des standards (ERC-20 pour des jetons fongibles, ERC-721 pour des jetons non fongibles/NFT).
- **Finance décentralisée (DeFi)** : prêts, échanges décentralisés (DEX), sans intermédiaire financier traditionnel.
- **Gouvernance décentralisée (DAO)** : votes et décisions automatisées selon des règles codées.
- **Traçabilité** : suivi de chaîne logistique, certification de provenance.

### Les standards ERC : un aperçu

Les standards ERC (*Ethereum Request for Comments*) sont des interfaces normalisées, adoptées par la communauté des développeurs, que doit respecter un smart contract pour être reconnu et manipulé de façon uniforme par les portefeuilles, plateformes d'échange et autres contrats de l'écosystème — un rôle analogue à celui d'une interface ou d'un protocole normalisé en génie logiciel classique. Deux standards sont particulièrement structurants :

- **ERC-20** (jetons fongibles) définit un ensemble minimal de fonctions qu'un contrat de jeton doit implémenter (`transfer`, `balanceOf`, `approve`, notamment), permettant à n'importe quel jeton respectant ce standard d'être immédiatement compatible avec l'ensemble des portefeuilles et plateformes de l'écosystème, sans intégration spécifique préalable ;
- **ERC-721** (jetons non fongibles, NFT) définit une interface où chaque jeton émis est identifié individuellement par un numéro unique et n'est pas interchangeable avec un autre jeton du même contrat — adapté à la représentation d'actifs uniques (œuvre numérique, titre de propriété, objet de collection).

Un point pédagogique important : le respect d'un standard ERC est une convention logicielle vérifiée par la présence des bonnes signatures de fonctions, non une garantie de sécurité intrinsèque — un contrat peut parfaitement implémenter l'interface ERC-20 tout en contenant, par ailleurs, l'une des vulnérabilités étudiées au chapitre 5.

## 6. Le problème de l'oracle

Un smart contract, par conception, ne peut accéder qu'aux données déjà présentes sur la blockchain : il ne peut pas consulter directement une donnée du monde extérieur (cours d'une action, résultat sportif, météo). Un **oracle** est un service tiers chargé d'apporter cette donnée externe sur la blockchain. Ce point est une source de risque majeure : la sécurité de tout le smart contract dépend alors de la fiabilité de l'oracle utilisé, qui redevient un point de confiance centralisé, en tension avec l'objectif initial de décentralisation.

### Stratégies d'atténuation du risque d'oracle

Plusieurs approches, chacune avec ses propres compromis, tentent de réduire (sans jamais totalement éliminer) le risque associé à un point de confiance centralisé introduit par un oracle :

- **agrégation de plusieurs sources indépendantes** : combiner (par exemple par médiane) les valeurs rapportées par plusieurs fournisseurs de données indépendants, de façon à ce qu'un attaquant doive compromettre simultanément une majorité d'entre eux pour manipuler la donnée finale ;
- **réseaux d'oracles décentralisés** : faire reporter la donnée par un ensemble de nœuds opérés par des entités distinctes, avec un mécanisme de consensus et d'incitation économique dédié (garantie déposée, pénalisée en cas de donnée manifestement erronée), rapprochant conceptuellement le problème de l'oracle du problème général du consensus étudié au chapitre 3 ;
- **délai et fenêtres de temps moyennées** : plutôt que d'utiliser un prix instantané (manipulable ponctuellement, notamment via un prêt flash, chapitre 5), utiliser une moyenne calculée sur une fenêtre de temps, rendant la manipulation plus coûteuse car devant être soutenue plus longtemps ;
- **oracles optimistes avec période de contestation** : publier une donnée avec un délai pendant lequel tout observateur peut la contester en apportant une preuve contraire, avant qu'elle ne soit considérée comme définitive par les contrats consommateurs.

Aucune de ces approches ne supprime totalement le problème de fond : par construction, un smart contract ne peut jamais avoir une certitude cryptographique équivalente à celle qu'il a sur ses propres données internes lorsqu'il s'agit d'une information provenant du monde extérieur — un rappel utile face au discours parfois trop optimiste sur la capacité de la blockchain à « garantir la vérité ».

## À retenir

- Un smart contract est un programme auto-exécutable et strictement déterministe (contrainte nécessaire pour que tous les nœuds du réseau convergent vers le même résultat), pas nécessairement un contrat juridique au sens classique.
- Le mécanisme de gas rémunère l'exécution, avec un coût par opération reflétant approximativement la ressource réellement consommée, et contourne élégamment le problème théorique de l'arrêt en interrompant de force toute exécution dépassant le budget alloué.
- Les standards ERC (ERC-20 pour les jetons fongibles, ERC-721 pour les jetons non fongibles) normalisent les interfaces des smart contracts pour assurer leur interopérabilité avec l'écosystème, sans garantir pour autant leur sécurité intrinsèque.
- Le problème de l'oracle rappelle qu'un smart contract ne résout la confiance que pour les données déjà présentes sur la chaîne, pas pour les données du monde réel qui y sont importées ; plusieurs stratégies (agrégation, décentralisation des oracles, moyennes temporelles) atténuent ce risque sans jamais l'éliminer complètement.
- La prudence méthodique du cycle de développement (tests, testnets, audit avant déploiement) découle directement de l'immutabilité par défaut des smart contracts déployés, un point développé en détail au chapitre 5.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
