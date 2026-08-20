# Chapitre 1 — Introduction et principes fondamentaux

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/01-introduction-principes-fondamentaux.txt){ target=_blank }

## 1. Les objectifs de la cryptographie

La cryptographie moderne vise à garantir, en présence d'un adversaire potentiel, quatre propriétés fondamentales :

- **Confidentialité** : seules les parties autorisées peuvent accéder au contenu d'une information.
- **Intégrité** : toute modification non autorisée d'une information est détectable.
- **Authenticité** : l'origine d'une information (identité de l'émetteur) est vérifiable.
- **Non-répudiation** : l'émetteur d'un message ne peut pas nier ultérieurement en être l'auteur (propriété apportée notamment par la signature numérique).

Ces quatre propriétés ne sont pas systématiquement toutes requises en même temps. Un simple chiffrement symétrique d'un fichier personnel n'a par exemple besoin que de confidentialité. À l'inverse, un contrat signé électroniquement a surtout besoin d'intégrité, d'authenticité et de non-répudiation, et pas forcément de confidentialité (le contrat peut être public). Une partie du travail d'un architecte de sécurité consiste précisément à identifier, pour chaque flux d'information, quelles propriétés sont réellement nécessaires, car chacune a un coût (calcul, complexité de mise en œuvre, gestion de clés).

### 1.1 Illustration par le contexte

| Scénario | Confidentialité | Intégrité | Authenticité | Non-répudiation |
|---|---|---|---|---|
| Message chiffré entre deux amis | nécessaire | souhaitable | souhaitable | non nécessaire |
| Mise à jour logicielle signée | non nécessaire (le code peut être public) | nécessaire | nécessaire | souhaitable |
| Contrat électronique | dépend du contexte | nécessaire | nécessaire | nécessaire |
| Trafic bancaire en ligne (TLS) | nécessaire | nécessaire | nécessaire | rarement exigée pour le client |

Ce tableau illustre pourquoi aucun mécanisme unique ne couvre tous les besoins : c'est précisément l'articulation des primitives présentées dans les chapitres suivants (chiffrement symétrique, hachage, MAC, chiffrement asymétrique, signature) qui permet de composer la combinaison de propriétés adaptée à chaque situation.

## 2. Le principe de Kerckhoffs

Formulé par Auguste Kerckhoffs en 1883 : **la sécurité d'un système cryptographique ne doit reposer que sur le secret de la clé, jamais sur le secret de l'algorithme**. Un algorithme dont la sécurité dépend de rester confidentiel (« security through obscurity ») est fragile : il suffit qu'il soit divulgué ou rétro-ingénié une seule fois pour effondrer la sécurité de tous les systèmes qui l'utilisent. À l'inverse, un algorithme public, largement étudié par la communauté scientifique (comme AES, RSA), gagne en confiance précisément parce qu'il résiste à un examen ouvert et prolongé — lien direct avec le module *Cryptanalyse*.

### 2.1 Pourquoi la publication renforce la confiance

Un algorithme cryptographique standardisé (AES, SHA-2, RSA, ECC) est publié avec sa spécification complète, ses vecteurs de test, et souvent son code de référence. Cela permet :

- à des équipes académiques et industrielles indépendantes de chercher des faiblesses pendant des années avant tout déploiement massif ;
- à des concours publics (comme celui qui a désigné Rijndael comme AES en 2001) de sélectionner l'algorithme le plus robuste parmi plusieurs candidats soumis à un examen collectif ;
- à l'écosystème (bibliothèques, matériel, protocoles) de converger vers un nombre restreint de primitives bien comprises plutôt qu'une multitude d'implémentations maison, chacune potentiellement fragile.

Un exemple historique bien documenté illustre l'inverse du principe de Kerckhoffs : l'algorithme A5/1, utilisé pour chiffrer les communications GSM, est resté secret pendant plusieurs années avant d'être rétro-ingénié ; une fois sa structure connue, des faiblesses structurelles ont pu être analysées publiquement. De même, l'algorithme de chiffrement CSS des DVD, conçu de façon fermée, a été cassé peu après sa divulgation. Ces cas illustrent concrètement pourquoi la communauté cryptographique privilégie aujourd'hui systématiquement des algorithmes ouverts et standardisés plutôt que des conceptions propriétaires non publiées.

### 2.2 Ce que Kerckhoffs n'interdit pas

Le principe ne dit pas qu'il faut publier sa clé, son mot de passe ou les détails d'une configuration précise (topologie réseau, version logicielle exacte). Il porte spécifiquement sur l'algorithme. Garder une configuration discrète relève de la défense en profondeur (module *Fondamentaux*), mais ne doit jamais être le fondement principal de la sécurité d'un mécanisme cryptographique.

## 3. Confusion et diffusion (Claude Shannon, 1949)

Deux propriétés structurantes de tout chiffrement robuste, formalisées par Claude Shannon dans son article fondateur *Communication Theory of Secrecy Systems* :

- **Confusion** : rendre la relation entre la clé et le texte chiffré la plus complexe possible, pour qu'une analyse statistique du chiffré ne révèle rien sur la clé.
- **Diffusion** : répartir l'influence de chaque bit du texte clair (et de la clé) sur l'ensemble du texte chiffré, pour qu'une modification d'un seul bit du clair change en moyenne la moitié des bits du chiffré (effet avalanche).

Ces deux notions justifient directement la structure des chiffrements par blocs modernes (tours de substitution et de permutation dans AES, chapitre 2). Sans confusion, un attaquant pourrait exploiter une corrélation statistique simple entre clé et chiffré (comme dans un chiffrement par substitution monoalphabétique, cassable par analyse de fréquence). Sans diffusion, une petite modification du message clair ne modifierait qu'une petite portion prévisible du chiffré, facilitant des attaques différentielles.

### 3.1 Effet avalanche : illustration empirique

L'effet avalanche est plus facile à observer sur une fonction de hachage (chapitre 3) qu'à la main sur un chiffrement par bloc complet, mais le principe est identique. À titre d'illustration immédiate :

```bash
# Deux entrées ne différant que d'un caractère
echo -n "cours de securite" | sha256sum
echo -n "cours de securite." | sha256sum
```

Les deux empreintes produites n'ont statistiquement aucun bit en commun de façon prévisible, bien que les entrées ne diffèrent que d'un seul caractère : c'est la manifestation concrète de la diffusion. Un attaquant observant uniquement le chiffré (ou l'empreinte) ne peut donc rien inférer sur la nature de la modification apportée au clair.

