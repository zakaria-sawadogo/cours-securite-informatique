# Chapitre 3 — Authentification et gestion de sessions

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=09-securite-des-applications/slides/03-authentification-gestion-sessions.txt){ target=_blank }

## 1. Authentification : rappels et bonnes pratiques

### Stockage des mots de passe

Rappel des modules *Cryptanalyse* et *Cryptographie* : un mot de passe ne doit jamais être stocké en clair, ni haché avec une fonction générique rapide seule (SHA-256 brut) ; utiliser une fonction dédiée et volontairement lente, salée (bcrypt, scrypt, Argon2).

Cette exigence découle directement d'une différence de finalité entre les fonctions de hachage génériques (conçues pour être rapides, ex. vérifier l'intégrité d'un fichier volumineux) et les fonctions dédiées au mot de passe (conçues au contraire pour être lentes et coûteuses en ressources, précisément pour ralentir un attaquant tentant de tester un grand nombre de mots de passe candidats après le vol d'une base de hachages) :

```python
# Vulnérable : hachage rapide, sans sel, trivialement cassable
# par table arc-en-ciel ou par force brute massivement parallélisée
motdepasse_stocke = sha256(motdepasse).hexdigest()

# Correct : fonction dédiée, lente, avec sel généré aléatoirement
# et intégré automatiquement au résultat stocké
motdepasse_stocke = bcrypt.hashpw(motdepasse, bcrypt.gensalt(rounds=12))
```

Trois propriétés distinctes doivent être réunies :

- **Sel (*salt*)** unique par utilisateur, empêchant qu'un même mot de passe produise le même hachage pour deux comptes différents, et rendant inefficaces les tables précalculées (tables arc-en-ciel).
- **Facteur de travail (*work factor*) réglable** : le nombre d'itérations internes de l'algorithme (paramètre `rounds` ci-dessus pour bcrypt) peut être augmenté progressivement à mesure que la puissance de calcul disponible pour un attaquant augmente, sans changer d'algorithme.
- **Résistance au calcul massivement parallèle (GPU/ASIC)** : Argon2 et scrypt, en plus d'être lents, consomment volontairement une quantité importante de mémoire, ce qui limite l'efficacité d'un parallélisme massif bon marché typique des architectures GPU, contrairement à des fonctions peu gourmandes en mémoire.

### Politique de mots de passe

Les recommandations actuelles (NIST SP 800-63B notamment) privilégient une **longueur minimale suffisante** (12 caractères ou plus) plutôt que des règles de composition complexes peu efficaces en pratique (les utilisateurs contournent des règles trop contraignantes par des motifs prévisibles) ; vérification contre des listes de mots de passe déjà compromis (recommandée) ; suppression de l'expiration périodique forcée sans raison (mesure historiquement répandue mais aujourd'hui considérée contre-productive, car elle pousse à des variations prévisibles).

Le raisonnement sous-jacent à cette évolution des recommandations mérite d'être compris plutôt que simplement mémorisé : une règle imposant, par exemple, au moins une majuscule, un chiffre et un caractère spécial, conduit en pratique une majorité d'utilisateurs à produire des motifs très prévisibles (une majuscule en première position, un chiffre ou un caractère spécial en fin de chaîne), motifs qu'un attaquant expérimenté intègre directement dans ses dictionnaires d'attaque. Une phrase de passe longue mais simple à composer (plusieurs mots assemblés) offre, à effort de mémorisation comparable pour l'utilisateur, un espace de recherche bien plus large pour l'attaquant.

### Authentification multi-facteurs (MFA)

