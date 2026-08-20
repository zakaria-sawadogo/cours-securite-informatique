# Technologie Blockchain

**Volume horaire :** 16h CM · 8h TP (24h)

## Objectifs pédagogiques

- Comprendre les principes fondamentaux d'une blockchain (structure de données, chaînage cryptographique, décentralisation).
- Comprendre les principaux mécanismes de consensus (preuve de travail, preuve d'enjeu, consensus byzantin) et leurs compromis.
- Comprendre le fonctionnement des smart contracts et leurs risques de sécurité spécifiques.
- Développer un regard critique sur les usages, limites et enjeux réglementaires de la technologie blockchain.

## Prérequis

Ce module s'appuie fortement sur les notions du module *Cryptographie* (fonctions de hachage, signatures numériques, arbres de Merkle) : il est recommandé de suivre ce dernier en amont ou en parallèle.

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction : principes fondamentaux de la blockchain](cours/01-introduction-principes-blockchain.md) | 2h |
| 2 | [Structures de données : chaînage cryptographique et arbres de Merkle](cours/02-structures-donnees-merkle.md) | 3h |
| 3 | [Mécanismes de consensus](cours/03-mecanismes-consensus.md) | 3h |
| 4 | [Smart contracts et Ethereum](cours/04-smart-contracts-ethereum.md) | 3h |
| 5 | [Sécurité et vulnérabilités des smart contracts](cours/05-securite-smart-contracts.md) | 2h |
| 6 | [Applications, enjeux et limites](cours/06-applications-enjeux-limites.md) | 3h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Implémentation d'une mini-blockchain en Python](tp/tp1-mini-blockchain-python.md) | 2h |
| 2 | [Simulation d'un consensus par preuve de travail](tp/tp2-simulation-consensus-pow.md) | 2h |
| 3 | [Développement d'un smart contract simple (Solidity)](tp/tp3-smart-contract-solidity.md) | 2h |
| 4 | [Audit de vulnérabilités d'un smart contract](tp/tp4-audit-smart-contract.md) | 2h |

## Évaluation

- TP notés : 40 %
- Exposé sur un cas d'usage ou une faille blockchain réelle : 20 %
- Examen final : 40 %

## Environnement technique

Python pour le TP1/TP2 ; Remix IDE (en ligne, environnement de test local) et Solidity pour les TP3/TP4 — aucune manipulation sur un réseau principal (mainnet) réel ni de fonds réels dans le cadre de ce cours.

## Bibliographie

- A. Antonopoulos, *Mastering Bitcoin*.
- A. Antonopoulos, G. Wood, *Mastering Ethereum*.
- S. Nakamoto, *Bitcoin: A Peer-to-Peer Electronic Cash System* (2008).
- ConsenSys, *Smart Contract Best Practices*.