## 4. Cryptographie symétrique vs asymétrique : vue d'ensemble

| | Symétrique | Asymétrique |
|---|---|---|
| Clés | une seule clé, partagée entre émetteur et récepteur | une paire de clés (publique/privée) |
| Vitesse | rapide | beaucoup plus lente |
| Problème principal | distribution sécurisée de la clé partagée | vérification de l'authenticité de la clé publique |
| Exemples | AES, ChaCha20 | RSA, Diffie-Hellman, ECC |

En pratique, les systèmes réels combinent les deux : la cryptographie asymétrique établit ou échange une clé de session, ensuite utilisée pour un chiffrement symétrique rapide du volume de données (principe du **chiffrement hybride**, à l'œuvre notamment dans TLS, détaillé au chapitre 6).

### 4.1 Le problème combinatoire de la distribution de clés symétriques

La cryptographie asymétrique n'a pas été inventée par confort théorique : elle répond à un problème opérationnel concret. Si `n` personnes veulent pouvoir communiquer deux à deux de façon confidentielle en utilisant uniquement de la cryptographie symétrique, il faut une clé distincte pour chaque paire, soit :

```
nombre de clés = n(n-1)/2
```

Pour illustrer avec des ordres de grandeur simples (exemple pédagogique) :

| Nombre de participants (n) | Nombre de clés symétriques nécessaires |
|---|---|
| 10 | 45 |
| 100 | 4 950 |
| 1 000 | 499 500 |

Ce nombre croît de façon quadratique : au-delà de quelques dizaines de participants, la distribution et le renouvellement manuel de toutes ces clés deviennent ingérables. La cryptographie asymétrique résout ce problème : chaque participant ne détient qu'**une seule paire de clés**, et diffuse librement sa clé publique — le nombre d'objets cryptographiques à gérer croît alors linéairement (`n`), et non plus quadratiquement.

## 5. Notion de sécurité prouvable et de sécurité calculatoire

La sécurité d'un système cryptographique moderne repose généralement sur une hypothèse calculatoire : un problème mathématique réputé difficile à résoudre en un temps raisonnable avec les moyens de calcul actuels et prévisibles (factorisation pour RSA, logarithme discret pour Diffie-Hellman/ECC). Il ne s'agit pas d'une preuve d'impossibilité absolue (à l'exception du **masque jetable** — *one-time pad* — seul système prouvé théoriquement incassable, mais impraticable car il exige une clé aussi longue que le message, utilisée une seule fois).

### 5.1 Niveaux de sécurité en bits

Pour comparer la robustesse de mécanismes de nature mathématique différente (chiffrement symétrique, factorisation, logarithme discret), on utilise la notion de **niveau de sécurité en bits** : un niveau de `k` bits signifie qu'un attaquant doit effectuer de l'ordre de `2^k` opérations pour casser le mécanisme par la meilleure attaque connue. Le tableau suivant, cohérent avec les recommandations usuelles (NIST, ANSSI) évoquées au fil du module, donne une correspondance indicative entre familles d'algorithmes pour un même niveau de sécurité :

| Niveau de sécurité | AES (symétrique) | RSA (modulus) | ECC (taille de courbe) |
|---|---|---|---|
| 128 bits | AES-128 | ≈ 3072 bits | ≈ 256 bits |
| 192 bits | AES-192 | ≈ 7680 bits | ≈ 384 bits |
| 256 bits | AES-256 | ≈ 15360 bits | ≈ 512 bits |

Cette correspondance explique pourquoi les clés RSA sont numériquement bien plus grandes que les clés ECC ou AES à sécurité équivalente (rappel détaillé au chapitre 4) : la difficulté de factorisation d'un grand nombre croît beaucoup moins vite avec la taille de la clé que la difficulté d'une recherche exhaustive sur une clé symétrique.

### 5.2 Pourquoi 2^128 est considéré comme hors de portée

Un ordre de grandeur pédagogique couramment cité pour illustrer l'infaisabilité d'une recherche exhaustive sur une clé de 128 bits : même en supposant un matériel capable de tester un très grand nombre de clés par seconde, parcourir l'espace des `2^128` clés possibles prendrait un temps dépassant très largement toute échelle humaine ou même cosmologique envisageable avec la technologie actuelle. C'est cette marge de sécurité considérable, et non une preuve mathématique d'impossibilité, qui fonde la confiance accordée à AES-128 et aux mécanismes de niveau de sécurité comparable.

## 6. Modèles d'attaquant

L'analyse de sécurité d'un mécanisme cryptographique dépend explicitement de ce que l'on suppose que l'attaquant peut observer ou obtenir. Les modèles classiques, du plus faible au plus fort, sont :

- **Ciphertext-only** : l'attaquant n'a accès qu'à des textes chiffrés.
- **Known-plaintext** : l'attaquant connaît des couples (clair, chiffré) correspondants.
- **Chosen-plaintext (CPA)** : l'attaquant peut faire chiffrer des textes clairs de son choix et observer le résultat.
- **Chosen-ciphertext (CCA)** : l'attaquant peut en plus faire déchiffrer des textes chiffrés de son choix (sauf celui qu'il cherche à casser).

Un mécanisme moderne (AES-GCM, RSA-OAEP) est conçu pour résister au modèle le plus fort raisonnablement pertinent pour son usage, généralement CCA. Ce vocabulaire sera repris et approfondi au module *Cryptanalyse*.

## 7. Vocabulaire de base

| Terme | Définition |
|---|---|
| Texte clair (plaintext) | message original, non protégé |
| Texte chiffré (ciphertext) | résultat du chiffrement |
| Clé | paramètre secret contrôlant le chiffrement/déchiffrement |
| Algorithme (chiffre, cipher) | procédure de transformation, publique (Kerckhoffs) |
| Cryptosystème | ensemble algorithme + gestion des clés + protocole d'usage |
| Nonce | valeur utilisée une seule fois avec une même clé, souvent non secrète |
| Vecteur d'initialisation (IV) | valeur d'amorçage d'un mode opératoire, souvent aléatoire |
| Sel (salt) | valeur aléatoire additionnelle, typiquement utilisée en hachage de mots de passe |
| Empreinte (digest) | sortie de taille fixe d'une fonction de hachage |
| Oracle | dans une analyse de sécurité, entité hypothétique répondant à des requêtes de l'attaquant (ex. « oracle de déchiffrement ») |

## 8. Bonnes pratiques actuelles

- **Ne jamais concevoir son propre algorithme cryptographique** pour un usage réel : même des cryptographes expérimentés se trompent ; utiliser des standards éprouvés (AES, RSA/ECC, SHA-2/SHA-3) et des bibliothèques largement auditées (OpenSSL, libsodium, BoringSSL).
- **Générer tout matériel aléatoire (clés, nonces, sels) via un générateur cryptographiquement sûr (CSPRNG)**, jamais via une fonction pseudo-aléatoire générique (comme `rand()` en C) inadaptée à un usage cryptographique.
- **Suivre les paramètres recommandés par les référentiels actuels** (ANSSI, NIST) plutôt que des choix historiques ou par défaut potentiellement obsolètes.
- **Prévoir dès la conception l'agilité cryptographique** : la capacité à remplacer un algorithme ou une taille de clé sans réécrire toute l'architecture, en anticipation de futures dépréciations (illustré au chapitre 6 avec la transition post-quantique).

```bash
# Exemple : génération d'un matériel aléatoire de qualité cryptographique avec OpenSSL
openssl rand -hex 32          # 32 octets (256 bits) aléatoires, encodés en hexadécimal
openssl version -a            # vérifier la version d'OpenSSL et sa configuration
```

## 9. Pièges d'implémentation courants

- **« Rouler sa propre crypto »** : réinventer un algorithme ou un protocole plutôt que d'utiliser un composant standard et audité — source très fréquente de failles en pratique.
- **Comparaisons non constantes en temps** : comparer deux secrets (par exemple un MAC calculé et un MAC reçu) avec une comparaison naïve peut fuiter de l'information via le temps d'exécution (attaque temporelle, *timing attack*) ; il faut utiliser une fonction de comparaison en temps constant fournie par la bibliothèque cryptographique.
- **Confondre encodage et chiffrement** : Base64 n'apporte aucune confidentialité, ce n'est qu'un encodage réversible sans clé.
- **Clés codées en dur dans le code source** ou committées dans un dépôt de version, exposant durablement le secret même après correction.
- **Absence de plan de rotation des clés** : une clé compromise sans mécanisme de renouvellement prévu prolonge l'impact d'un incident bien au-delà du nécessaire.

## À retenir

- Confidentialité, intégrité, authenticité, non-répudiation : quatre objectifs distincts, pas toujours tous nécessaires simultanément selon le contexte.
- Le principe de Kerckhoffs interdit de fonder la sécurité sur le secret de l'algorithme ; seule la clé doit rester secrète.
- Confusion et diffusion (Shannon) structurent la conception des chiffrements par blocs modernes et expliquent l'effet avalanche.
- La cryptographie asymétrique répond notamment au problème combinatoire de la distribution de clés symétriques à grande échelle.
- Le niveau de sécurité en bits permet de comparer des mécanismes de nature mathématique différente (symétrique, factorisation, logarithme discret).
- La cryptographie moderne combine en pratique symétrique (rapide, volume) et asymétrique (échange de clé, authentification) dans des schémas hybrides.
- Ne jamais concevoir sa propre cryptographie ; utiliser des standards publics, éprouvés, et des implémentations auditées.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
