# Chapitre 2 — Structures de données : chaînage cryptographique et arbres de Merkle

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/02-structures-donnees-merkle.txt){ target=_blank }

## 1. Le chaînage cryptographique des blocs

Chaque bloc contient, dans son en-tête, l'**empreinte de hachage du bloc précédent**. Cette construction crée une dépendance en cascade : modifier le contenu d'un bloc ancien change son empreinte, ce qui invalide l'empreinte enregistrée dans le bloc suivant, et ainsi de suite jusqu'au sommet de la chaîne. Un attaquant souhaitant altérer un bloc ancien devrait donc recalculer (et, selon le mécanisme de consensus, refaire valider) tous les blocs suivants plus rapidement que le reste du réseau honnête — d'où le terme d'immutabilité « pratique » plutôt qu'absolue, et la recommandation d'attendre un nombre suffisant de confirmations avant de considérer une transaction comme définitive.

### Illustration pédagogique du chaînage

Le fragment Python suivant (purement pédagogique, très simplifié par rapport à une implémentation réelle) illustre le principe du chaînage sur une liste de blocs minimalistes, afin de rendre concrète l'idée de « dépendance en cascade » :

```python
import hashlib

def empreinte(bloc: dict) -> str:
    """Calcule l'empreinte d'un bloc à partir de ses champs (exemple pédagogique)."""
    contenu = f"{bloc['index']}{bloc['precedent']}{bloc['donnees']}{bloc['nonce']}"
    return hashlib.sha256(contenu.encode()).hexdigest()

# Construction d'une mini-chaîne de 3 blocs
genese = {"index": 0, "precedent": "0" * 64, "donnees": "bloc genese", "nonce": 0}
genese["empreinte"] = empreinte(genese)

bloc1 = {"index": 1, "precedent": genese["empreinte"], "donnees": "Alice paie Bob", "nonce": 0}
bloc1["empreinte"] = empreinte(bloc1)

bloc2 = {"index": 2, "precedent": bloc1["empreinte"], "donnees": "Bob paie Chloé", "nonce": 0}
bloc2["empreinte"] = empreinte(bloc2)

# Si l'on modifie les données du bloc1 après coup...
bloc1["donnees"] = "Alice paie Eve"        # falsification tentée
# ...son empreinte change, ce qui invalide le champ "precedent" enregistré dans bloc2 :
assert empreinte(bloc1) != bloc1["empreinte"]     # l'empreinte stockée ne correspond plus au contenu
assert bloc2["precedent"] != empreinte(bloc1)     # la cascade de rupture se propage vers l'avant
```

Cet exemple simplifié omet volontairement l'arbre de Merkle (une seule donnée par bloc) et le mécanisme de consensus (le `nonce` n'est pas ajusté pour satisfaire une difficulté) : il vise uniquement à rendre visible, en quelques lignes, la propriété de dépendance en cascade au cœur de l'immutabilité pratique.

## 2. Arbre de Merkle