Combine au moins deux catégories parmi : ce que l'utilisateur **sait** (mot de passe), ce qu'il **possède** (application d'authentification, clé de sécurité physique), ce qu'il **est** (biométrie). Réduit fortement l'impact d'un mot de passe compromis (par phishing, fuite de données, réutilisation).

| Facteur | Exemple | Point de vigilance |
|---|---|---|
| Code à usage unique par SMS | code reçu par message texte | vulnérable au détournement de numéro de téléphone (SIM swapping) et à l'interception réseau ; facteur le moins robuste des trois exemples |
| Application d'authentification (TOTP) | code à usage unique généré localement, basé sur un secret partagé et l'heure courante | ne dépend pas du réseau téléphonique ; nécessite néanmoins que le secret initial soit transmis et stocké de façon sûre lors de l'enregistrement |
| Clé de sécurité physique (FIDO2/WebAuthn) | clé USB ou NFC dédiée | résistant au phishing par construction, car la clé vérifie cryptographiquement l'origine du site avant de répondre, contrairement aux deux facteurs précédents |

Un point de vigilance pédagogique important : la MFA réduit le risque associé à un facteur compromis, mais n'élimine pas le besoin des autres bonnes pratiques (stockage correct des mots de passe, limitation des tentatives) ; elle s'ajoute à la défense en profondeur plutôt que de la remplacer.

## 2. Défaillances d'authentification fréquentes

| Défaillance | Risque |
|---|---|
| Absence de limitation du nombre de tentatives | attaque par force brute ou par dictionnaire praticable |
| Messages d'erreur distinguant « utilisateur inconnu » de « mot de passe incorrect » | permet à un attaquant d'énumérer les comptes existants |
| Absence de MFA sur les comptes à privilèges | un mot de passe compromis suffit à compromettre un compte critique |
| Réinitialisation de mot de passe mal sécurisée | jeton de réinitialisation prévisible ou sans expiration, permettant une prise de contrôle de compte |
| **Credential stuffing** non contré | réutilisation automatisée de couples identifiant/mot de passe issus de fuites de données antérieures sur d'autres services |

### Illustration : énumération de comptes par message d'erreur

```
# Vulnérable : deux messages distincts révèlent l'existence du compte
"Ce nom d'utilisateur n'existe pas."
"Mot de passe incorrect pour cet utilisateur."

# Correct : un message unique, indépendant de la cause réelle de l'échec
"Identifiant ou mot de passe incorrect."
```

Le même principe de non-distinction s'applique au *temps de réponse* : si la vérification du mot de passe n'est effectuée que lorsque le login existe (court-circuit), le temps de réponse diffère selon les cas et peut, à lui seul, permettre à un attaquant d'énumérer les comptes existants même avec un message d'erreur unique — d'où l'intérêt de toujours effectuer une opération de temps comparable (par exemple une vérification de hachage factice) même lorsque le login n'existe pas.

### Réinitialisation de mot de passe : exigences

Un flux de réinitialisation de mot de passe doit réunir plusieurs propriétés simultanément pour rester sûr : jeton **imprévisible** (généré par un générateur aléatoire cryptographiquement sûr, jamais dérivé d'une information devinable comme l'horodatage), **à usage unique**, **de durée de vie limitée** (quelques minutes à quelques heures selon la sensibilité), et **invalidé** dès qu'un nouveau mot de passe a été défini ou qu'une nouvelle demande de réinitialisation a été émise. L'e-mail ou le canal utilisé pour transmettre ce jeton doit lui-même être considéré comme faisant partie du périmètre de sécurité du compte.

### Limitation des tentatives et *credential stuffing*

Une simple limitation du nombre de tentatives par compte ne suffit pas à contrer le *credential stuffing*, où l'attaquant teste un couple identifiant/mot de passe différent à chaque tentative (issus d'une fuite de données antérieure sur un service tiers), répartissant volontairement ses tentatives sur un grand nombre de comptes cibles pour rester sous le seuil de détection par compte. Des contre-mesures complémentaires sont nécessaires : limitation par adresse IP ou empreinte de client, détection de motifs de trafic automatisés, et surtout MFA, qui rend inopérant un mot de passe correct mais volé si le second facteur n'est pas également compromis.

## 3. Gestion de sessions

Après authentification, un **identifiant de session** (généralement transmis via un cookie) permet à l'application de reconnaître l'utilisateur sur les requêtes suivantes sans redemander ses identifiants à chaque fois.

### Exigences de sécurité d'un identifiant de session

- **Imprévisibilité** : généré par un générateur aléatoire cryptographiquement sûr, suffisamment long pour résister à une attaque par force brute.
- **Régénération après authentification** : un identifiant de session attribué avant authentification ne doit jamais rester valide après (prévention de la **fixation de session**, où un attaquant impose à sa victime un identifiant de session qu'il connaît à l'avance).
- **Expiration** : durée de vie limitée, avec expiration après une période d'inactivité et invalidation explicite à la déconnexion.
- **Attributs de cookie sécurisés** : `Secure` (transmission uniquement en HTTPS), `HttpOnly` (inaccessible au JavaScript côté client, limitant l'impact d'un XSS — chapitre 4), `SameSite` (limite l'envoi automatique du cookie lors de requêtes intersites, contre-mesure au CSRF — chapitre 4).

### Illustration : attaque par fixation de session

```
1. L'attaquant obtient un identifiant de session valide mais non
   authentifié auprès du site cible (ex. simplement en visitant la page
   de connexion), puis force la victime à l'utiliser (lien piégé
   contenant l'identifiant, ou site vulnérable acceptant un identifiant
   fourni en paramètre d'URL).
2. La victime, sans le savoir, s'authentifie sur le site en utilisant
   cet identifiant de session déjà connu de l'attaquant.
3. Si le serveur ne régénère pas l'identifiant de session après
   l'authentification, l'identifiant reste le même avant et après :
   l'attaquant, qui le connaissait déjà, dispose désormais d'une
   session authentifiée en tant que la victime.
```

**Contre-mesure directe** : générer systématiquement un *nouvel* identifiant de session immédiatement après une authentification réussie, et invalider l'ancien — de sorte qu'un identifiant connu avant authentification n'ait plus aucune valeur après.

### Tableau récapitulatif des attributs de cookie de session

| Attribut | Effet | Contre quoi |
|---|---|---|
| `Secure` | le cookie n'est transmis que sur une connexion HTTPS | interception en clair sur un réseau non chiffré |
| `HttpOnly` | le cookie est inaccessible depuis un script JavaScript côté client | vol du cookie de session via une vulnérabilité XSS (chapitre 4) |
| `SameSite=Strict` / `Lax` | limite l'envoi automatique du cookie lors d'une requête provenant d'un autre site | falsification de requête intersite, CSRF (chapitre 4) |
| Durée de vie limitée | expiration automatique après une période fixe ou d'inactivité | usage prolongé d'une session volée ou oubliée sur un poste partagé |

## 4. Authentification fédérée et jetons (aperçu)

De nombreuses applications modernes délèguent l'authentification à un fournisseur d'identité tiers (OAuth 2.0, OpenID Connect) plutôt que de gérer elles-mêmes des mots de passe. Ces protocoles introduisent leurs propres risques (mauvaise validation d'un jeton, absence de vérification de la signature d'un JWT, confusion entre autorisation et authentification) qui nécessitent une compréhension précise du protocole utilisé plutôt qu'une implémentation ad hoc.

Une distinction conceptuelle essentielle, souvent source de confusion : **OAuth 2.0** est, à l'origine, un protocole d'*autorisation* (accorder à une application tierce un accès délégué et limité à des ressources, sans partager le mot de passe), tandis qu'**OpenID Connect (OIDC)** est une couche construite au-dessus d'OAuth 2.0 spécifiquement destinée à l'*authentification* (prouver l'identité de l'utilisateur). Utiliser OAuth 2.0 seul comme mécanisme d'authentification, sans la couche OIDC, est une erreur de conception fréquente pouvant conduire à des failles d'usurpation d'identité.

### JWT (JSON Web Token) : points de vigilance

- Toujours vérifier la **signature** du jeton reçu, avec l'algorithme attendu explicitement fixé côté serveur (ne jamais faire confiance à l'algorithme annoncé dans l'en-tête du jeton lui-même — une classe de vulnérabilité réelle où un attaquant modifie l'algorithme annoncé en `none` ou substitue une clé publique en clé symétrique).
- Vérifier systématiquement l'expiration (`exp`) et l'émetteur attendu.
- Ne jamais stocker d'informations sensibles non chiffrées dans le contenu d'un JWT, qui n'est qu'encodé (Base64), pas chiffré, et donc lisible par quiconque intercepte le jeton.

