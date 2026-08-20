# Chapitre 5 — Signatures numériques et infrastructures à clés publiques (PKI)

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=06-cryptographie/slides/05-signatures-pki.txt){ target=_blank }

## 1. Principe de la signature numérique

La signature numérique fournit **authenticité**, **intégrité** et **non-répudiation** : le signataire utilise sa clé privée pour produire une signature liée à un message, que quiconque peut vérifier avec la clé publique correspondante, sans pouvoir produire lui-même une signature valide.

### Schéma général

1. Calculer l'empreinte du message : `h = H(message)` (fonction de hachage, chapitre 3).
2. Signer l'empreinte avec la clé privée : `signature = Sign(clé privée, h)`.
3. Vérification : recalculer `H(message)` et vérifier la signature avec la clé publique.

Signer l'empreinte plutôt que le message entier permet de traiter des messages de taille arbitraire avec des opérations cryptographiques de taille fixe, et rend toute modification du message détectable (l'empreinte changerait).

### 1.1 Non-répudiation : ce qu'elle apporte concrètement par rapport à un MAC

Rappel du chapitre 3 : un MAC prouve qu'un message provient d'une partie possédant la clé secrète partagée, mais comme les deux parties (émetteur et vérificateur) possèdent cette même clé, aucune des deux ne peut prouver, face à un tiers, laquelle des deux a réellement produit le MAC. Avec une signature numérique, seule la clé **privée**, détenue par une seule partie, permet de produire une signature valide ; la clé **publique**, utilisée pour la vérification, ne permet à personne de forger une signature. Un tiers (par exemple un juge, ou un système d'audit) peut donc vérifier une signature avec la seule clé publique et en déduire, avec une confiance justifiée, que seule la partie détenant la clé privée correspondante a pu la produire — c'est cette asymétrie qui fonde la non-répudiation.

### 1.2 Signature avec récupération de message vs signature avec appendice

La plupart des schémas modernes (RSA-PSS, ECDSA, Ed25519) sont des **signatures avec appendice** : la signature est une valeur distincte, transmise à côté du message, et sa vérification requiert de disposer du message complet pour en recalculer l'empreinte. Il existe aussi historiquement des schémas à **récupération de message**, où le message (ou une partie) peut être reconstitué directement à partir de la signature elle-même ; ces schémas restent marginaux en pratique face aux schémas à appendice, plus flexibles pour des messages de taille arbitraire.

## 2. RSA-PSS

Schéma de signature basé sur RSA avec un padding probabiliste (PSS — Probabilistic Signature Scheme), aujourd'hui recommandé pour la signature (par opposition à un ancien schéma déterministe PKCS#1 v1.5, encore présent mais moins recommandé pour de nouveaux déploiements).

### 2.1 Pourquoi « probabiliste » est préférable à « déterministe » pour signer

PSS introduit un sel aléatoire dans le calcul de la signature, de sorte que signer deux fois le même message avec la même clé privée produit deux signatures différentes (mais toutes deux valides à la vérification). Ce comportement probabiliste renforce la sécurité de la preuve mathématique associée au schéma (résistance démontrée sous des hypothèses standards, dans un cadre de preuve de sécurité appelé « modèle de l'oracle aléatoire »), alors que PKCS#1 v1.5, plus ancien et déterministe, s'appuie sur une analyse de sécurité moins directe, bien qu'aucune attaque pratique rédhibitoire ne soit connue contre une implémentation correcte pour la signature (la situation est différente pour le chiffrement RSA avec PKCS#1 v1.5, où des attaques structurelles bien documentées existent, d'où l'exigence d'OAEP au chapitre 4).

```bash
# Génération d'une paire de clés RSA, signature et vérification avec RSA-PSS via OpenSSL
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:3072 -out rsa_prive.pem
openssl pkey -in rsa_prive.pem -pubout -out rsa_publique.pem

openssl dgst -sha256 -sign rsa_prive.pem -sigopt rsa_padding_mode:pss \
    -out message.sig message.txt

openssl dgst -sha256 -verify rsa_publique.pem -sigopt rsa_padding_mode:pss \
    -signature message.sig message.txt
```

## 3. ECDSA et Ed25519

- **ECDSA** : signature sur courbe elliptique, largement déployée (certificats TLS, Bitcoin). Point de vigilance : la sécurité d'ECDSA dépend crucialement d'un nombre aléatoire (« nonce ») unique et non prévisible à chaque signature — un nonce réutilisé ou prévisible permet de retrouver directement la clé privée (illustré par des compromissions réelles, notamment sur certaines consoles de jeu et portefeuilles Bitcoin mal implémentés).
- **Ed25519** : schéma de signature moderne basé sur la courbe Curve25519, déterministe (ne dépend pas d'un aléa externe à chaque signature, éliminant le risque de nonce faible), rapide et largement recommandé pour les nouveaux déploiements (SSH, protocoles modernes).

### 3.1 Pourquoi un nonce ECDSA faible expose directement la clé privée

Sans entrer dans le détail complet de la construction, l'intuition est la suivante : la signature ECDSA combine la clé privée `d` et un nonce secret `k` de telle façon que si deux signatures différentes sont produites avec le **même** `k` (ou avec des valeurs de `k` liées par une relation connue de l'attaquant, par exemple issues d'un générateur pseudo-aléatoire de mauvaise qualité), un simple système d'équations permet de retrouver `k`, puis `d`, à partir des deux signatures et des messages correspondants — sans jamais casser directement le logarithme discret sous-jacent. C'est exactement le mécanisme exploité dans des compromissions réelles et documentées : une console de jeu grand public dont l'implémentation ECDSA réutilisait un nonce fixe au lieu d'un nonce aléatoire a ainsi vu sa clé de signature de firmware extraite ; plusieurs portefeuilles de cryptomonnaie utilisant un générateur aléatoire défaillant sur certains environnements mobiles ont subi le même type de compromission, avec un impact financier direct pour les utilisateurs concernés.

### 3.2 Signature déterministe : deux approches pour éviter ce piège

- **RFC 6979** : dérive un nonce ECDSA de façon déterministe à partir de la clé privée et du message (via HMAC), en éliminant la dépendance à un générateur aléatoire externe défaillant au moment de la signature, tout en restant compatible avec ECDSA standard.
- **Ed25519** : élimine le problème par construction même du schéma, le nonce étant systématiquement dérivé de façon déterministe et interne à l'algorithme dès sa conception, sans configuration supplémentaire requise de la part de l'implémenteur — un facteur important dans sa popularité pour les nouveaux déploiements (clés SSH, signatures de paquets logiciels, protocoles comme WireGuard).

```bash
# Génération d'une paire de clés Ed25519, signature et vérification avec OpenSSL
openssl genpkey -algorithm ED25519 -out ed25519_prive.pem
openssl pkey -in ed25519_prive.pem -pubout -out ed25519_publique.pem

openssl pkeyutl -sign -inkey ed25519_prive.pem -rawin -in message.txt -out message.sig
openssl pkeyutl -verify -pubin -inkey ed25519_publique.pem -rawin -in message.txt -sigfile message.sig
```

## 4. Certificats numériques

Un certificat numérique lie une identité (personne, serveur, organisation) à une clé publique, la liaison étant elle-même garantie par la signature numérique d'une **autorité de certification (AC/CA)** de confiance. Format standard le plus répandu : **X.509**.

### Contenu typique d'un certificat X.509

- Identité du sujet (ex. nom de domaine pour un certificat de serveur).
- Clé publique du sujet.
- Identité de l'autorité de certification émettrice.
- Période de validité.
- Signature numérique de l'AC sur l'ensemble de ces informations.

### 4.1 Extensions courantes d'un certificat X.509

Au-delà des champs de base, un certificat X.509 moderne comporte des **extensions** qui précisent son usage :

- **Subject Alternative Name (SAN)** : liste des noms de domaine (ou adresses IP) couverts par le certificat ; les navigateurs modernes vérifient ce champ en priorité pour valider un nom de domaine, plutôt que l'ancien champ *Common Name*.
- **Key Usage / Extended Key Usage** : précise les usages autorisés de la clé (signature de certificat, authentification serveur, authentification client, signature de code, etc.), limitant le risque qu'un certificat émis pour un usage soit détourné vers un autre.
- **Basic Constraints** : indique si le certificat peut lui-même signer d'autres certificats (certificat d'AC) ou non (certificat final, dit « feuille »), une distinction essentielle pour la chaîne de confiance (section 5).

```bash
# Inspection du contenu d'un certificat X.509 avec OpenSSL
openssl x509 -in certificat.pem -noout -text

# Vérification d'un certificat serveur en se connectant directement en TLS
openssl s_client -connect exemple.org:443 -servername exemple.org </dev/null
```

## 5. Infrastructure à clés publiques (PKI)

Ensemble des composants et procédures permettant de créer, distribuer, vérifier et révoquer des certificats :

- **Autorité de certification (AC/CA)** : émet et signe les certificats.
- **Autorité d'enregistrement (RA)** : vérifie l'identité du demandeur avant émission.
- **Chaîne de confiance** : un certificat intermédiaire est lui-même signé par une AC racine, dont la clé publique est préinstallée (« ancre de confiance ») dans les navigateurs et systèmes d'exploitation.
- **Révocation** : mécanismes de liste de révocation (CRL) ou de vérification en ligne (OCSP), permettant d'invalider un certificat compromis avant sa date d'expiration normale.

### 5.1 Pourquoi une chaîne intermédiaire plutôt qu'une signature directe par l'AC racine

Les AC racines ne signent en pratique presque jamais directement les certificats des serveurs finaux. Leur clé privée, extrêmement sensible, est conservée hors ligne dans des conditions de sécurité physique et procédurale très strictes, et n'est utilisée que pour signer un nombre restreint de **certificats d'AC intermédiaires**. Ce sont ces AC intermédiaires, dont la clé privée reste en ligne pour un usage opérationnel, qui signent au quotidien les certificats des serveurs finaux. Ce découplage limite fortement les conséquences d'une éventuelle compromission d'une AC intermédiaire (elle peut être révoquée sans remettre en cause la racine elle-même) et illustre un principe de défense en profondeur appliqué directement à la gestion de clés (module *Fondamentaux*).

### 5.2 CRL vs OCSP

- **CRL** (Certificate Revocation List) : liste, publiée périodiquement par l'AC, de tous les numéros de série des certificats révoqués avant leur expiration normale. Le client doit télécharger cette liste (potentiellement volumineuse) et vérifier localement la présence du certificat concerné.
- **OCSP** (Online Certificate Status Protocol) : le client interroge directement un serveur de l'AC pour connaître le statut d'un certificat précis, en temps quasi réel, sans avoir à télécharger une liste complète. Une variante, **OCSP stapling**, permet au serveur lui-même de joindre une réponse OCSP récente et signée directement lors de la poignée de main TLS, évitant au client une requête réseau supplémentaire vers l'AC et réduisant l'exposition d'information sur les sites visités par le client.

### Vérification d'un certificat côté client

1. Le certificat est-il signé par une AC de confiance (remontée de la chaîne jusqu'à une ancre de confiance) ?
2. Le certificat est-il dans sa période de validité ?
3. Le certificat correspond-il bien à l'identité attendue (ex. nom de domaine) ?
4. Le certificat a-t-il été révoqué ?

Une faille dans n'importe laquelle de ces vérifications (ex. accepter un certificat expiré, ou ne pas vérifier le nom de domaine) annule la protection apportée par toute la chaîne cryptographique — un point classique d'audit de sécurité applicative (module *Audit organisation et technique*, chapitre 5).

## 6. Modèles alternatifs

- **Web of Trust** (utilisé historiquement par PGP/GPG) : confiance distribuée entre pairs plutôt que hiérarchique, sans autorité centrale.
- **Certificate Transparency** : registres publics et vérifiables des certificats émis, permettant de détecter des émissions frauduleuses par une AC compromise ou négligente.

### 6.1 Web of Trust : principe et limites

Dans le modèle du Web of Trust, chaque utilisateur signe lui-même les clés publiques d'autres utilisateurs dont il a personnellement vérifié l'identité (par exemple lors d'une rencontre en personne, une pratique parfois formalisée sous le nom de « key signing party »). La confiance accordée à une clé inconnue dépend alors du réseau de signatures qui la relie, directement ou indirectement, à des clés déjà connues de l'utilisateur. Ce modèle évite la dépendance à une autorité centrale unique, mais peine à s'appliquer à grande échelle (le web grand public), car il exige un effort de vérification manuel et social difficile à industrialiser — ce qui explique pourquoi le modèle hiérarchique des PKI X.509, malgré ses propres limites, reste dominant pour sécuriser le web.

### 6.2 Certificate Transparency : détecter une émission frauduleuse

Certificate Transparency repose sur des journaux publics, structurés en arbres de Merkle (chapitre 3), dans lesquels chaque certificat émis par une AC participante doit être enregistré de façon vérifiable et infalsifiable a posteriori. Les navigateurs modernes exigent, pour certains types de certificats, une preuve d'inclusion dans au moins un journal reconnu avant d'accorder leur confiance. Ce mécanisme permet à un titulaire de domaine, ou à des chercheurs en sécurité, de surveiller ces journaux publics et de détecter rapidement l'émission d'un certificat non sollicité pour un domaine donné — une défense complémentaire face à une AC compromise ou négligente qui émettrait un certificat frauduleux sans que le titulaire légitime en soit informé par ailleurs.

## 7. Pièges d'implémentation courants

- **Vérifier la signature d'un certificat sans vérifier la chaîne complète jusqu'à une ancre de confiance**, ou accepter une AC intermédiaire non correctement liée à une racine reconnue.
- **Ignorer le résultat de la vérification de révocation** (CRL/OCSP), ou échouer silencieusement en cas d'indisponibilité du service de révocation plutôt que d'adopter une politique explicite (accepter ou refuser par défaut selon le niveau de risque).
- **Réutiliser un nonce ECDSA**, ou utiliser un générateur aléatoire de mauvaise qualité pour le produire — l'un des pièges les plus documentés en cryptographie appliquée, illustré par plusieurs compromissions réelles (section 3.1).
- **Confondre validation du certificat et validation du contenu applicatif** : un certificat valide prouve l'identité du serveur, pas la légitimité ou l'innocuité du contenu qu'il sert.

## À retenir

- La signature numérique apporte la non-répudiation, absente d'un simple MAC (chapitre 3), grâce à l'asymétrie entre clé privée (signature) et clé publique (vérification).
- RSA-PSS est recommandé pour la signature RSA face à l'ancien schéma déterministe PKCS#1 v1.5.
- ECDSA exige un nonce aléatoire de haute qualité à chaque signature, sous peine d'exposer directement la clé privée ; Ed25519 (et RFC 6979 pour ECDSA) éliminent ce risque par construction déterministe.
- Une PKI repose sur une chaîne de confiance hiérarchique (racine hors ligne, intermédiaires en ligne) ; sa robustesse dépend entièrement de la rigueur de vérification à chaque maillon (identité, validité, révocation).
- OCSP stapling et Certificate Transparency renforcent la PKI classique en accélérant la détection de révocation et d'émission frauduleuse.
- Le Web of Trust offre une alternative décentralisée à la PKI hiérarchique, mais peine à s'appliquer à l'échelle du web grand public.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
