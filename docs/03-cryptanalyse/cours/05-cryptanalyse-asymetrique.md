# Chapitre 5 — Cryptanalyse des systèmes asymétriques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/05-cryptanalyse-asymetrique.txt){ target=_blank }

## 1. RSA : rappel du principe et hypothèse de sécurité

La sécurité de RSA repose sur la difficulté supposée de **factoriser un grand nombre entier** produit de deux grands nombres premiers (*n = p × q*). Aucun algorithme classique connu ne factorise efficacement (en temps polynomial) un nombre suffisamment grand — c'est une hypothèse calculatoire, pas une preuve mathématique absolue.

### Rappel des relations mathématiques de RSA

Pour construire une paire de clés, on choisit deux grands nombres premiers distincts *p* et *q*, on calcule *n = p × q* (le module, rendu public) et l'indicatrice d'Euler *φ(n) = (p−1)(q−1)*. On choisit un exposant public *e* premier avec *φ(n)* (souvent 65537 en pratique, un choix qui offre un bon compromis entre rapidité de chiffrement et sécurité), puis on calcule l'exposant privé *d*, l'inverse modulaire de *e* modulo *φ(n)* : *e × d ≡ 1 (mod φ(n))*. Le chiffrement d'un message *m* s'écrit *c = m^e mod n*, et le déchiffrement *m = c^d mod n*.

La sécurité repose entièrement sur le fait que, connaissant uniquement *n* et *e* (les valeurs publiques), un attaquant ne peut pas calculer *d* efficacement — ce qui, à ce jour, exige de connaître *φ(n)*, ce qui exige à son tour de connaître *p* et *q*, c'est-à-dire de **factoriser n**. C'est cette chaîne de réductions qui relie directement la sécurité pratique de RSA au problème mathématique de la factorisation.

### Exemple pédagogique à très petits nombres (à but strictement illustratif — jamais utilisable en pratique)

L'exemple suivant, avec des nombres volontairement minuscules pour rester calculable à la main, illustre le mécanisme (il ne représente évidemment aucune sécurité réelle) :

```python
# Exemple RSA pédagogique à très petits nombres — usage strictement
# didactique, à des lieues de toute sécurité réelle (des clés RSA
# réelles utilisent des nombres premiers de centaines de chiffres).

p, q = 61, 53
n = p * q                      # n = 3233 (module public)
phi = (p - 1) * (q - 1)        # phi(n) = 3120
e = 17                          # exposant public choisi, premier avec phi(n)

def inverse_modulaire(e, phi):
    # Algorithme d'Euclide étendu (version simplifiée à but pédagogique)
    for d in range(2, phi):
        if (e * d) % phi == 1:
            return d
    return None

d = inverse_modulaire(e, phi)   # d = 2753 (exposant privé)

m = 65                          # message clair jouet (un petit entier)
c = pow(m, e, n)                # chiffrement : c = m^e mod n
m_retrouve = pow(c, d, n)       # déchiffrement : m = c^d mod n
assert m_retrouve == m
```

Un attaquant ne connaissant que *n = 3233* et *e = 17* devrait, pour retrouver *d*, factoriser *n* en *p = 61* et *q = 53* — trivial ici par essai de division, mais totalement impraticable dès que *p* et *q* comptent plusieurs centaines de chiffres, ce qui est le cas des clés RSA réelles (2048 bits ou plus, soit des nombres premiers d'environ 300 chiffres décimaux chacun).

## 2. Attaques par factorisation

- **Force brute / essai de division** : totalement impraticable pour des clés de taille réaliste (2048 bits ou plus).
- **Méthode de Fermat** : exploite le fait que si *p* et *q* sont proches l'un de l'autre, *n* peut s'écrire comme une différence de deux carrés (*n = a² − b²*), ce qui permet de retrouver rapidement les facteurs en cherchant *a* tel que *a² − n* soit un carré parfait. Cette méthode illustre pourquoi les générateurs de clés RSA doivent s'assurer que *p* et *q* sont suffisamment éloignés l'un de l'autre.
- **Algorithmes sous-exponentiels** : le *General Number Field Sieve* (GNFS) est l'algorithme classique le plus efficace connu pour factoriser de grands nombres ; sa complexité, bien que sous-exponentielle (plus rapide que la force brute, mais bien plus lente qu'un algorithme polynomial), reste hors de portée pour des clés RSA de 2048 bits avec la technologie actuelle. C'est pourquoi la taille de clé recommandée a augmenté avec le temps (512 bits cassé dès 1999, 768 bits cassé en 2009, 1024 bits considéré fragilisé, 2048 bits recommandé aujourd'hui).
- **Algorithme rho de Pollard** : une méthode probabiliste, plus efficace que l'essai de division mais toujours de complexité exponentielle dans le nombre de bits, souvent utilisée pour trouver rapidement de petits facteurs avant de recourir à une méthode plus lourde comme le GNFS.
- **Menace quantique** : l'algorithme de Shor, exécuté sur un ordinateur quantique suffisamment puissant, factoriserait un grand nombre en temps polynomial — casserait RSA (et le logarithme discret, donc Diffie-Hellman/ECC classiques). Cette menace motive la standardisation en cours d'algorithmes post-quantiques (ex. NIST PQC : ML-KEM/Kyber, ML-DSA/Dilithium).