Un arbre de Merkle organise un ensemble de données (les transactions d'un bloc) de façon à pouvoir résumer l'ensemble par une seule empreinte (la **racine de Merkle**) tout en permettant de prouver efficacement qu'une donnée particulière fait bien partie de l'ensemble, sans avoir à transmettre l'ensemble complet.

### Construction

1. Chaque transaction est hachée individuellement (feuilles de l'arbre).
2. Les empreintes sont regroupées deux par deux et hachées ensemble pour former le niveau supérieur.
3. Ce processus est répété jusqu'à obtenir une seule empreinte : la racine de Merkle, stockée dans l'en-tête du bloc.

Lorsque le nombre de feuilles à un niveau donné est impair, une convention doit être adoptée pour former les paires (par exemple dupliquer la dernière feuille) ; ce détail d'implémentation, anodin en apparence, a par le passé été source de subtilités de sécurité dans certaines implémentations réelles (risque de confusion entre deux ensembles de transactions différents produisant, par construction naïve, la même racine) — une illustration du principe général selon lequel les détails d'implémentation d'une primitive cryptographique ont une importance de sécurité directe, et non purement cosmétique.

### Exemple pédagogique de construction (Python)

```python
import hashlib

def h(x: bytes) -> bytes:
    return hashlib.sha256(x).digest()

def racine_merkle(transactions: list[bytes]) -> bytes:
    """Construit itérativement la racine de Merkle d'une liste de transactions
    (implémentation pédagogique, sans les optimisations d'une bibliothèque réelle)."""
    niveau = [h(tx) for tx in transactions]          # feuilles : hachage de chaque transaction
    if not niveau:
        return h(b"")
    while len(niveau) > 1:
        if len(niveau) % 2 == 1:
            niveau.append(niveau[-1])                 # convention : dupliquer la dernière feuille si nombre impair
        niveau = [h(niveau[i] + niveau[i + 1]) for i in range(0, len(niveau), 2)]
    return niveau[0]

transactions = [b"Alice->Bob:10", b"Bob->Chloe:4", b"Chloe->Alice:1", b"Alice->David:2"]
print(racine_merkle(transactions).hex())
```

### Preuve de Merkle (Merkle proof)

Pour prouver qu'une transaction `T` appartient à un bloc, il suffit de fournir `T` et la liste des empreintes « sœurs » sur le chemin vers la racine (de l'ordre de log₂(n) empreintes pour n transactions), plutôt que l'ensemble des n transactions. Cette propriété est essentielle pour les **clients légers (SPV — Simplified Payment Verification)**, qui vérifient qu'une transaction est incluse dans un bloc sans télécharger l'intégralité de la blockchain.

L'efficacité de cette preuve est ce qui rend la structure de Merkle si utile en pratique : pour un bloc contenant, par exemple, 2 048 transactions (2¹¹), une preuve d'inclusion ne nécessite que 11 empreintes intermédiaires (plus la racine déjà connue), soit un volume de données de l'ordre de quelques centaines d'octets, indépendamment du fait que le bloc entier pèse potentiellement plusieurs mégaoctets. C'est un exemple caractéristique de structure de données offrant une **preuve de taille logarithmique** pour une propriété portant sur un ensemble de taille linéaire — un compromis extrêmement favorable, comparé à la transmission de l'ensemble complet des transactions qu'exigerait une simple liste non structurée.

### Exemple pédagogique de vérification d'une preuve

```python
def verifier_preuve(feuille: bytes, preuve: list[tuple[bytes, str]], racine_attendue: bytes) -> bool:
    """preuve est une liste de tuples (empreinte_soeur, position) où position vaut
    'gauche' ou 'droite', indiquant de quel côté placer l'empreinte sœur (exemple pédagogique)."""
    courant = h(feuille)
    for empreinte_soeur, position in preuve:
        if position == "gauche":
            courant = h(empreinte_soeur + courant)
        else:
            courant = h(courant + empreinte_soeur)
    return courant == racine_attendue
```

Ce court exemple illustre un point pédagogique important : la vérification d'une preuve de Merkle ne nécessite que la fonction de hachage et la preuve elle-même — aucune donnée de l'arbre entier n'est requise, ce qui permet à un client léger de vérifier une transaction avec des ressources minimales (mémoire, bande passante).

## 3. Pourquoi cette double structure (chaîne de blocs + arbre par bloc) ?

- Le **chaînage entre blocs** garantit l'intégrité de l'ordre et de l'historique global.
- L'**arbre de Merkle au sein d'un bloc** garantit l'intégrité de l'ensemble des transactions d'un bloc tout en permettant des vérifications partielles efficaces.

Cette architecture illustre directement l'usage des fonctions de hachage (module *Cryptographie*, chapitre 3) comme brique de construction pour des propriétés de plus haut niveau (intégrité globale d'un registre distribué).

### Propriétés cryptographiques requises de la fonction de hachage

Cette double structure ne fonctionne correctement que si la fonction de hachage utilisée possède certaines propriétés étudiées en détail dans le module *Cryptographie* :

- **résistance aux collisions** : il doit être calculatoirement infaisable de trouver deux entrées distinctes produisant la même empreinte — sans cela, un attaquant pourrait substituer une transaction frauduleuse à une transaction légitime tout en conservant la même racine de Merkle ;
- **résistance à la préimage** : il doit être infaisable, à partir d'une empreinte, de retrouver une entrée qui la produit — propriété moins critique ici que pour le stockage de mots de passe, mais toujours pertinente ;
- **effet avalanche** : une modification infime de l'entrée (un seul bit) doit produire une empreinte de sortie complètement différente, sans corrélation apparente avec l'entrée d'origine — c'est cette propriété qui garantit qu'aucune modification, même mineure, d'une transaction ancienne ne peut passer inaperçue dans la racine de Merkle ni dans le chaînage des blocs.

Bitcoin et Ethereum utilisent des fonctions de la famille SHA-2/SHA-3/Keccak, étudiées en détail dans le chapitre correspondant du module *Cryptographie* ; ce chapitre présuppose cette base et se concentre sur l'usage qui en est fait pour structurer un registre distribué.

## 4. Identifiants et adresses

