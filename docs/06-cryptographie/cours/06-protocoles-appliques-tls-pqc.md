# Chapitre 6 — Protocoles appliqués : TLS et cryptographie post-quantique

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/06-protocoles-appliques-tls-pqc.txt){ target=_blank }

## 1. TLS (Transport Layer Security) : rôle et positionnement

TLS sécurise les communications entre un client et un serveur (HTTPS, mais aussi d'autres protocoles applicatifs) en fournissant confidentialité, intégrité et authentification du serveur (et optionnellement du client). Il illustre concrètement l'intégration de toutes les primitives vues dans ce module.

### 1.1 Position de TLS dans la pile réseau

TLS s'insère entre la couche transport (TCP) et la couche applicative : une application (navigateur web, client de messagerie, etc.) ouvre une connexion TCP classique, puis établit par-dessus une session TLS, avant d'échanger les données applicatives (requêtes HTTP, par exemple) qui transitent alors chiffrées et authentifiées de bout en bout de la connexion. Ce positionnement permet à TLS de protéger, de façon relativement transparente, n'importe quel protocole applicatif construit au-dessus (HTTPS pour HTTP, SMTPS pour le courrier électronique, etc.), sans que ce protocole applicatif ait lui-même à implémenter de mécanisme cryptographique.

## 2. La poignée de main TLS 1.3 (simplifiée)

1. **ClientHello** : le client propose les suites cryptographiques supportées et envoie des paramètres pour un échange de clé (ex. clé publique éphémère pour Diffie-Hellman sur courbe elliptique).
2. **ServerHello** : le serveur choisit une suite cryptographique, complète l'échange de clé et envoie son certificat.
3. **Vérification du certificat** : le client valide la chaîne de confiance jusqu'à une AC racine reconnue (chapitre 5).
4. **Dérivation des clés de session** : à partir du secret partagé issu de l'échange de clé, une fonction de dérivation de clé produit les clés symétriques utilisées pour le reste de la session.
5. **Communication chiffrée** : l'ensemble du trafic applicatif est protégé par un chiffrement symétrique authentifié (AES-GCM ou ChaCha20-Poly1305).

TLS 1.3 (2018) a simplifié la poignée de main par rapport à TLS 1.2, supprimé les suites cryptographiques jugées faibles (RC4, CBC sans protection adéquate, RSA pour l'échange de clé sans confidentialité persistante), et impose désormais un échange de clé Diffie-Hellman éphémère par défaut.

### 2.1 Ce que fait précisément l'étape de dérivation de clés

L'étape 4 illustre directement les chapitres précédents : le secret partagé brut issu de l'échange ECDHE (chapitre 4) n'est jamais utilisé tel quel comme clé de chiffrement. Il sert d'entrée à **HKDF** (chapitre 3, fondée sur HMAC), qui dérive à partir de ce secret plusieurs clés distinctes et indépendantes : une clé pour chiffrer les données du client vers le serveur, une autre pour le sens serveur vers client, ainsi que des clés séparées pour certaines phases intermédiaires de la poignée de main elle-même (« clés de handshake », distinctes des « clés d'application » utilisées pour les données finales). Cette séparation stricte des clés par usage limite l'impact d'une éventuelle faiblesse localisée à un seul de ces flux.

### 2.2 Réduction du nombre d'allers-retours (1-RTT et 0-RTT)

Une des motivations principales de la refonte de TLS 1.3 était de réduire la latence d'établissement de connexion. TLS 1.2 nécessitait généralement deux allers-retours complets avant de pouvoir échanger des données applicatives chiffrées ; TLS 1.3 y parvient en un seul aller-retour (« 1-RTT »), le client envoyant directement ses paramètres d'échange de clé dans le `ClientHello` sans attendre de savoir quelle suite le serveur choisira. TLS 1.3 introduit également un mode optionnel **0-RTT**, permettant au client, lors d'une reconnexion à un serveur déjà visité, d'envoyer des données applicatives dès le premier message, en réutilisant un secret établi lors d'une session précédente. Ce gain de latence a un coût en sécurité qu'il faut connaître : les données envoyées en 0-RTT ne bénéficient pas de la même protection contre les attaques par rejeu (*replay*) qu'un flux 1-RTT classique, ce qui limite son usage recommandé à des requêtes idempotentes (sans effet de bord dupliqué en cas de rejeu).

## 3. Confidentialité persistante (Perfect Forward Secrecy)

Propriété garantissant que la compromission ultérieure de la clé privée à long terme d'un serveur ne permet pas de déchiffrer rétroactivement des sessions passées interceptées et enregistrées. Obtenue en utilisant des clés d'échange **éphémères** (générées pour chaque session puis détruites), typiquement via Diffie-Hellman éphémère sur courbe elliptique (ECDHE). C'est une des raisons pour lesquelles TLS 1.3 impose ce mécanisme par défaut.

### 3.1 Scénario illustratif

Imaginons qu'un adversaire enregistre passivement l'intégralité du trafic TLS chiffré échangé entre un client et un serveur sur plusieurs mois (scénario purement illustratif). Si le serveur utilisait un transport de clé RSA statique (ancien schéma TLS, aujourd'hui supprimé de TLS 1.3), la compromission ultérieure de la clé privée RSA du serveur — par exemple lors d'un incident de sécurité découvert bien plus tard — permettrait à l'adversaire de déchiffrer rétroactivement l'ensemble du trafic enregistré. Avec ECDHE, chaque session utilise une paire de clés éphémères générée puis détruite immédiatement après l'établissement du secret de session : la compromission ultérieure de la clé privée long terme du serveur (utilisée uniquement pour authentifier, c'est-à-dire signer, l'échange, et non pour le chiffrer directement) ne permet pas de reconstituer les secrets de session déjà détruits. C'est cette différence structurelle qui a motivé le retrait complet du transport de clé RSA dans TLS 1.3.

## 4. Suites cryptographiques et obsolescence

Une « suite cryptographique » combine un mécanisme d'échange de clé, un algorithme de signature, un chiffrement symétrique et une fonction de hachage. Les recommandations évoluent avec le temps à mesure que des faiblesses sont découvertes (rappel du module *Cryptanalyse*) : SSL 2.0/3.0, TLS 1.0/1.1, RC4, l'export de clés faibles (« EXPORT »), sont aujourd'hui déconseillés voire interdits par les référentiels de sécurité (ANSSI, PCI-DSS).

### 4.1 Composition d'une suite en TLS 1.3

TLS 1.3 a simplifié la notation des suites cryptographiques par rapport à TLS 1.2 : l'échange de clé et l'algorithme de signature sont désormais négociés séparément de la suite de chiffrement symétrique proprement dite, qui se limite à un couple algorithme AEAD + fonction de hachage (utilisée pour HKDF), par exemple `TLS_AES_256_GCM_SHA384` ou `TLS_CHACHA20_POLY1305_SHA256`. Cette simplification réduit fortement le nombre de combinaisons possibles par rapport à TLS 1.2, où le nombre très élevé de suites disponibles (certaines faibles) rendait la configuration serveur plus sujette à erreur.

```bash
# Lister les suites cryptographiques TLS 1.3 supportées par OpenSSL
openssl ciphers -s -tls1_3

# Se connecter à un serveur en forçant TLS 1.3 et afficher la suite négociée
openssl s_client -connect exemple.org:443 -tls1_3 -servername exemple.org </dev/null 2>/dev/null | grep -A1 "Cipher"
```

### 4.2 Bonnes pratiques actuelles de configuration serveur

- **Désactiver explicitement TLS 1.0/1.1 et SSL 2.0/3.0** côté serveur, même si le client ne les propose théoriquement jamais, pour éliminer tout risque de négociation dégradée forcée par un attaquant actif.
- **N'activer que des suites AEAD** (AES-GCM, ChaCha20-Poly1305), en écartant toute suite basée sur CBC sans protection adéquate ou sur RC4.
- **Activer HSTS** (*HTTP Strict Transport Security*) côté serveur web pour empêcher qu'un client accepte une connexion HTTP non chiffrée vers le même domaine après une première visite en HTTPS, réduisant l'exposition à une attaque de rétrogradation (*downgrade*).
- **Surveiller et renouveler les certificats avant expiration**, en automatisant ce renouvellement plutôt qu'en s'appuyant sur une procédure manuelle sujette à l'oubli.

## 5. Erreurs d'implémentation TLS fréquentes (audit)

- Ne pas vérifier le nom de domaine du certificat.
- Accepter des certificats auto-signés ou expirés en production.
- Désactiver la vérification de certificat pendant le développement, oubli en production (vulnérabilité réelle et récurrente).
- Utiliser des suites cryptographiques obsolètes non désactivées côté serveur.

Ces points recoupent directement la grille d'audit applicatif vue dans le module *Audit organisation et technique*.

### 5.1 Heartbleed : un exemple documenté d'impact d'une faille d'implémentation

La vulnérabilité **Heartbleed** (CVE-2014-0160), découverte en 2014 dans l'implémentation OpenSSL de l'extension *Heartbeat* de TLS, illustre bien la différence entre une faille de conception cryptographique et une faille d'implémentation logicielle : le protocole TLS lui-même n'était pas en cause, mais un défaut de vérification de longueur dans le code de traitement des messages Heartbeat permettait à un attaquant distant de lire, message après message, des fragments de la mémoire du serveur au-delà des données censées être renvoyées — pouvant potentiellement exposer des clés privées, des identifiants de session ou d'autres données sensibles présentes en mémoire au moment de l'attaque. Ce cas, largement documenté publiquement, rappelle qu'un protocole cryptographique correctement conçu reste entièrement dépendant de la rigueur de son implémentation logicielle : la meilleure spécification ne protège pas contre un bug d'implémentation dans le code qui la met en œuvre.

## 6. La menace post-quantique et la migration en cours

L'algorithme de Shor (rappel : module *Cryptanalyse*, chapitre 5), exécuté sur un ordinateur quantique suffisamment puissant, casserait RSA, Diffie-Hellman et ECC classiques. Bien qu'un tel ordinateur ne soit pas disponible aujourd'hui à l'échelle nécessaire, deux considérations motivent une action dès maintenant :

- **« Harvest now, decrypt later »** : un adversaire peut enregistrer aujourd'hui du trafic chiffré intercepté, dans l'espoir de le déchiffrer plus tard une fois l'informatique quantique suffisamment mature — un risque réel pour des données à confidentialité longue durée.
- **Délai de migration** : le déploiement d'une nouvelle famille cryptographique à l'échelle d'Internet prend typiquement plusieurs années.

Le NIST a standardisé en 2024 des algorithmes post-quantiques (**ML-KEM**, anciennement Kyber, pour l'échange de clé ; **ML-DSA**, anciennement Dilithium, pour la signature), dont l'intégration progressive dans TLS et d'autres protocoles est en cours, souvent sous forme **hybride** (combinant un mécanisme classique et un mécanisme post-quantique, pour ne pas dépendre uniquement de la maturité encore récente de ces nouveaux algorithmes).

### 6.1 Pourquoi le chiffrement symétrique est moins directement menacé

L'algorithme de Shor s'attaque spécifiquement aux problèmes structurés sur lesquels reposent RSA (factorisation) et Diffie-Hellman/ECC (logarithme discret) : ce sont des problèmes possédant une structure algébrique que l'algorithme quantique sait exploiter. Le chiffrement symétrique (AES) et les fonctions de hachage (SHA-2, SHA-3) ne reposent pas sur ce type de structure ; ils restent exposés uniquement à l'algorithme de Grover, qui offre au mieux une accélération quadratique de la recherche exhaustive (réduisant en théorie la sécurité effective d'AES-128 à un niveau comparable à AES-64, ce qui reste hors de portée pratique). C'est pourquoi les recommandations actuelles de transition post-quantique se concentrent sur l'échange de clé et la signature (RSA, Diffie-Hellman, ECC, ECDSA), tandis qu'il est généralement conseillé, par précaution, d'utiliser des tailles de clé symétrique déjà robustes (AES-256 plutôt qu'AES-128) pour les données dont la confidentialité doit être garantie sur le très long terme.

### 6.2 Familles d'algorithmes post-quantiques standardisés

Les algorithmes post-quantiques standardisés par le NIST reposent sur des familles mathématiques différentes de celles utilisées par RSA/ECC, réputées résistantes aux algorithmes quantiques connus à ce jour :

| Algorithme (nom NIST) | Ancien nom | Usage | Famille mathématique |
|---|---|---|---|
| ML-KEM | Kyber | Encapsulation de clé (échange) | Réseaux euclidiens structurés (*lattices*) |
| ML-DSA | Dilithium | Signature numérique | Réseaux euclidiens structurés (*lattices*) |
| SLH-DSA | SPHINCS+ | Signature numérique | Fonctions de hachage uniquement |

La diversité de ces familles (réseaux euclidiens pour deux d'entre elles, hachage seul pour la troisième) répond à une logique de diversification du risque similaire à celle motivant la coexistence de SHA-2 et SHA-3 (chapitre 3) : si une faiblesse inattendue venait à être découverte dans une famille mathématique, une alternative reposant sur des fondements différents resterait disponible.

### 6.3 Mode hybride : pourquoi ne pas basculer directement au post-quantique

Un déploiement post-quantique **hybride** combine, pour un même échange de clé, un mécanisme classique (par exemple X25519) et un mécanisme post-quantique (par exemple ML-KEM), le secret final étant dérivé de la combinaison des deux. Tant qu'au moins un des deux mécanismes reste sûr, le secret combiné reste sûr. Cette approche prudente permet de commencer à déployer une protection contre la menace « harvest now, decrypt later » dès aujourd'hui, sans dépendre uniquement de la confiance encore relativement récente accordée à ces nouveaux algorithmes standardisés, dont l'examen public continue de s'approfondir avec le temps — exactement le même raisonnement de prudence collective qui, historiquement, a présidé à l'adoption progressive d'AES ou de SHA-3 après leurs propres processus de sélection publique.

## 7. Pièges d'implémentation courants

- **Laisser des suites cryptographiques obsolètes activées côté serveur** « au cas où », par excès de compatibilité avec d'anciens clients, plutôt que de les désactiver explicitement.
- **Désactiver temporairement la vérification de certificat pour faciliter le développement ou le débogage**, et oublier de réactiver cette vérification avant la mise en production — une source récurrente d'incidents réels.
- **Ne pas surveiller la date d'expiration des certificats**, provoquant des interruptions de service évitables lorsqu'un certificat expire sans avoir été renouvelé à temps.
- **Traiter la migration post-quantique comme un projet à reporter indéfiniment**, alors que le risque « harvest now, decrypt later » s'applique dès aujourd'hui aux données interceptées, même si leur déchiffrement effectif n'interviendrait que plus tard.

## À retenir

- TLS combine, de façon opérationnelle, échange de clé asymétrique, authentification par certificat et chiffrement symétrique authentifié — la synthèse pratique de tout ce module.
- TLS 1.3 réduit la latence d'établissement de connexion (1-RTT, voire 0-RTT avec des précautions spécifiques contre le rejeu) tout en supprimant les suites cryptographiques jugées faibles.
- La confidentialité persistante (clés éphémères ECDHE) protège les échanges passés même en cas de compromission future d'une clé long terme, contrairement à l'ancien transport de clé RSA statique.
- Une faille d'implémentation (Heartbleed) peut compromettre la sécurité d'un protocole pourtant correctement conçu : la rigueur d'implémentation compte autant que la conception cryptographique elle-même.
- La transition post-quantique est engagée par anticipation, notamment face au risque « harvest now, decrypt later » sur des données sensibles à long terme ; elle cible en priorité l'échange de clé et la signature, moins directement menacés côté chiffrement symétrique.
- Le déploiement hybride (classique + post-quantique) permet une transition prudente, sans dépendre uniquement de la maturité encore récente des nouveaux algorithmes standardisés.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