## 3. Attaques exploitant une mauvaise implémentation de RSA

Ces attaques ne cassent pas le problème mathématique sous-jacent mais exploitent des erreurs pratiques, bien plus fréquentes en réalité :

| Attaque | Condition d'application |
|---|---|
| **Exposant public *e* trop petit sans padding** (*e* = 3) | message court, chiffré sans padding correct (OAEP), permet une racine cubique directe |
| **Modules partagés / clés faibles** | deux clés RSA différentes partagent accidentellement un facteur premier → PGCD des deux modules révèle instantanément la factorisation |
| **Attaque de Wiener** | exposant privé *d* choisi trop petit par rapport au module → récupération de *d* par développement en fractions continues |
| **Attaque de Coppersmith** | conditions particulières sur des bits connus de *p*, *q* ou du message (par exemple une fraction significative des bits de *p* déjà connue) |
| **Attaque de diffusion (Håstad)** | un même message chiffré avec le même petit exposant public (*e* = 3) vers plusieurs destinataires disposant de modules différents → combinaison des chiffrés (théorème des restes chinois) pour retrouver le message sans factoriser aucun module |
| **Réutilisation de nonce / mauvaise génération aléatoire** | générateur pseudo-aléatoire faible lors de la génération des nombres premiers → cas réel documenté sur des équipements embarqués bon marché |

### Approfondissement : intuition de l'attaque du module commun / PGCD

Si deux modules RSA distincts *n₁ = p × q₁* et *n₂ = p × q₂* partagent accidentellement le même facteur premier *p* (par exemple à cause d'un générateur de nombres aléatoires de mauvaise qualité produisant des premiers avec trop peu d'entropie), alors le calcul du plus grand commun diviseur *PGCD(n₁, n₂)* révèle instantanément *p*, sans aucune tentative de factorisation classique — un simple algorithme d'Euclide, de complexité négligeable, suffit. C'est une illustration frappante de la section 1 du chapitre 1 : un problème mathématiquement difficile (factoriser *n*) peut devenir trivial dès qu'une hypothèse implicite de génération (indépendance et aléa suffisant des nombres premiers) est violée en pratique.

### Approfondissement : intuition de l'attaque de Wiener

Si l'exposant privé *d* est choisi trop petit (approximativement *d* < n^0,25, une borne précisée par les travaux ultérieurs de Boneh et Durfee qui l'ont étendue), la fraction *e/n* admet un développement en fractions continues dont l'un des convergents permet de retrouver *d* directement, en temps polynomial. Cette attaque illustre pourquoi un exposant privé « pratique » (choisi petit pour accélérer le déchiffrement) est une optimisation dangereuse : la norme actuelle recommande un exposant privé de taille comparable au module lui-même, quitte à accepter un coût de déchiffrement plus élevé (souvent compensé en pratique par le théorème des restes chinois, qui accélère le calcul sans réduire *d*).

## 4. Diffie-Hellman et logarithme discret

