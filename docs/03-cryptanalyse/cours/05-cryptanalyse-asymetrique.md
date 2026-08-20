# Chapitre 5 — Cryptanalyse des systèmes asymétriques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=03-cryptanalyse/slides/05-cryptanalyse-asymetrique.txt){ target=_blank }

## 1. RSA : rappel du principe et hypothèse de sécurité

La sécurité de RSA repose sur la difficulté supposée de **factoriser un grand nombre entier** produit de deux grands nombres premiers (*n = p × q*). Aucun algorithme classique connu ne factorise efficacement (en temps polynomial) un nombre suffisamment grand — c'est une hypothèse calculatoire, pas une preuve mathématique absolue.

## 2. Attaques par factorisation

- **Force brute / essai de division** : totalement impraticable pour des clés de taille réaliste (2048 bits ou plus).
- **Algorithmes sous-exponentiels** : le *General Number Field Sieve* (GNFS) est l'algorithme classique le plus efficace connu pour factoriser de grands nombres ; sa complexité, bien que sous-exponentielle, reste hors de portée pour des clés RSA de 2048 bits avec la technologie actuelle. C'est pourquoi la taille de clé recommandée a augmenté avec le temps (512 bits cassé dès 1999, 768 bits cassé en 2009, 1024 bits considéré fragilisé, 2048 bits recommandé aujourd'hui).
- **Menace quantique** : l'algorithme de Shor, exécuté sur un ordinateur quantique suffisamment puissant, factoriserait un grand nombre en temps polynomial — casserait RSA (et le logarithme discret, donc Diffie-Hellman/ECC classiques). Cette menace motive la standardisation en cours d'algorithmes post-quantiques (ex. NIST PQC : ML-KEM/Kyber, ML-DSA/Dilithium).

## 3. Attaques exploitant une mauvaise implémentation de RSA

Ces attaques ne cassent pas le problème mathématique sous-jacent mais exploitent des erreurs pratiques, bien plus fréquentes en réalité :

| Attaque | Condition d'application |
|---|---|
| **Exposant public *e* trop petit sans padding** (*e* = 3) | message court, chiffré sans padding correct (OAEP), permet une racine cubique directe |
| **Modules partagés / clés faibles** | deux clés RSA différentes partagent accidentellement un facteur premier → PGCD des deux modules révèle instantanément la factorisation |
| **Attaque de Wiener** | exposant privé *d* choisi trop petit par rapport au module → récupération de *d* par développement en fractions continues |
| **Attaque de Coppersmith** | conditions particulières sur des bits connus de *p*, *q* ou du message |
| **Réutilisation de nonce / mauvaise génération aléatoire** | générateur pseudo-aléatoire faible lors de la génération des nombres premiers → cas réel documenté sur des équipements embarqués bon marché |

## 4. Diffie-Hellman et logarithme discret

La sécurité de Diffie-Hellman (et d'ElGamal) repose sur la difficulté du **problème du logarithme discret** : étant donné *g*, *p* et *gˣ mod p*, retrouver *x* est supposé difficile. Les meilleures attaques classiques connues (*index calculus*) ont une complexité comparable à la factorisation RSA pour une taille de groupe équivalente, d'où des recommandations de taille de clé similaires.

**Attaque pratique notable** : l'attaque *Logjam* (2015) a montré que de nombreux serveurs réutilisaient les mêmes petits groupes premiers standardisés pour Diffie-Hellman, permettant un précalcul mutualisé rendant l'attaque de logarithme discret praticable contre des groupes de 512 bits — illustration qu'un paramètre partagé et sous-dimensionné affaiblit tous les systèmes qui le réutilisent.

## 5. Courbes elliptiques (ECC)

Le problème du logarithme discret sur courbe elliptique (ECDLP) est considéré plus difficile, à taille de clé égale, que son équivalent sur les entiers : une clé ECC de 256 bits offre un niveau de sécurité comparable à une clé RSA de 3072 bits, pour un coût de calcul et une taille de clé bien moindres — ce qui explique l'adoption croissante d'ECC (ECDSA, X25519) dans les protocoles modernes (TLS 1.3, SSH).

## 6. Démarche générale de cryptanalyse d'un système asymétrique

1. Vérifier les paramètres utilisés (taille de clé, courbe/groupe standardisé et non « maison »).
2. Rechercher des erreurs de génération de clé (facteurs partagés, aléa faible).
3. Vérifier l'usage correct du padding (OAEP pour le chiffrement RSA, PSS pour la signature).
4. Ne considérer la factorisation/logarithme discret « brut » que comme dernier recours théorique : en pratique, l'immense majorité des compromissions viennent des points 1 à 3.

## À retenir

- La sécurité de RSA et Diffie-Hellman repose sur des hypothèses calculatoires (factorisation, logarithme discret), pas sur une preuve d'impossibilité absolue.
- Les attaques réellement exploitées en pratique ciblent presque toujours une mauvaise implémentation (aléa faible, paramètres partagés, padding absent) plutôt que l'algorithme mathématique lui-même.
- La menace de l'informatique quantique (algorithme de Shor) motive la transition en cours vers la cryptographie post-quantique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
