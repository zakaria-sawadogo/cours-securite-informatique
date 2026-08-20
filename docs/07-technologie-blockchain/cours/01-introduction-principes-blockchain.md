# Chapitre 1 — Introduction : principes fondamentaux de la blockchain

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/01-introduction-principes-blockchain.txt){ target=_blank }

## 1. Définition

Une blockchain est un registre distribué, répliqué sur de nombreux nœuds d'un réseau pair-à-pair, structuré en blocs chaînés cryptographiquement, dont le contenu passé est rendu extrêmement difficile à modifier une fois validé, sans nécessiter d'autorité centrale unique de confiance.

Cette définition volontairement générale mérite d'être décomposée en ses ingrédients constitutifs, chacun étant étudié en détail dans un chapitre ultérieur de ce module :

- un **registre** (*ledger*) : une liste ordonnée et append-only d'enregistrements (les transactions) ;
- **distribué** : ce registre n'existe pas en un exemplaire unique conservé par une entité, mais est répliqué intégralement (ou partiellement, pour les clients légers) chez de nombreux participants indépendants ;
- **pair-à-pair** : les participants communiquent directement entre eux, sans serveur central obligatoire, par un protocole de propagation de type *gossip* ;
- **blocs chaînés cryptographiquement** : les enregistrements sont regroupés en blocs, et chaque bloc référence cryptographiquement le précédent (chapitre 2) ;
- **difficile à modifier une fois validé** : propriété obtenue par la combinaison du chaînage cryptographique et d'un mécanisme de consensus coûteux à tromper (chapitre 3) ;
- **sans autorité centrale unique** : la validation des blocs est distribuée entre de nombreux participants suivant une règle algorithmique connue de tous, et non décidée par un tiers de confiance unique.

Il est important de noter dès à présent qu'aucun de ces ingrédients n'est, pris isolément, une nouveauté informatique : les fonctions de hachage, les listes chaînées, les systèmes répliqués et les protocoles de consensus distribué existaient tous avant 2008. L'apport de Bitcoin est de les **combiner** de façon à obtenir, pour la première fois, un système monétaire numérique fonctionnant sans tiers de confiance central — une synthèse d'ingénierie plus qu'une invention cryptographique isolée.

### Registre centralisé, registre partagé, registre distribué

Pour bien situer l'apport de la blockchain, il est utile de comparer trois architectures de registre :

| Architecture | Qui détient la vérité ? | Exemple |
|---|---|---|
| **Registre centralisé** | une entité unique (une banque tient les comptes de ses clients) | système bancaire classique, base de données d'entreprise |
| **Registre partagé (répliqué, sans consensus décentralisé)** | plusieurs copies existent, mais une entité reste maîtresse des écritures (réplication maître-esclave) | grappe de bases de données classiques répliquées pour la disponibilité |
| **Registre distribué avec consensus décentralisé (blockchain)** | aucune entité unique ; l'accord se fait par un protocole algorithmique entre pairs mutuellement méfiants | Bitcoin, Ethereum |

La différence essentielle entre les deuxième et troisième lignes n'est donc pas la simple réplication (qui existe depuis longtemps dans les systèmes distribués classiques), mais la capacité à faire converger vers un état unique un ensemble de participants qui ne se font *pas confiance* les uns envers les autres et qui peuvent inclure des acteurs malveillants — un problème nettement plus difficile que la simple tolérance aux pannes.

## 2. Genèse : Bitcoin (2008-2009)

Publié sous le pseudonyme Satoshi Nakamoto en 2008, Bitcoin est la première application combinant, de façon inédite, des briques cryptographiques déjà connues (signatures numériques, fonctions de hachage) avec un mécanisme de consensus décentralisé (preuve de travail, chapitre 3) pour résoudre le **problème de la double dépense** sans tiers de confiance central : comment empêcher qu'une même unité de monnaie numérique soit dépensée deux fois, sans banque centrale ?

### Le problème de la double dépense

Dans un système de monnaie purement numérique (une suite de bits), rien n'empêche *a priori* un utilisateur de copier cette suite de bits et de la transmettre deux fois à deux bénéficiaires différents — contrairement à un billet physique, qui ne peut se trouver, à un instant donné, que dans une seule main. Avant Bitcoin, la solution standard à ce problème consistait à faire confiance à un tiers centralisé (une banque, un opérateur de monnaie électronique) qui tient un registre unique des soldes et refuse toute transaction qui dépenserait deux fois le même montant. L'apport conceptuel de Bitcoin est de montrer qu'un réseau de participants mutuellement méfiants, sans tiers central, peut néanmoins se mettre d'accord sur un **ordre unique** des transactions — et donc détecter et rejeter toute tentative de double dépense — à condition d'accepter un mécanisme de consensus coûteux (la preuve de travail) qui rend économiquement irrationnelle la falsification de cet ordre.

### Des idées antérieures à la synthèse de 2008