- Une **transaction** est identifiée par l'empreinte de son propre contenu (le « hash de transaction »).
- Une **adresse** (compte) dérive généralement d'une clé publique (elle-même souvent hachée pour former une adresse plus courte), reliant directement ce chapitre au module *Cryptographie* (signatures numériques, chapitre 5) : chaque transaction est signée par la clé privée correspondant à l'adresse d'origine, et cette signature est vérifiable par tout nœud du réseau.

### Modèle UTXO contre modèle de compte

Deux grandes familles de modèles de représentation des soldes coexistent dans les blockchains actuelles, avec des implications directes sur la façon dont les transactions sont structurées et vérifiées :

| Modèle | Principe | Exemple |
|---|---|---|
| **UTXO** (*Unspent Transaction Output*) | chaque transaction consomme un ou plusieurs « jetons » de sortie non dépensés issus de transactions antérieures, et en produit de nouveaux ; le solde d'une adresse est la somme des UTXO qui lui sont associés | Bitcoin |
| **Modèle de compte** | chaque adresse possède un solde global directement mis à jour (comme un compte bancaire classique), analogue au champ `soldes` de l'exemple de smart contract présenté au chapitre 4 | Ethereum |

Le modèle UTXO offre un parallélisme naturel pour la validation (des UTXO indépendants peuvent être vérifiés séparément) et facilite le respect de la confidentialité (une nouvelle adresse peut être générée pour chaque transaction reçue), mais rend plus complexe la représentation d'un état global mutable — d'où le choix du modèle de compte pour Ethereum, mieux adapté à l'exécution de smart contracts à état persistant (chapitre 4).

## 5. Variantes et structures alternatives

- **DAG (Directed Acyclic Graph)** : certaines architectures (ex. IOTA) remplacent la chaîne linéaire de blocs par un graphe orienté acyclique de transactions, visant une meilleure scalabilité, au prix d'autres compromis de sécurité et de maturité.
- **État global (state trie)** : les blockchains à smart contracts (Ethereum, chapitre 4) maintiennent, en plus de l'historique des transactions, une structure arborescente représentant l'état courant de tous les comptes et contrats, également authentifiée cryptographiquement (racine d'état incluse dans chaque bloc).

Ethereum va plus loin que le simple couple chaînage + arbre de Merkle des transactions : chaque bloc y contient en réalité **trois racines distinctes** dans son en-tête — la racine de Merkle des transactions du bloc, une racine résumant l'état global de tous les comptes et contrats après exécution du bloc (le *state trie*, techniquement un arbre de Merkle-Patricia, une variante combinant arbre de Merkle et arbre préfixe pour un accès et une mise à jour efficaces), et une racine résumant les reçus d'exécution des transactions. Cette architecture plus riche prépare directement le chapitre 4, où l'état global inclut non seulement des soldes mais aussi le code et les variables persistantes des smart contracts déployés.

### Limites de la structure de Merkle simple

Il est utile, à ce stade, de noter une limite pédagogique de l'arbre de Merkle « statique » tel que présenté en section 2 : il est optimisé pour prouver l'appartenance d'un élément à un ensemble **figé** au moment de la construction de l'arbre. Or l'état global d'une blockchain à smart contracts change à chaque bloc (nouveaux soldes, nouvelles variables de contrat). Les arbres de Merkle-Patricia (et plus généralement les *arbres de Merkle persistants* ou *versionnés*) répondent à ce besoin en permettant de dériver efficacement, à chaque mise à jour, une nouvelle racine sans reconstruire l'arbre entier depuis zéro — un raffinement d'ingénierie que ce cours ne détaille pas davantage mais qu'il est utile de savoir identifier pour la suite du module.

## À retenir

- Le chaînage cryptographique entre blocs et l'arbre de Merkle au sein d'un bloc sont deux mécanismes complémentaires garantissant respectivement l'intégrité de l'historique global et celle du contenu de chaque bloc.
- Une preuve de Merkle permet de vérifier l'inclusion d'une transaction en un nombre d'étapes logarithmique par rapport au nombre de transactions du bloc, sans télécharger l'intégralité de la blockchain — fondement des clients légers (SPV).
- Le bon fonctionnement de cette architecture dépend de propriétés cryptographiques précises de la fonction de hachage utilisée (résistance aux collisions, effet avalanche), étudiées en détail dans le module *Cryptographie*.
- Le modèle UTXO (Bitcoin) et le modèle de compte (Ethereum) représentent différemment les soldes et ont des implications directes sur le parallélisme de validation et la facilité à représenter un état mutable complexe.
- Les blockchains à smart contracts étendent la structure vue dans ce chapitre avec une racine d'état global (state trie), authentifiant non seulement les transactions mais tout l'état courant du réseau.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
