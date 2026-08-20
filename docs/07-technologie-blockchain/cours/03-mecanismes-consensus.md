# Chapitre 3 — Mécanismes de consensus

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/03-mecanismes-consensus.txt){ target=_blank }

## 1. Pourquoi un mécanisme de consensus ?

Dans un réseau décentralisé sans autorité centrale, il faut un mécanisme permettant à des participants potentiellement nombreux, anonymes et parfois malveillants de s'accorder sur un état unique et cohérent du registre (quel bloc suit quel bloc), tout en résistant à des tentatives de manipulation. C'est une instance du problème classique dit **des généraux byzantins** en informatique distribuée.

### Le problème des généraux byzantins

Formulé initialement en informatique distribuée classique, ce problème imagine plusieurs généraux d'une armée assiégeant une ville, communiquant uniquement par messagers (dont certains peuvent être interceptés, retardés, ou porter de faux messages), devant néanmoins se coordonner sur une décision commune (attaquer ou battre en retraite) malgré la présence possible de généraux traîtres parmi eux. La difficulté centrale est double : il faut à la fois **s'accorder** sur une décision commune (tous les généraux honnêtes doivent choisir la même action) et le faire **malgré des messages potentiellement falsifiés ou perdus**, sans savoir a priori qui, parmi les participants, est malveillant.

Transposé à une blockchain publique, chaque nœud du réseau joue le rôle d'un général : les « messages » sont les blocs et transactions propagés sur le réseau pair-à-pair, et les « généraux traîtres » sont les nœuds malveillants tentant de faire accepter par le reste du réseau une version falsifiée de l'historique (par exemple pour dépenser deux fois la même unité de monnaie). La différence essentielle avec le problème académique classique est l'échelle : une blockchain publique doit résister à un nombre de participants potentiellement très grand, anonymes, pouvant rejoindre ou quitter le réseau à tout moment (contrairement aux formulations académiques classiques qui supposent un ensemble de participants fixe et connu à l'avance) — c'est cette contrainte supplémentaire qui a motivé des mécanismes de consensus spécifiques comme la preuve de travail.

### Vivacité et sûreté : deux propriétés à concilier

Tout protocole de consensus distribué doit, en théorie, satisfaire deux propriétés dont la combinaison est difficile dans un réseau asynchrone et adversarial :

- **sûreté (safety)** : le protocole ne doit jamais faire converger des nœuds honnêtes vers deux états définitifs contradictoires (pas de double dépense acceptée) ;
- **vivacité (liveness)** : le protocole doit continuer à progresser (produire de nouveaux blocs) malgré la présence de nœuds défaillants ou malveillants, et ne pas rester bloqué indéfiniment.

Les différents mécanismes présentés dans ce chapitre opèrent des compromis différents entre ces deux propriétés, sous des hypothèses différentes sur le réseau (synchrone ou non) et sur la proportion de participants malveillants tolérée.

## 2. Preuve de travail (Proof of Work, PoW)

Principe : pour proposer un nouveau bloc, un participant (« mineur ») doit résoudre un défi calculatoire coûteux — trouver une valeur (le *nonce*) telle que l'empreinte de hachage du bloc satisfasse une condition de difficulté (par exemple, commencer par un certain nombre de zéros). Ce calcul est intentionnellement coûteux à réaliser mais trivial à vérifier par les autres nœuds.

- **Sécurité** : falsifier l'historique nécessiterait de disposer d'une puissance de calcul supérieure à celle du reste du réseau honnête combiné (« attaque des 51 % »), un coût économique dissuasif pour un réseau suffisamment décentralisé et puissant.
- **Limite majeure** : consommation énergétique très importante (Bitcoin), motivant la recherche d'alternatives.

### Illustration pédagogique de la preuve de travail

Le fragment Python suivant illustre, de façon très simplifiée, la recherche d'un nonce satisfaisant une condition de difficulté arbitraire (nombre de zéros en début d'empreinte) — un exercice de démonstration, sans commune mesure avec la difficulté réelle du réseau Bitcoin :

```python
import hashlib

def miner(donnees_bloc: str, difficulte: int) -> tuple[int, str]:
    """Recherche par force brute un nonce tel que l'empreinte du bloc commence
    par `difficulte` zéros hexadécimaux (exemple pédagogique très simplifié)."""
    prefixe_cible = "0" * difficulte
    nonce = 0
    while True:
        contenu = f"{donnees_bloc}{nonce}".encode()
        empreinte = hashlib.sha256(contenu).hexdigest()
        if empreinte.startswith(prefixe_cible):
            return nonce, empreinte
        nonce += 1

nonce, empreinte = miner("Alice paie Bob 10 unités", difficulte=5)
print(f"Nonce trouvé : {nonce}, empreinte : {empreinte}")
```