Plusieurs briques utilisées par Bitcoin existaient déjà dans la littérature académique avant 2008 : les fonctions de hachage cryptographiques et les arbres de Merkle (Ralph Merkle, fin des années 1970), les systèmes de preuve de travail conçus initialement contre le spam (*Hashcash*, fin des années 1990), et diverses propositions de monnaie numérique décentralisée qui n'avaient pas rencontré d'adoption large. Le mérite de la proposition de Satoshi Nakamoto est d'avoir assemblé ces éléments en un système cohérent, complet et effectivement déployé, avec une incitation économique (la récompense de minage) alignant l'intérêt individuel des participants sur la sécurité collective du réseau.

## 3. Propriétés recherchées

| Propriété | Signification |
|---|---|
| **Décentralisation** | aucune entité unique ne contrôle le registre ; il est répliqué et validé par de nombreux participants indépendants |
| **Immutabilité (pratique)** | modifier un bloc déjà validé et suffisamment confirmé nécessiterait de refaire le travail de validation de tous les blocs suivants, un coût prohibitif au-delà d'une certaine profondeur |
| **Transparence** | (pour les blockchains publiques) l'historique complet des transactions est consultable par tous |
| **Résistance à la censure** | aucune autorité unique ne peut empêcher une transaction valide d'être incluse, dans un réseau suffisamment décentralisé |

Ces propriétés sont des objectifs de conception, pas des garanties absolues : elles dépendent fortement du degré réel de décentralisation du réseau (chapitre 3) et de la sécurité des implémentations (chapitre 5).

### Décentralisation : trois dimensions distinctes

Le terme « décentralisation » est souvent employé de façon floue. Il est utile de le décomposer en (au moins) trois dimensions indépendantes, un même réseau pouvant être décentralisé selon l'une et centralisé selon une autre :

- **décentralisation architecturale** : combien de machines physiques distinctes composent le réseau, et où sont-elles situées ?
- **décentralisation politique** : combien d'individus ou d'organisations distincts contrôlent effectivement ces machines (un même opérateur peut faire tourner des milliers de nœuds) ?
- **décentralisation logique** : l'état et les règles du système forment-ils un ensemble monolithique unique, ou pourrait-il en principe être scindé en deux moitiés indépendantes sans perte de sens (un carnet d'adresses réparti est décentralisé de façon logique ; un registre de comptes bancaires cohérent ne l'est pas) ?

Un exemple pédagogique : un réseau comptant dix mille nœuds répartis dans le monde entier (forte décentralisation architecturale) mais dont 90 % de la puissance de minage appartient à trois entités minières (faible décentralisation politique) offre, en pratique, un niveau de résistance à la censure et de sécurité contre une attaque à 51 % nettement inférieur à ce que suggère le seul décompte des nœuds.

## 4. Blockchain publique, privée, à permission

| Type | Accès en écriture | Accès en lecture | Exemple |
|---|---|---|---|
| **Publique** (permissionless) | ouvert à tous | ouvert à tous | Bitcoin, Ethereum |
| **À permission (consortium)** | limité à des participants identifiés | selon règles définies | réseaux inter-bancaires, chaînes logistiques inter-entreprises |
| **Privée** | contrôlé par une seule organisation | souvent restreint | usage interne d'entreprise, proche d'une base de données répliquée classique |

Une confusion fréquente : une blockchain « privée » ou « à permission » perd une grande partie des propriétés de décentralisation et de résistance à la censure d'une blockchain publique — le choix du type doit être justifié par le besoin réel, pas par un effet de mode.

Un exemple pédagogique permet d'illustrer ce choix. Imaginons un consortium fictif de cinq entreprises de logistique d'une même région souhaitant partager un registre commun de suivi de conteneurs : les cinq entreprises se connaissent, ont des intérêts partiellement alignés (fluidifier les échanges) et partiellement divergents (chacune souhaite éviter d'être accusée à tort d'un retard). Une blockchain à permission, où chacune des cinq organisations exploite un nœud validateur identifié, répond bien à ce besoin : elle élimine la nécessité de faire confiance à l'infrastructure d'une seule des cinq entreprises, sans pour autant nécessiter l'anonymat et la résistance à la censure d'un réseau ouvert à des inconnus comme Bitcoin. À l'inverse, un cas d'usage où n'importe quel individu dans le monde doit pouvoir participer sans autorisation préalable (un système de paiement mondial résistant à la censure d'un État) requiert par construction une blockchain publique.

## 5. Anatomie d'un bloc (aperçu)

Chaque bloc contient typiquement : un en-tête (empreinte du bloc précédent, horodatage, racine de Merkle des transactions du bloc — chapitre 2, un nonce pour le mécanisme de consensus), et le corps (liste des transactions incluses).

### Structure schématique d'un en-tête de bloc

