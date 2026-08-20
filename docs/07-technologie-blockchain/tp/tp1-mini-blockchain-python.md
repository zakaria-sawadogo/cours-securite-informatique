# TP1 — Implémentation d'une mini-blockchain en Python (2h)

## Objectifs
- Implémenter les structures de données fondamentales d'une blockchain pour en comprendre le fonctionnement interne.

## Exercice 1 — Structure d'un bloc

En Python, définir une classe `Bloc` avec les attributs : `index`, `horodatage`, `transactions` (liste de chaînes simples), `hash_precedent`, `nonce`, et une méthode `calculer_hash()` utilisant `hashlib.sha256` sur la concaténation des attributs pertinents.

```python
import hashlib
import time

class Bloc:
    def __init__(self, index, transactions, hash_precedent):
        self.index = index
        self.horodatage = time.time()
        self.transactions = transactions
        self.hash_precedent = hash_precedent
        self.nonce = 0
        self.hash = self.calculer_hash()

    def calculer_hash(self):
        contenu = f"{self.index}{self.horodatage}{self.transactions}{self.hash_precedent}{self.nonce}"
        return hashlib.sha256(contenu.encode()).hexdigest()
```

## Exercice 2 — Chaînage des blocs

Implémenter une classe `Blockchain` gérant une liste de blocs, avec une méthode `creer_bloc_genese()` (premier bloc, `hash_precedent = "0"`) et une méthode `ajouter_bloc(transactions)` créant un nouveau bloc référençant le hash du dernier bloc de la chaîne.

## Exercice 3 — Vérification d'intégrité

Implémenter une méthode `est_valide()` qui parcourt la chaîne et vérifie, pour chaque bloc : que son `hash` correspond bien à `calculer_hash()` recalculé, et que son `hash_precedent` correspond bien au `hash` du bloc précédent. Tester en modifiant volontairement une transaction d'un bloc intermédiaire et en vérifiant que `est_valide()` détecte l'altération.

## Exercice 4 — Arbre de Merkle simplifié

Implémenter une fonction `racine_merkle(transactions)` qui, à partir d'une liste de transactions (chaînes de caractères), construit récursivement l'arbre de Merkle (hachage des transactions deux par deux jusqu'à une racine unique) et renvoie la racine. Intégrer cette racine dans le calcul du hash du bloc (exercice 1) à la place de la concaténation brute des transactions.

## À rendre

Le script Python complet, avec une démonstration : création d'une chaîne de 5 blocs, vérification de validité, altération d'un bloc et nouvelle vérification montrant la détection de l'altération.
