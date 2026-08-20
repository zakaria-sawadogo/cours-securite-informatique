# Chapitre 4 — Attaques sur les fonctions de hachage

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/04-attaques-fonctions-hachage.txt){ target=_blank }

## 1. Rappel des propriétés attendues

Une fonction de hachage cryptographique doit garantir : la **résistance à la préimage** (impossible de retrouver un message à partir de son empreinte), la **résistance à la seconde préimage** (impossible de trouver un autre message ayant la même empreinte qu'un message donné), et la **résistance aux collisions** (impossible de trouver deux messages distincts ayant la même empreinte).

Ces trois propriétés sont hiérarchisées mais pas totalement indépendantes : une fonction qui ne résiste pas aux collisions n'est pas nécessairement vulnérable à la recherche de préimage (retrouver un message précis reste souvent plus coûteux que trouver deux messages arbitraires en collision), ce qui explique que MD5 ou SHA-1, bien que cassés pour les collisions, ne le soient pas au même degré pour la préimage — une nuance importante pour évaluer le risque réel dans un contexte d'usage donné (signature de document vs vérification d'intégrité simple).

## 2. Le paradoxe des anniversaires et l'attaque par collision

Pour une empreinte de *n* bits, trouver une collision par force brute « naïve » coûterait en théorie 2^n essais. Le **paradoxe des anniversaires** montre qu'il suffit en réalité d'environ 2^(n/2) essais pour trouver une collision avec une probabilité significative (par analogie : dans un groupe de 23 personnes, la probabilité que deux partagent un anniversaire dépasse 50 %, bien moins que les 365 attendus intuitivement).

### Justification mathématique du paradoxe

Pour un espace de *N* valeurs possibles (*N* = 365 pour les anniversaires, *N* = 2^n pour une empreinte de *n* bits), la probabilité qu'aucune collision n'apparaisse parmi *k* tirages aléatoires indépendants s'approxime, pour *k* petit devant *N*, par :

```
P(pas de collision) ≈ exp( − k² / (2N) )
```

La probabilité qu'une collision apparaisse dépasse donc 50 % dès que *k* ≈ 1,177 × √N, soit un ordre de grandeur en **racine carrée de N** — d'où le nombre d'essais en 2^(n/2) pour une empreinte de *n* bits (puisque N = 2^n, √N = 2^(n/2)). C'est cette croissance en racine carrée, bien plus lente que la croissance linéaire naïve attendue intuitivement, qui rend l'attaque par collision beaucoup plus praticable que l'attaque par préimage (qui reste, elle, en 2^n).

### Illustration pédagogique sur un espace réduit

```python
import random

def demo_paradoxe_anniversaires(bits_empreinte=16, essais_max=200000):
    """Démonstration pédagogique sur une empreinte volontairement réduite
    (16 bits, soit 65536 valeurs possibles) pour observer en pratique
    la croissance en racine carrée annoncée par la théorie. Ne pas
    confondre avec un hachage cryptographique réel : ceci est un
    tirage aléatoire uniforme simulé, à but strictement illustratif."""
    espace = 2 ** bits_empreinte
    vues = {}
    for tentative in range(1, essais_max + 1):
        empreinte_simulee = random.randrange(espace)
        if empreinte_simulee in vues:
            return tentative  # nombre de tirages avant la première collision
        vues[empreinte_simulee] = tentative
    return None  # pas de collision trouvée dans la limite fixée

# La théorie prédit une collision attendue autour de ~1.25 * sqrt(2^16) ≈ 320 tirages,
# très loin des 65536 valeurs de l'espace complet.
```

En exécutant cette simulation plusieurs fois, on observe empiriquement que la première collision survient typiquement après quelques centaines de tirages seulement — pas après des dizaines de milliers — ce qui illustre concrètement pourquoi la sécurité effective d'une fonction de hachage contre les collisions est de *n/2* bits et non *n* bits.

Conséquence directe : une fonction de hachage de *n* bits n'offre qu'une sécurité de *n/2* bits contre les collisions, ce qui explique pourquoi MD5 (128 bits, soit 64 bits de sécurité en collision — déjà faible en théorie, et en pratique bien pire à cause de faiblesses structurelles additionnelles) est aujourd'hui totalement cassé en pratique, et pourquoi SHA-1 (160 bits, soit 80 bits de sécurité théorique en collision) est déconseillé depuis la démonstration d'une collision pratique en 2017 (attaque « SHAttered »), obtenue bien plus rapidement que 2^80 grâce à des faiblesses structurelles de l'algorithme, pas seulement grâce au paradoxe des anniversaires générique.

## 3. Attaques historiques marquantes

- **MD5** : des faiblesses structurelles permettant de construire des collisions bien plus rapidement que la borne générique du paradoxe des anniversaires sont publiées par Xiaoyun Wang et ses coauteurs en 2004-2005 ; des collisions pratiques deviennent alors accessibles avec des moyens de calcul modestes, exploitées ensuite pour forger de faux certificats numériques (des chercheurs ont démontré en 2008 la possibilité de construire un faux certificat d'autorité de certification exploitant une collision MD5 à préfixe choisi, puis l'affaire du malware Flame en 2012 a confirmé l'exploitation réelle d'une technique de collision MD5 pour falsifier une signature de code).
- **SHA-1** : collision pratique démontrée par des chercheurs de Google et du CWI (centre de recherche néerlandais) en 2017 (attaque nommée « SHAttered »), accélérant la migration généralisée vers SHA-2/SHA-3 dans les navigateurs, systèmes de contrôle de version et infrastructures à clé publique.
- **SHA-2 (SHA-256, SHA-512) et SHA-3** : à ce jour, aucune attaque pratique ne remet en cause leur sécurité ; ce sont les standards recommandés. SHA-3, basé sur une construction fondamentalement différente de Merkle-Damgård (la construction en éponge, *sponge construction*, dérivée de l'algorithme Keccak), a été standardisé en partie pour offrir une alternative structurellement indépendante en cas de découverte future d'une faiblesse générique affectant les constructions Merkle-Damgård.

## 4. Attaque par extension de longueur (length extension attack)

Les fonctions basées sur la construction de Merkle-Damgård (MD5, SHA-1, SHA-2) sont vulnérables à l'attaque par extension de longueur : connaissant `H(message)` et la longueur de `message` (mais pas `message` lui-même), un attaquant peut calculer `H(message || padding || extension)` pour une extension de son choix, **sans connaître `message`**.

### Pourquoi cette attaque fonctionne : la structure interne de Merkle-Damgård

Une fonction de hachage Merkle-Damgård traite le message par blocs successifs : un état interne (accumulateur) est mis à jour bloc par bloc par une fonction de compression, et l'empreinte finale correspond simplement à la valeur de cet état interne après le dernier bloc (padding inclus). Autrement dit, `H(message)` **n'est rien d'autre que l'état interne complet** de l'algorithme à la fin du traitement — un attaquant qui connaît cette valeur dispose donc de tout ce qu'il faut pour reprendre le calcul là où il s'est arrêté et continuer à traiter des blocs supplémentaires, exactement comme le ferait le détenteur légitime du message original, sans jamais avoir besoin de connaître ce message.

```python
# Pseudo-code conceptuel illustrant le principe (pas une implémentation
# réelle d'un algorithme de hachage) :

def hash_merkle_damgard(message, etat_initial, fonction_compression):
    etat = etat_initial
    for bloc in decouper_en_blocs(ajouter_padding(message)):
        etat = fonction_compression(etat, bloc)
    return etat  # l'empreinte EST l'état interne final

def attaque_extension_longueur(empreinte_connue, longueur_message_original,
                                 extension, fonction_compression):
    """L'attaquant réutilise l'empreinte connue comme état de départ :
    il n'a jamais eu besoin de connaître le message original."""
    etat = empreinte_connue  # état interne récupéré tel quel
    # Il faut recréer le padding qu'aurait ajouté l'algorithme au
    # message original (dépend de sa longueur, connue ou devinée),
    # puis reprendre le traitement avec les blocs de l'extension.
    for bloc in decouper_en_blocs(extension):
        etat = fonction_compression(etat, bloc)
    return etat  # empreinte valide de (message_original || padding || extension)
```

Conséquence pratique : construire une authentification de message par simple concaténation `H(clé || message)` est dangereux — un attaquant peut forger un MAC valide pour un message étendu sans connaître la clé, en repartant directement de l'empreinte publiée. La construction **HMAC** (chapitre Cryptographie) a été spécifiquement conçue pour neutraliser cette attaque, en appliquant la fonction de hachage **deux fois** avec des clés dérivées différentes (`HMAC(K, m) = H((K' ⊕ opad) || H((K' ⊕ ipad) || m))`), de sorte que l'attaquant ne dispose jamais directement d'un état interne réutilisable correspondant à un secret.

## 5. Hachage de mots de passe : attaques et contre-mesures

| Attaque | Principe | Contre-mesure |
|---|---|---|
| Force brute / dictionnaire | tester des mots de passe candidats et comparer les empreintes | politique de mots de passe robustes, limitation du nombre d'essais |
| Table arc-en-ciel | tables précalculées d'empreintes pour inverser rapidement un hachage non salé | **salage** (valeur aléatoire unique par mot de passe, ajoutée avant hachage) |
| Calcul massivement parallèle (GPU/ASIC) | les fonctions de hachage génériques (SHA-256) sont très rapides à calculer, donc favorables à l'attaquant | fonctions **spécialement conçues pour être lentes** : bcrypt, scrypt, Argon2 (facteur de coût réglable) |

### Approfondissement : pourquoi la lenteur volontaire est une propriété de sécurité

Une fonction de hachage générique comme SHA-256 est conçue pour être **rapide**, une propriété désirable pour vérifier l'intégrité d'un fichier volumineux, mais désastreuse pour le hachage de mots de passe : un attaquant disposant de matériel parallèle (GPU, voire circuits dédiés ASIC) peut tester un très grand nombre de candidats par seconde. Les fonctions dédiées au hachage de mots de passe introduisent délibérément un coût :

- **bcrypt** : coût réglable par un facteur de travail exponentiel (chaque incrément double le temps de calcul), basé sur une variante de l'algorithme de chiffrement Blowfish.
- **scrypt** : ajoute une exigence de **mémoire** importante et réglable, ce qui pénalise spécifiquement les architectures matérielles massivement parallèles (GPU/ASIC), moins efficaces lorsqu'elles doivent aussi gérer beaucoup de mémoire par unité de calcul.
- **Argon2** (vainqueur de la compétition *Password Hashing Competition*, recommandé aujourd'hui par défaut) : généralise ce principe en un algorithme **résistant à la mémoire** (*memory-hard*) avec trois paramètres réglables indépendamment — coût en temps, coût en mémoire, et degré de parallélisme — permettant d'ajuster finement le compromis sécurité/performance selon le contexte de déploiement.

Le point commun de ces trois familles est de rendre le coût unitaire d'un essai de mot de passe suffisamment élevé pour qu'un attaquant, même équipé de matériel puissant, ne puisse tester qu'un nombre limité de candidats dans un temps raisonnable — contrairement à une fonction générique rapide, où le facteur limitant est presque uniquement la puissance de calcul brute disponible.

## 6. Méthodologie d'évaluation d'une fonction de hachage

1. Vérifier la taille de sortie (borne théorique de sécurité en collision = taille/2).
2. Vérifier l'absence d'attaques publiées de préimage/collision pratiques, et distinguer une faiblesse purement théorique (réduction de complexité en dessous de la borne générique, sans attaque praticable) d'une attaque réellement exploitable avec des ressources réalistes.
3. Vérifier le contexte d'usage : une fonction sûre pour l'intégrité de fichiers (SHA-256) ne l'est pas nécessairement pour le hachage de mots de passe sans adaptation (facteur de travail, salage) — un hachage rapide et une fonction dédiée lente répondent à des menaces différentes.
4. Vérifier, pour tout usage impliquant un secret (MAC), l'emploi d'une construction dédiée (HMAC) plutôt qu'une concaténation naïve `H(clé || message)`, vulnérable à l'extension de longueur pour les constructions Merkle-Damgård.

## À retenir

- Le paradoxe des anniversaires (P(collision) ≈ 1 − exp(−k²/2N)) divise par deux, en bits, la sécurité effective d'une fonction de hachage contre les collisions : *n* bits de sortie n'offrent que *n/2* bits de sécurité en collision.
- MD5 et SHA-1 sont cassés en pratique pour les collisions (bien plus rapidement que la borne générique du paradoxe des anniversaires, grâce à des faiblesses structurelles) et ne doivent plus être utilisés pour un usage sécuritaire ; SHA-2 et SHA-3 restent recommandés.
- L'attaque par extension de longueur exploite le fait que l'empreinte d'une fonction Merkle-Damgård EST son état interne final, réutilisable directement par un attaquant : c'est pourquoi HMAC, et non une simple concaténation `H(clé‖message)`, doit être utilisé pour authentifier des messages.
- Le hachage de mots de passe exige des fonctions dédiées (bcrypt, scrypt, Argon2), volontairement lentes, coûteuses en mémoire et salées — jamais une fonction générique seule (SHA-256 brut), rapide par conception et donc favorable à un attaquant équipé de matériel parallèle.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