Le tableau suivant détaille, à titre pédagogique, les champs typiquement présents dans l'en-tête d'un bloc de type Bitcoin — les blockchains à smart contracts (chapitre 4) ajoutent des champs supplémentaires (racine de l'état global, notamment) :

| Champ | Rôle |
|---|---|
| Empreinte du bloc précédent | assure le chaînage cryptographique (chapitre 2) |
| Horodatage (*timestamp*) | date approximative de création du bloc, utilisée notamment pour ajuster la difficulté du consensus |
| Racine de Merkle | résume l'ensemble des transactions du bloc en une seule empreinte (chapitre 2) |
| Nonce | valeur ajustée par le mineur pour satisfaire la condition de difficulté du consensus (chapitre 3) |
| Numéro de version / difficulté | paramètres de protocole utilisés par les nœuds pour valider le bloc |

Un point pédagogique important : l'en-tête d'un bloc est de taille fixe et petite (quelques dizaines d'octets), indépendamment du nombre de transactions qu'il résume — c'est précisément ce que permet l'usage de la racine de Merkle plutôt que la liste complète des transactions (chapitre 2). Cette propriété est ce qui rend possible la vérification par des clients légers ne téléchargeant que les en-têtes de blocs.

## 6. Ce que la blockchain résout, et ce qu'elle ne résout pas

- Elle résout la **cohérence d'un registre partagé sans tiers de confiance unique**, dans un contexte où les participants peuvent être mutuellement méfiants.
- Elle ne résout pas, par elle-même, la **véracité des données saisies** (« garbage in, garbage out ») : une blockchain garantit l'intégrité de ce qui y a été inscrit, pas la véracité de l'information au moment de sa saisie (problème dit de l'« oracle », chapitre 4).
- Elle a un coût réel : stockage répliqué, consommation énergétique pour certains mécanismes de consensus (chapitre 3), latence de confirmation, complexité opérationnelle — d'où l'importance d'évaluer si une base de données traditionnelle ne répondrait pas mieux à un besoin donné (chapitre 6).

### Le problème de l'oracle, une première approche

Ce point mérite d'être approfondi dès l'introduction, car il est fréquemment source de malentendus chez les étudiants découvrant la blockchain : une blockchain garantit l'intégrité de ce qui a été inscrit dans le registre (personne ne peut modifier discrètement une transaction déjà validée), mais elle ne peut, par construction, rien garantir sur la véracité de l'information au moment où elle y a été inscrite. Un exemple pédagogique simple : si un entrepôt fictif enregistre sur une blockchain « 500 sacs de céréales reçus le 12 mars », la blockchain garantit qu'aucun acteur ne pourra plus tard prétendre que l'enregistrement disait « 400 sacs » — mais rien ne garantit que 500 sacs ont réellement été livrés ce jour-là plutôt que 300. Ce problème, dit **problème de l'oracle**, est repris et approfondi au chapitre 4 dans le contexte spécifique des smart contracts, où il devient particulièrement critique lorsque des sommes d'argent sont automatiquement transférées sur la base d'une donnée externe.

### Comparaison synthétique avec une base de données traditionnelle

Le tableau suivant résume, à titre pédagogique, les différences de compromis entre une blockchain publique et une base de données centralisée classique, afin de préparer la grille de décision développée au chapitre 6 :

| Critère | Base de données centralisée | Blockchain publique |
|---|---|---|
| Débit de transactions | très élevé (dépend uniquement du matériel) | limité par le mécanisme de consensus (chapitre 3) |
| Tiers de confiance nécessaire | oui (l'opérateur de la base) | non, si le réseau est suffisamment décentralisé |
| Coût de modification a posteriori | faible (une simple requête `UPDATE`) | très élevé au-delà de quelques confirmations |
| Résistance à la censure | dépend entièrement de l'opérateur | forte, si le réseau est suffisamment décentralisé |
| Coût opérationnel | maîtrisé, proportionnel à l'usage | réplication systématique chez tous les nœuds, potentiellement coûteuse en énergie (PoW) |

## À retenir

- La blockchain combine des primitives cryptographiques connues (fonctions de hachage, signatures numériques) avec un mécanisme de consensus décentralisé pour résoudre le problème de la double dépense sans autorité centrale — une synthèse d'ingénierie plutôt qu'une rupture cryptographique isolée.
- Les propriétés de décentralisation, immutabilité et résistance à la censure sont des objectifs de conception dont le degré réel dépend de l'architecture choisie ; la décentralisation elle-même se décompose en dimensions architecturale, politique et logique qui peuvent diverger.
- Le choix entre blockchain publique, à permission ou privée doit être guidé par le besoin réel (nombre et confiance mutuelle des participants, nécessité de résistance à la censure), pas par un effet de mode.
- Le problème de l'oracle rappelle qu'une blockchain garantit l'intégrité de ce qui y est inscrit, jamais la véracité de l'information au moment de sa saisie — un point central repris au chapitre 4.
- Une blockchain n'est pas une solution universelle : son usage doit être justifié par un besoin réel de décentralisation, pas adopté par principe, et se compare toujours à l'alternative d'une base de données traditionnelle.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
