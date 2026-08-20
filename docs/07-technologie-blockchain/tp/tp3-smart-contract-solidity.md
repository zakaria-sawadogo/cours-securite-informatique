# TP3 — Développement d'un smart contract simple (Solidity) (2h)

## Cadre du TP

Développement exclusivement sur un réseau de test local (Remix VM) ou un testnet public, sans fonds réels.

## Objectifs
- Écrire, compiler et déployer un smart contract simple sur Remix.
- Interagir avec le contrat déployé.

## Exercice 1 — Prise en main de Remix

Ouvrir [remix.ethereum.org](https://remix.ethereum.org), créer un fichier `CompteSimple.sol` reprenant l'exemple du chapitre 4 du cours. Compiler le contrat et le déployer sur l'environnement « Remix VM » (blockchain locale simulée).

## Exercice 2 — Interaction avec le contrat

Depuis l'interface Remix, appeler la fonction `deposer()` avec plusieurs comptes de test différents (Remix en fournit plusieurs avec des fonds de test), vérifier la mise à jour du mapping `soldes`, puis appeler `retirer()` et vérifier le solde après retrait. Documenter chaque appel et son résultat (capture d'écran).

## Exercice 3 — Ajout d'une fonctionnalité

Étendre le contrat avec une fonction `transferer(address destinataire, uint256 montant)` permettant à un utilisateur de transférer une partie de son solde interne à un autre compte, sans passer par un transfert d'Ether réel (uniquement une mise à jour du mapping). Ajouter les vérifications nécessaires (solde suffisant, destinataire valide).

## Exercice 4 — Événements

Ajouter un événement Solidity (`event Depot(address indexed compte, uint256 montant);`) émis à chaque dépôt, et vérifier dans les logs de transaction de Remix qu'il est bien émis avec les bonnes valeurs. Expliquer par écrit l'utilité des événements pour une application externe (ex. interface web) qui souhaiterait suivre l'activité du contrat sans interroger l'état à chaque instant.

## Exercice 5 — Tests

Écrire au moins 3 tests unitaires (avec le module de test intégré de Remix ou Hardhat/Foundry si l'environnement est disponible) couvrant : un dépôt réussi, un retrait avec solde suffisant, un retrait refusé pour solde insuffisant.

## À rendre

Le fichier `.sol` complet, les captures d'écran de déploiement/interaction, et les tests unitaires.