La sécurité de Diffie-Hellman (et d'ElGamal) repose sur la difficulté du **problème du logarithme discret** : étant donné *g*, *p* et *gˣ mod p*, retrouver *x* est supposé difficile. Les meilleures attaques classiques connues (*index calculus*) ont une complexité comparable à la factorisation RSA pour une taille de groupe équivalente, d'où des recommandations de taille de clé similaires.

L'algorithme d'*index calculus* exploite une structure particulière des groupes multiplicatifs modulo un nombre premier (contrairement aux courbes elliptiques, voir section 5) : il précalcule les logarithmes discrets d'un ensemble de « petits » nombres premiers (une base de facteurs), puis exprime le logarithme cible comme combinaison de ces valeurs précalculées. C'est cette structure exploitable qui explique pourquoi le logarithme discret sur les entiers modulo un nombre premier nécessite des tailles de clé comparables à celles de RSA (2048 bits ou plus), contrairement au logarithme discret sur courbe elliptique, dépourvu de structure équivalente exploitable par *index calculus*.

**Attaque pratique notable** : l'attaque *Logjam* (2015) a montré que de nombreux serveurs réutilisaient les mêmes petits groupes premiers standardisés pour Diffie-Hellman, permettant un précalcul mutualisé (la partie la plus coûteuse d'*index calculus* ne dépend que du groupe, pas de la clé spécifique) rendant l'attaque de logarithme discret praticable contre des groupes de 512 bits — illustration qu'un paramètre partagé et sous-dimensionné affaiblit tous les systèmes qui le réutilisent, un précalcul unique amortissant son coût sur un très grand nombre de cibles.

## 5. Courbes elliptiques (ECC)

Le problème du logarithme discret sur courbe elliptique (ECDLP) est considéré plus difficile, à taille de clé égale, que son équivalent sur les entiers : une clé ECC de 256 bits offre un niveau de sécurité comparable à une clé RSA de 3072 bits, pour un coût de calcul et une taille de clé bien moindres — ce qui explique l'adoption croissante d'ECC (ECDSA, X25519) dans les protocoles modernes (TLS 1.3, SSH).

### Pourquoi ECDLP résiste-t-il mieux, à taille égale ?

Le groupe des points d'une courbe elliptique ne possède pas de structure exploitable équivalente à celle utilisée par *index calculus* sur les entiers : les meilleures attaques génériques connues contre ECDLP, comme l'**algorithme rho de Pollard adapté aux courbes elliptiques**, ont une complexité en O(√N), où *N* est l'ordre du groupe — c'est-à-dire une complexité purement exponentielle, comparable à une recherche exhaustive optimisée par le paradoxe des anniversaires (voir chapitre 4), sans raccourci structurel connu. C'est cette absence de structure exploitable qui permet à ECC d'offrir un niveau de sécurité donné avec des clés bien plus courtes que RSA ou Diffie-Hellman classique.

### Point de vigilance : le choix de la courbe

Toutes les courbes elliptiques ne se valent pas : certains paramétrages standardisés ont fait l'objet de suspicions (génération opaque de paramètres, sans justification vérifiable, dans le cas de certaines courbes historiques normalisées par des organismes gouvernementaux), ce qui a motivé l'adoption croissante de courbes dont les paramètres sont générés de façon vérifiable et reproductible (comme Curve25519), un critère de confiance qui s'ajoute à la seule robustesse mathématique du problème ECDLP.

## 6. Démarche générale de cryptanalyse d'un système asymétrique

1. Vérifier les paramètres utilisés (taille de clé, courbe/groupe standardisé, largement audité, et non « maison »).
2. Rechercher des erreurs de génération de clé (facteurs partagés, aléa faible, exposant privé anormalement petit).
3. Vérifier l'usage correct du padding (OAEP pour le chiffrement RSA, PSS pour la signature) — un module et un exposant robustes ne protègent en rien contre une utilisation sans padding adapté.
4. Ne considérer la factorisation/logarithme discret « brut » que comme dernier recours théorique : en pratique, l'immense majorité des compromissions viennent des points 1 à 3.

## À retenir

- La sécurité de RSA et Diffie-Hellman repose sur des hypothèses calculatoires (factorisation, logarithme discret), pas sur une preuve d'impossibilité absolue ; les meilleurs algorithmes classiques connus (GNFS, index calculus) restent sous-exponentiels mais impraticables aux tailles de clé recommandées actuelles.
- Les attaques réellement exploitées en pratique ciblent presque toujours une mauvaise implémentation (aléa faible menant à des facteurs premiers partagés, exposant privé trop petit exploitable par l'attaque de Wiener, absence de padding OAEP/PSS, diffusion d'un même message à petit exposant) plutôt que l'algorithme mathématique lui-même.
- ECC atteint un niveau de sécurité équivalent à RSA avec des clés bien plus courtes, car le logarithme discret sur courbe elliptique ne possède pas d'équivalent à l'attaque par index calculus qui affaiblit RSA et Diffie-Hellman classique.
- La menace de l'informatique quantique (algorithme de Shor, complexité polynomiale) casserait à la fois RSA, Diffie-Hellman et ECC classiques, ce qui motive la transition en cours vers la cryptographie post-quantique (NIST PQC).

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
