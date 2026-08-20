# Chapitre 1 — Introduction : principes fondamentaux de la blockchain

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/01-introduction-principes-blockchain.txt){ target=_blank }

## 1. Définition

Une blockchain est un registre distribué, répliqué sur de nombreux nœuds d'un réseau pair-à-pair, structuré en blocs chaînés cryptographiquement, dont le contenu passé est rendu extrêmement difficile à modifier une fois validé, sans nécessiter d'autorité centrale unique de confiance.

## 2. Genèse : Bitcoin (2008-2009)

Publié sous le pseudonyme Satoshi Nakamoto en 2008, Bitcoin est la première application combinant, de façon inédite, des briques cryptographiques déjà connues (signatures numériques, fonctions de hachage) avec un mécanisme de consensus décentralisé (preuve de travail, chapitre 3) pour résoudre le **problème de la double dépense** sans tiers de confiance central : comment empêcher qu'une même unité de monnaie numérique soit dépensée deux fois, sans banque centrale ?

## 3. Propriétés recherchées

| Propriété | Signification |
|---|---|
| **Décentralisation** | aucune entité unique ne contrôle le registre ; il est répliqué et validé par de nombreux participants indépendants |
| **Immutabilité (pratique)** | modifier un bloc déjà validé et suffisamment confirmé nécessiterait de refaire le travail de validation de tous les blocs suivants, un coût prohibitif au-delà d'une certaine profondeur |
| **Transparence** | (pour les blockchains publiques) l'historique complet des transactions est consultable par tous |
| **Résistance à la censure** | aucune autorité unique ne peut empêcher une transaction valide d'être incluse, dans un réseau suffisamment décentralisé |

Ces propriétés sont des objectifs de conception, pas des garanties absolues : elles dépendent fortement du degré réel de décentralisation du réseau (chapitre 3) et de la sécurité des implémentations (chapitre 5).

## 4. Blockchain publique, privée, à permission

| Type | Accès en écriture | Accès en lecture | Exemple |
|---|---|---|---|
| **Publique** (permissionless) | ouvert à tous | ouvert à tous | Bitcoin, Ethereum |
| **À permission (consortium)** | limité à des participants identifiés | selon règles définies | réseaux inter-bancaires, chaînes logistiques inter-entreprises |
| **Privée** | contrôlé par une seule organisation | souvent restreint | usage interne d'entreprise, proche d'une base de données répliquée classique |

Une confusion fréquente : une blockchain « privée » ou « à permission » perd une grande partie des propriétés de décentralisation et de résistance à la censure d'une blockchain publique — le choix du type doit être justifié par le besoin réel, pas par un effet de mode.

## 5. Anatomie d'un bloc (aperçu)

Chaque bloc contient typiquement : un en-tête (empreinte du bloc précédent, horodatage, racine de Merkle des transactions du bloc — chapitre 2, un nonce pour le mécanisme de consensus), et le corps (liste des transactions incluses).

## 6. Ce que la blockchain résout, et ce qu'elle ne résout pas

- Elle résout la **cohérence d'un registre partagé sans tiers de confiance unique**, dans un contexte où les participants peuvent être mutuellement méfiants.
- Elle ne résout pas, par elle-même, la **véracité des données saisies** (« garbage in, garbage out ») : une blockchain garantit l'intégrité de ce qui y a été inscrit, pas la véracité de l'information au moment de sa saisie (problème dit de l'« oracle », chapitre 4).
- Elle a un coût réel : stockage répliqué, consommation énergétique pour certains mécanismes de consensus (chapitre 3), latence de confirmation, complexité opérationnelle — d'où l'importance d'évaluer si une base de données traditionnelle ne répondrait pas mieux à un besoin donné (chapitre 6).

## À retenir

- La blockchain combine des primitives cryptographiques connues avec un mécanisme de consensus décentralisé pour résoudre le problème de la double dépense sans autorité centrale.
- Les propriétés de décentralisation, immutabilité et résistance à la censure sont des objectifs de conception dont le degré réel dépend de l'architecture choisie.
- Une blockchain n'est pas une solution universelle : son usage doit être justifié par un besoin réel de décentralisation, pas adopté par principe.
