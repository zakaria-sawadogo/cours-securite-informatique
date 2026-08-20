# Chapitre 6 — Attaques par canaux auxiliaires et attaques pratiques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/06-attaques-canaux-auxiliaires.txt){ target=_blank }

## 1. Principe des attaques par canaux auxiliaires (side-channel)

Une attaque par canal auxiliaire n'exploite pas de faiblesse mathématique de l'algorithme, mais une **information physique fuitée par son implémentation** : temps d'exécution, consommation électrique, rayonnement électromagnétique, comportement du cache processeur, messages d'erreur. Ces attaques rappellent que la sécurité d'un système cryptographique dépend autant de son implémentation que de sa conception mathématique.

On distingue généralement deux grandes familles d'attaques par canal auxiliaire :

- les attaques **passives**, où l'attaquant se contente d'observer une fuite (temps, consommation, rayonnement) sans interagir autrement avec le système ;
- les attaques **actives**, où l'attaquant perturbe volontairement le fonctionnement du dispositif (variation de tension, d'horloge, impulsion laser) pour provoquer une erreur de calcul exploitable — c'est le principe des **attaques par injection de faute** (voir section 7).

## 2. Attaques temporelles (timing attacks)

Si le temps d'exécution d'une opération cryptographique dépend des bits secrets manipulés (ex. une comparaison de mot de passe qui s'arrête au premier octet différent), un attaquant mesurant précisément ce temps peut déduire de l'information sur le secret, octet par octet.

**Contre-mesure** : comparaisons en temps constant (*constant-time comparison*), qui parcourent systématiquement toute la donnée indépendamment du résultat intermédiaire.

### Illustration : comparaison vulnérable contre comparaison en temps constant

```python
# Comparaison VULNÉRABLE : s'arrête dès le premier octet différent,
# donc le temps d'exécution dépend du nombre d'octets corrects trouvés
# avant la première différence -> fuite exploitable par mesure de temps.
def comparaison_vulnerable(secret, tentative):
    if len(secret) != len(tentative):
        return False
    for a, b in zip(secret, tentative):
        if a != b:
            return False   # sortie anticipée : fuite temporelle
    return True

# Comparaison en TEMPS CONSTANT : parcourt systématiquement toute la
# donnée, accumule les différences par XOR, et ne conclut qu'à la fin.
def comparaison_temps_constant(secret, tentative):
    if len(secret) != len(tentative):
        return False
    difference_accumulee = 0
    for a, b in zip(secret, tentative):
        difference_accumulee |= a ^ b   # pas de branchement dépendant du secret
    return difference_accumulee == 0
```

En pratique, il ne suffit pas d'écrire un tel code : le compilateur, le processeur (prédiction de branchement, exécution spéculative) et le cache peuvent réintroduire des variations de temps subtiles. C'est pourquoi les bibliothèques cryptographiques sérieuses fournissent des primitives de comparaison en temps constant testées et auditées, plutôt que de laisser chaque développeur réimplémenter la sienne.

## 3. Attaques par oracle de padding (padding oracle)

Décrites en détail en TP3 : si un système révèle (même indirectement, par un code d'erreur différent ou un temps de réponse différent) si un texte chiffré déchiffré présente un padding valide selon un schéma comme PKCS#7 en mode CBC, un attaquant peut déchiffrer progressivement l'intégralité du message sans connaître la clé, en interrogeant le système bloc par bloc, octet par octet. Cette classe d'attaque (Vaudenay, 2002) a compromis en pratique de nombreuses implémentations TLS/SSL (ex. attaque *POODLE*, 2014).

### Principe algorithmique simplifié de l'attaque

L'attaque exploite la structure du mode CBC : le déchiffrement d'un bloc *Cᵢ* dépend du bloc chiffré précédent *Cᵢ₋₁* par un XOR final (`clairᵢ = déchiffrement_bloc(Cᵢ) ⊕ Cᵢ₋₁`). En modifiant délibérément des octets du bloc précédent (que l'attaquant contrôle entièrement s'il forge son propre texte chiffré, ou dont il connaît la valeur dans un texte chiffré intercepté) et en observant si le serveur signale un padding valide ou invalide, l'attaquant peut déduire, octet par octet en partant de la fin du bloc, la valeur du clair intermédiaire :

1. Modifier le dernier octet du bloc précédent et soumettre le texte chiffré modifié au système.
2. Observer la réponse : padding valide (le dernier octet du clair déchiffré vaut alors `0x01`, un padding PKCS#7 minimal) ou invalide.
3. En essayant les 256 valeurs possibles de l'octet modifié, retrouver la valeur exacte de l'octet de clair intermédiaire correspondant, par un simple calcul de XOR.
4. Répéter en remontant octet par octet vers le début du bloc, en ajustant les octets déjà retrouvés pour maintenir un padding cohérent (`0x02 0x02`, puis `0x03 0x03 0x03`, etc.), jusqu'à déchiffrer le bloc entier.
5. Répéter l'opération pour chaque bloc du message.

Le coût de cette attaque est de l'ordre de 256 requêtes par octet dans le pire cas, soit un nombre de requêtes linéaire en la taille du message — largement praticable dès lors que l'oracle (le système qui distingue padding valide/invalide) est accessible et interrogeable un grand nombre de fois, ce qui explique la gravité de cette classe d'attaque une fois découverte dans un système réel.

## 4. Attaques par analyse de consommation électrique et électromagnétique

- **SPA (Simple Power Analysis)** : observation directe de la trace de consommation, révélant parfois la séquence d'opérations effectuées (ex. distinguer un carré d'une multiplication en exponentiation modulaire — une différence de motif visible directement sur une seule trace, sans traitement statistique).
- **DPA (Differential Power Analysis)** : analyse statistique de nombreuses traces de consommation pour un grand nombre d'entrées, permettant de retrouver des bits de clé même en présence de bruit. Le principe consiste à formuler une hypothèse sur une portion de la clé, à prédire la consommation attendue sous cette hypothèse pour chaque trace collectée, puis à corréler statistiquement cette prédiction à la consommation réellement mesurée : l'hypothèse correcte se distingue des hypothèses incorrectes par une corrélation significativement plus élevée, même lorsque le bruit de mesure rendrait toute lecture directe d'une trace individuelle impossible.
- **CPA (Correlation Power Analysis)** : raffinement statistique de la DPA, utilisant un coefficient de corrélation (par exemple celui de Pearson) plutôt qu'une simple différence de moyennes, ce qui améliore l'efficacité de l'attaque avec moins de traces nécessaires.

Ces attaques concernent principalement des dispositifs physiques (cartes à puce, modules matériels de sécurité, objets connectés) où l'attaquant a un accès physique ou de proximité.

### Contre-mesures classiques contre l'analyse de consommation

- **Masquage (masking)** : décomposer chaque valeur secrète manipulée en plusieurs parts aléatoires dont la combinaison reconstitue la valeur réelle, de sorte qu'aucune part individuelle ne corrèle avec le secret.
- **Ajout de bruit matériel** : introduire des variations aléatoires de consommation ou d'horloge pour dégrader le rapport signal/bruit exploité par l'attaquant.
- **Équilibrage des opérations** : concevoir les circuits pour que les opérations sur un bit à 0 et un bit à 1 consomment une énergie similaire, réduisant la fuite exploitable à la source.

## 5. Attaques par cache (cache-timing attacks)

Exploitent les différences de temps d'accès entre données présentes ou absentes du cache processeur. Des implémentations logicielles d'AES utilisant des tables précalculées (T-tables) indexées par des bits de la clé ont ainsi été attaquées avec succès, motivant l'adoption d'instructions matérielles dédiées (AES-NI) exécutant le chiffrement en temps constant indépendamment de la clé.

### Deux techniques classiques d'attaque par cache

- **Prime+Probe** : l'attaquant remplit (« prime ») certaines lignes de cache avec ses propres données, laisse la victime exécuter son calcul, puis mesure (« probe ») le temps d'accès à ses propres données pour déterminer si la victime a évincé certaines lignes de cache — ce qui révèle indirectement quelles zones mémoire (et donc potentiellement quels index de table dépendant de la clé) la victime a consultées.
- **Flush+Reload** : applicable lorsque l'attaquant partage de la mémoire physique avec la victime (bibliothèque partagée, environnement virtualisé partagé), cette technique consiste à vider une ligne de cache précise, laisser la victime s'exécuter, puis mesurer le temps d'accès à cette même ligne pour savoir si la victime l'a rechargée — une technique plus précise que Prime+Probe mais qui exige des conditions de partage mémoire spécifiques.

## 6. Attaques sur les générateurs de nombres aléatoires

Un générateur de nombres pseudo-aléatoires (PRNG) de mauvaise qualité ou mal initialisé (faible entropie au démarrage, graine prévisible) compromet directement toute construction cryptographique qui en dépend (génération de clés, de nonces, de vecteurs d'initialisation). Cas réel documenté : une faille dans le générateur aléatoire d'une distribution Linux (Debian, 2006-2008) a réduit l'espace des clés SSH générées à quelques dizaines de milliers de valeurs possibles, les rendant énumérables.

### Ce que cet exemple enseigne sur la conception d'un générateur cryptographique

Cet incident, provoqué par une modification apparemment mineure du code source affectant l'accumulation d'entropie, illustre plusieurs principes durables :

- toute source d'aléa utilisée en cryptographie doit être clairement identifiée, testée et si possible standardisée (`/dev/urandom` ou un CSPRNG dédié fourni par une bibliothèque auditée), plutôt que réimplémentée ;
- une modification apparemment inoffensive d'un composant cryptographique (ici, du point de vue du code source, un changement limité) peut avoir des conséquences catastrophiques et rester non détectée pendant une longue période, ce qui plaide pour des revues de code spécifiquement dédiées aux composants de sécurité, et pour des tests statistiques réguliers de la qualité de l'aléa produit ;
- les conséquences d'un aléa faible sont souvent silencieuses : le système continue de fonctionner normalement, ce qui rend ce type de faille particulièrement difficile à détecter sans audit spécifique.

## 7. Attaques par injection de faute (fault injection)

Contrairement aux attaques passives précédentes, l'injection de faute est une attaque **active** : l'attaquant perturbe volontairement l'environnement d'exécution (variation de tension d'alimentation, glitch d'horloge, exposition à un rayonnement ciblé) pour provoquer une erreur de calcul à un instant précis de l'exécution d'un algorithme cryptographique. L'analyse de la différence entre un résultat correct et un résultat fauté (**analyse différentielle de faute**, *Differential Fault Analysis*, DFA) peut alors, dans certains schémas, permettre de retrouver la clé secrète bien plus rapidement qu'une attaque purement mathématique — un exemple classique et bien documenté dans la littérature académique concerne les implémentations de RSA utilisant le théorème des restes chinois pour accélérer le calcul, où une seule faute correctement placée peut suffire à révéler un facteur du module.

### Contre-mesures contre l'injection de faute

- **Vérification systématique du résultat** avant de le renvoyer (par exemple, pour une signature, vérifier immédiatement sa validité avec la clé publique correspondante avant de la transmettre).
- **Détecteurs matériels** de conditions anormales (tension, horloge, température) déclenchant une réinitialisation ou un blocage du dispositif.
- **Redondance de calcul** (répéter l'opération et comparer les résultats) pour détecter une incohérence provoquée par une perturbation ponctuelle.

## 8. Démarche défensive face aux canaux auxiliaires

- Utiliser des implémentations cryptographiques éprouvées et auditées (bibliothèques reconnues) plutôt que réimplémenter soi-même des primitives sensibles.
- Privilégier les implémentations en temps constant pour toute opération manipulant un secret, en gardant à l'esprit que le compilateur et le matériel peuvent réintroduire des variations que le code source seul ne laisse pas voir.
- S'assurer d'une source d'entropie fiable pour toute génération aléatoire cryptographique (`/dev/urandom`, CSPRNG dédié), et ne jamais réimplémenter un générateur pseudo-aléatoire cryptographique « maison ».
- Pour les dispositifs physiques sensibles, envisager des contre-mesures matérielles (masquage, bruit ajouté, détecteurs de perturbation) en complément des contre-mesures logicielles.
- Vérifier systématiquement le résultat d'une opération cryptographique sensible avant de le divulguer, comme filet de sécurité contre une éventuelle injection de faute.

## À retenir

- Les attaques par canaux auxiliaires exploitent l'implémentation, pas l'algorithme : elles rappellent qu'une preuve mathématique de sécurité ne suffit pas, et se répartissent entre attaques passives (mesure d'une fuite) et attaques actives (injection de faute).
- Le padding oracle est une attaque pratique majeure contre les usages naïfs de CBC (coût de l'ordre de 256 requêtes par octet), à l'origine de plusieurs failles réelles dans TLS/SSL.
- SPA, DPA et CPA exploitent la consommation électrique, avec des contre-mesures dédiées (masquage, bruit, équilibrage) ; Prime+Probe et Flush+Reload exploitent le comportement du cache processeur.
- Un générateur aléatoire de mauvaise qualité peut annuler la sécurité de tout système cryptographique qui en dépend, quelle que soit la robustesse théorique de l'algorithme utilisé — et ce type de défaillance reste souvent silencieux et difficile à détecter sans audit dédié.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