Ce court exemple permet d'illustrer deux propriétés essentielles de la preuve de travail : la recherche du nonce (le « minage ») est coûteuse (il faut essayer de nombreuses valeurs avant de trouver une empreinte satisfaisant la condition, le coût moyen croissant exponentiellement avec le paramètre `difficulte`), tandis que la **vérification** d'une solution proposée est quasi instantanée (il suffit de recalculer une seule empreinte). C'est cette asymétrie coût-de-production / coût-de-vérification qui est au cœur de tous les mécanismes de preuve de travail.

### Ajustement dynamique de la difficulté

Un aspect souvent sous-estimé : la difficulté n'est pas figée, mais réajustée périodiquement par le protocole (toutes les 2016 blocs pour Bitcoin, soit environ deux semaines compte tenu du temps de bloc cible de dix minutes) afin de maintenir un rythme moyen de production de blocs approximativement constant, indépendamment des variations de la puissance de calcul totale déployée par les mineurs du réseau. Ce mécanisme d'auto-régulation est essentiel : sans lui, une augmentation de la puissance de calcul totale (par exemple, l'arrivée massive de nouveaux mineurs) accélérerait mécaniquement la production de blocs, avec des conséquences sur la sécurité (moins de temps pour la propagation réseau, augmentant le risque de forks accidentels) et sur l'économie du réseau (rythme d'émission de nouvelles unités monétaires).

### Attaque des 51 % : mécanique et limites pratiques

Une attaque dite « des 51 % » consiste, pour un attaquant disposant d'une majorité de la puissance de calcul du réseau, à construire en secret une chaîne alternative plus longue que la chaîne publique (par exemple en omettant une transaction qu'il a lui-même émise), puis à la diffuser d'un coup pour que le réseau, suivant la règle de la chaîne la plus longue, l'adopte à la place de la chaîne légitime — permettant potentiellement une double dépense. Il est important de noter les limites pratiques de cette attaque, même en cas de réussite technique : elle ne permet pas de créer des unités monétaires à partir de rien, ni de voler des fonds d'adresses dont l'attaquant ne détient pas la clé privée (les signatures restent vérifiées normalement, chapitre 2) ; elle permet uniquement de réorganiser l'ordre récent des transactions et, potentiellement, d'annuler ses propres transactions récentes déjà confirmées. Pour un réseau suffisamment décentralisé et disposant d'une puissance de calcul totale élevée, le coût d'acquisition ou de location d'une telle puissance de calcul majoritaire, ainsi que le risque de dévaluer la cryptomonnaie que l'attaquant espère précisément exploiter, constituent des freins économiques dissuasifs — un raisonnement de sécurité économique plutôt que purement cryptographique.

## 3. Preuve d'enjeu (Proof of Stake, PoS)

Principe : le droit de proposer et de valider un bloc est attribué (souvent par tirage pondéré) en fonction de la quantité de cryptomonnaie qu'un participant a « mise en jeu » (staked), plutôt qu'en fonction de sa puissance de calcul.

- **Sécurité** : un participant malveillant risque de perdre (« slashing ») une partie de sa mise s'il tente de valider des blocs contradictoires ou invalides, ce qui aligne son intérêt économique avec le comportement honnête.
- **Avantage** : consommation énergétique très inférieure à PoW (Ethereum est passé de PoW à PoS en 2022, réduisant sa consommation énergétique de plus de 99 %).
- **Débat en cours** : risque de centralisation accrue au profit des plus gros détenteurs (« les riches deviennent validateurs, les validateurs gagnent des récompenses, les riches s'enrichissent davantage »).

### Mécanique simplifiée d'un tirage pondéré par la mise

Le pseudo-code suivant illustre, de façon pédagogique et très simplifiée, l'idée d'un tirage au sort pondéré par la mise, sans reproduire les détails effectifs (souvent plus complexes, impliquant des comités de validateurs et des tours multiples) des implémentations réelles :

```python
import random

def choisir_validateur(validateurs: dict[str, float]) -> str:
    """validateurs : dictionnaire {adresse: montant mis en jeu}.
    Retourne l'adresse choisie pour proposer le prochain bloc,
    avec une probabilité proportionnelle à sa mise (exemple pédagogique)."""
    adresses = list(validateurs.keys())
    poids = list(validateurs.values())
    return random.choices(adresses, weights=poids, k=1)[0]

validateurs = {"0xAlice": 320, "0xBob": 80, "0xChloe": 600}
print(choisir_validateur(validateurs))   # Chloé a la probabilité la plus élevée d'être choisie
```

### Le slashing : aligner incitations et sécurité

Le mécanisme de *slashing* (confiscation partielle ou totale de la mise) est le pivot économique de la sécurité en preuve d'enjeu. Il cible typiquement deux catégories de comportements fautifs, détectables algorithmiquement et de façon vérifiable par tout le réseau :

- **double signature (equivocation)** : un validateur signe deux blocs concurrents à la même hauteur de chaîne, une preuve cryptographique irréfutable de tentative de fraude (les deux signatures, publiées, suffisent à démontrer la faute) ;
- **inactivité prolongée** : un validateur hors ligne qui ne participe plus à la validation, pénalisé (généralement plus légèrement qu'une double signature) pour maintenir la vivacité du réseau.

Un point pédagogique important à souligner aux étudiants : contrairement à la preuve de travail, où le coût de l'attaque est un coût **externe** au système (l'électricité et le matériel consommés, perdus qu'il y ait fraude ou non), le coût du slashing est un coût **interne** directement prélevé sur l'actif que l'attaquant détenait dans le système — un changement de nature du mécanisme de dissuasion, pas seulement de son intensité.

### Variantes et raffinements du PoS

Plusieurs familles de conception existent au sein même de la preuve d'enjeu, chacune répondant différemment au compromis vivacité/sûreté/décentralisation :

- **PoS délégué (DPoS)** : les détenteurs de jetons votent pour un nombre restreint de délégués chargés de valider les blocs en leur nom, améliorant potentiellement le débit de transactions au prix d'une décentralisation politique réduite (peu de délégués effectifs) ;
- **PoS liquide** : les détenteurs peuvent déléguer leur pouvoir de validation à un tiers tout en conservant la propriété de leurs jetons et la possibilité de retirer leur délégation à tout moment ;
- **Comités de validateurs par époque** : plutôt qu'un tirage individuel à chaque bloc, un sous-ensemble de validateurs est désigné pour une période donnée (une « époque »), réduisant la surface d'attaque à un instant donné.

## 4. Tolérance byzantine pratique (PBFT et variantes)

Utilisée typiquement dans les blockchains à permission (nombre de participants limité et identifié) : les nœuds valident un bloc par un mécanisme de vote explicite, tolérant qu'une fraction des participants (typiquement jusqu'à un tiers) soit défaillante ou malveillante, avec une finalité de validation quasi immédiate (contrairement à PoW où la certitude augmente progressivement avec les confirmations).

### Pourquoi la limite d'un tiers ?

Cette limite n'est pas arbitraire : elle découle d'un résultat classique de la théorie de la tolérance aux fautes byzantines, selon lequel un protocole de consensus déterministe dans un réseau partiellement synchrone ne peut garantir simultanément la sûreté et la vivacité que si le nombre de participants défaillants ou malveillants reste strictement inférieur à un tiers du total des participants. Intuitivement, avec exactement un tiers de participants byzantins, un attaquant pourrait, dans le pire des cas, faire pencher artificiellement un vote en se positionnant tantôt d'un côté tantôt de l'autre selon ce qui sert sa fraude, empêchant les nœuds honnêtes de distinguer de façon fiable la majorité légitime. Ce résultat, bien antérieur à la blockchain, explique pourquoi les protocoles BFT classiques (comme PBFT, *Practical Byzantine Fault Tolerance*, proposé à la fin des années 1990) et leurs dérivés utilisés dans les blockchains à permission modernes retiennent systématiquement ce seuil.

### Déroulement schématique d'un tour de vote BFT

À titre pédagogique, un protocole de type PBFT se déroule typiquement en plusieurs phases de communication entre validateurs, avant qu'un bloc ne soit considéré comme définitivement validé :

1. **Proposition** : un validateur désigné (« le leader » du tour courant) diffuse un bloc candidat à l'ensemble des validateurs.
2. **Pré-vote** : chaque validateur diffuse aux autres son accord (ou désaccord) sur le bloc proposé, après vérification de sa validité.
3. **Vote de validation** : si un validateur observe qu'une majorité qualifiée (généralement plus des deux tiers) a pré-voté en faveur du même bloc, il diffuse à son tour un vote de validation.
4. **Finalisation** : dès qu'un validateur observe qu'une majorité qualifiée de votes de validation a été atteinte pour ce bloc, il le considère comme définitivement finalisé — sans possibilité de réorganisation ultérieure, contrairement à la finalité probabiliste de la preuve de travail.

Ce déroulement en plusieurs tours de communication explique pourquoi les protocoles BFT classiques passent difficilement à l'échelle au-delà de quelques dizaines ou centaines de validateurs (le volume de messages échangés croît rapidement avec le nombre de participants), ce qui justifie leur usage privilégié dans des contextes à permission où le nombre de validateurs reste maîtrisé, plutôt que dans un réseau public ouvert à un nombre non borné de participants anonymes.

## 5. Comparaison synthétique

| Mécanisme | Coût de sécurité | Consommation énergétique | Finalité | Exemple |
|---|---|---|---|---|
| PoW | puissance de calcul | très élevée | probabiliste (croît avec les confirmations) | Bitcoin |
| PoS | capital économique en jeu | faible | souvent quasi immédiate selon l'implémentation | Ethereum (depuis 2022) |
| BFT | identité/réputation des validateurs | faible | immédiate | réseaux à permission, certaines blockchains d'entreprise |

## 6. Fork et résolution de conflits

Lorsque deux blocs valides sont proposés presque simultanément (situation normale dans un réseau distribué), une **fourche (fork)** temporaire apparaît ; les nœuds convergent généralement vers la chaîne la plus longue (PoW) ou celle validée par le plus de participants (PoS) selon la règle de choix définie par le protocole. Un **hard fork** délibéré (changement de règles incompatible) peut aussi survenir lors d'une mise à jour majeure ou d'un désaccord de gouvernance dans la communauté.

### Fork accidentel contre fork délibéré

Il est pédagogiquement utile de distinguer clairement deux situations souvent confondues par les étudiants découvrant le sujet :

- le **fork accidentel (temporaire)** est un phénomène normal et attendu du fonctionnement d'un réseau distribué : deux mineurs ou validateurs trouvent, à quelques secondes d'intervalle, un bloc valide chacun sur leur propre vue de la chaîne ; le réseau se scinde brièvement en deux groupes de nœuds ayant reçu l'un ou l'autre bloc en premier, avant de converger automatiquement vers une seule chaîne dès que le bloc suivant est ajouté à l'une des deux branches (règle de la chaîne la plus longue, ou son équivalent en PoS) ; les transactions du bloc écarté (« orphelin ») retournent en attente et seront généralement incluses dans un bloc ultérieur ;
- le **hard fork délibéré** est un changement volontaire et incompatible des règles du protocole (par exemple, modifier la limite de taille des blocs, ou corriger une faille de sécurité découverte après coup) : les nœuds n'ayant pas mis à jour leur logiciel continuent de suivre les anciennes règles, ce qui peut faire naître **deux chaînes durablement distinctes** si une partie de la communauté refuse la mise à jour — un cas de figure qui s'est déjà produit dans l'histoire de plusieurs blockchains publiques majeures, y compris à la suite de désaccords de gouvernance profonds sur la marche à suivre après un incident de sécurité important. Un **soft fork**, par contraste, restreint les règles de validité de façon rétrocompatible (un bloc valide selon les nouvelles règles reste valide selon les anciennes), ce qui limite le risque de scission durable du réseau.

## À retenir

- Le mécanisme de consensus répond à un problème classique d'informatique distribuée (le problème des généraux byzantins) : accord dans un réseau potentiellement malveillant, sans autorité centrale, en conciliant sûreté (pas de décisions contradictoires) et vivacité (progression continue).
- PoW sécurise par un coût calculatoire externe au système (électricité, matériel) mais consomme beaucoup d'énergie, avec un mécanisme d'ajustement dynamique de la difficulté ; PoS sécurise par un risque économique interne et direct (slashing d'une mise déjà engagée dans le système) avec une empreinte énergétique bien moindre.
- La tolérance byzantine pratique (BFT) offre une finalité de validation immédiate et déterministe, au prix d'un passage à l'échelle plus difficile au-delà de quelques centaines de validateurs, ce qui la réserve typiquement aux réseaux à permission.
- Un fork accidentel (temporaire, résolu automatiquement par la règle de choix de chaîne) doit être distingué d'un hard fork délibéré (changement de règles incompatible, pouvant scinder durablement une communauté en deux chaînes distinctes).
- Le choix du mécanisme de consensus détermine directement les propriétés de sécurité, de décentralisation réelle et de coût du réseau — un compromis, pas une hiérarchie absolue de qualité, à mettre en regard du trilemme scalabilité/sécurité/décentralisation développé au chapitre 6.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
