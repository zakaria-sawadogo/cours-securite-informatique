# Chapitre 5 — Sécurité et vulnérabilités des smart contracts

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/05-securite-smart-contracts.txt){ target=_blank }

## 1. Pourquoi la sécurité des smart contracts est particulièrement critique

Un smart contract déployé sur une blockchain publique est, par construction, **immuable et son code est public** : contrairement à une application classique, il n'est généralement pas possible de corriger une faille par un simple correctif déployé discrètement — le code vulnérable reste actif et visible de tous jusqu'à une migration complexe. De plus, les contrats manipulent souvent directement des fonds réels, ce qui en fait des cibles à forte valeur immédiate pour un attaquant.

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

## 3. Débordement/soupassement d'entier (overflow/underflow)

Historiquement, un compteur non signé en Solidity pouvait « déborder » silencieusement (revenir à zéro après le maximum, ou à une valeur maximale après une soustraction sous zéro), exploité pour créer des soldes ou des quantités de jetons artificiellement énormes. Depuis Solidity 0.8, ces vérifications sont effectuées automatiquement par défaut (le compilateur annule la transaction en cas de dépassement), mais le risque reste réel pour du code utilisant des versions antérieures ou des blocs `unchecked` mal justifiés.

## 4. Contrôle d'accès défaillant

Une fonction sensible (ex. modification du propriétaire du contrat, retrait de fonds réservés à l'administrateur) sans vérification appropriée des droits de l'appelant (`require(msg.sender == proprietaire)`) permet à n'importe quel utilisateur de l'exécuter. C'est l'équivalent, dans le monde des smart contracts, du contrôle d'accès défaillant classé numéro un de l'OWASP Top 10 (module *Audit organisation et technique*).

## 5. Attaques par manipulation d'oracle de prix

Un contrat DeFi qui détermine un prix d'actif à partir d'une source manipulable (ex. la liquidité instantanée d'un seul échange décentralisé) peut être trompé par un attaquant qui manipule temporairement ce prix (souvent via un **prêt flash**, un emprunt sans garantie remboursé dans la même transaction) pour déclencher des conditions favorables artificielles dans le contrat cible.

## 6. Attaques par déni de service et boucles coûteuses en gas

Une fonction itérant sur une structure de données dont la taille peut être manipulée par un attaquant (ex. une liste d'utilisateurs qu'il peut faire croître) peut consommer plus de gas que le maximum autorisé par bloc, rendant la fonction définitivement inexécutable — un déni de service permanent et non corrigible sans migration du contrat.

## 7. Démarche d'audit d'un smart contract

1. Revue manuelle du code, en confrontant systématiquement chaque fonction sensible aux classes de vulnérabilités ci-dessus.
2. Analyse statique automatisée (outils dédiés type Slither, MythX).
3. Tests unitaires et de fuzzing (génération automatique d'entrées pour détecter des comportements inattendus).
4. Vérification formelle pour les contrats à très haute valeur (preuve mathématique de propriétés de sécurité).
5. Programme de récompense pour la découverte de vulnérabilités (*bug bounty*) avant et après déploiement.

## À retenir

- La réentrance, le contrôle d'accès défaillant et la manipulation d'oracle sont des classes de vulnérabilités récurrentes, à l'origine de pertes financières réelles considérables.
- L'immutabilité des smart contracts déployés rend la prévention (audit avant déploiement) bien plus critique que pour une application web classique corrigible a posteriori.
- Le patron checks-effects-interactions est la contre-mesure de référence contre la réentrance, illustrant l'importance de l'ordre des opérations dans le code.