### Illustration : attaque par confusion d'algorithme (« alg: none »)

Un JWT est composé de trois parties encodées en Base64 et séparées par des points : un en-tête décrivant l'algorithme de signature, une charge utile (les données), et une signature. L'en-tête d'un jeton légitime ressemble à :

```json
{ "alg": "HS256", "typ": "JWT" }
```

Si le code de vérification côté serveur lit l'algorithme annoncé *dans le jeton lui-même* plutôt que d'imposer un algorithme fixe attendu, un attaquant peut forger un jeton dont l'en-tête indique :

```json
{ "alg": "none", "typ": "JWT" }
```

Certaines bibliothèques mal utilisées acceptent alors le jeton comme valide sans vérifier de signature du tout, puisque l'algorithme annoncé indique explicitement l'absence de signature. **Contre-mesure** : le serveur doit toujours imposer explicitement, dans sa propre configuration, l'algorithme de signature attendu (jamais lu depuis le jeton reçu), et rejeter tout jeton n'utilisant pas cet algorithme précis.

### Jetons d'accès et jetons de rafraîchissement

Une bonne pratique répandue consiste à séparer un **jeton d'accès** (*access token*), de courte durée de vie, utilisé pour chaque requête à une ressource protégée, d'un **jeton de rafraîchissement** (*refresh token*), de durée de vie plus longue, utilisé uniquement pour obtenir un nouveau jeton d'accès sans redemander les identifiants complets à l'utilisateur. Cette séparation limite la fenêtre d'exploitation d'un jeton d'accès intercepté, tout en évitant de redemander une authentification complète trop fréquemment ; le jeton de rafraîchissement, plus sensible car plus longuement valide, doit bénéficier d'une protection de stockage renforcée (jamais accessible à un script côté client, par exemple).

## À retenir

- Une politique de mot de passe moderne privilégie la longueur et la vérification contre des mots de passe déjà compromis plutôt que des règles de composition contraignantes et peu efficaces ; le stockage repose sur une fonction dédiée, lente et salée (bcrypt, scrypt, Argon2), jamais sur un hachage rapide générique.
- La MFA combine des facteurs de nature différente (savoir, possession, être) et réduit fortement l'impact d'un mot de passe compromis, sans dispenser des autres bonnes pratiques d'authentification.
- L'énumération de comptes (par message d'erreur ou par temps de réponse) et le *credential stuffing* sont deux défaillances distinctes nécessitant des contre-mesures spécifiques (message d'erreur unique, temps de réponse constant, limitation multi-niveaux, MFA).
- La gestion de session exige régénération après authentification (prévention de la fixation de session), expiration maîtrisée, et attributs de cookie sécurisés (Secure, HttpOnly, SameSite).
- L'authentification fédérée (OAuth pour l'autorisation, OIDC pour l'authentification, JWT comme format de jeton) déplace le risque vers une bonne compréhension du protocole et une vérification rigoureuse des jetons (signature avec algorithme imposé côté serveur, expiration), plutôt que de l'éliminer.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
