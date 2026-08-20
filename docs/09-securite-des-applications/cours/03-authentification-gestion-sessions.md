# Chapitre 3 — Authentification et gestion de sessions

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=09-securite-des-applications/slides/03-authentification-gestion-sessions.txt){ target=_blank }

## 1. Authentification : rappels et bonnes pratiques

### Stockage des mots de passe

Rappel des modules *Cryptanalyse* et *Cryptographie* : un mot de passe ne doit jamais être stocké en clair, ni haché avec une fonction générique rapide seule (SHA-256 brut) ; utiliser une fonction dédiée et volontairement lente, salée (bcrypt, scrypt, Argon2).

### Politique de mots de passe

Les recommandations actuelles (NIST SP 800-63B notamment) privilégient une **longueur minimale suffisante** (12 caractères ou plus) plutôt que des règles de composition complexes peu efficaces en pratique (les utilisateurs contournent des règles trop contraignantes par des motifs prévisibles) ; vérification contre des listes de mots de passe déjà compromis (recommandée) ; suppression de l'expiration périodique forcée sans raison (mesure historiquement répandue mais aujourd'hui considérée contre-productive, car elle pousse à des variations prévisibles).

### Authentification multi-facteurs (MFA)

Combine au moins deux catégories parmi : ce que l'utilisateur **sait** (mot de passe), ce qu'il **possède** (application d'authentification, clé de sécurité physique), ce qu'il **est** (biométrie). Réduit fortement l'impact d'un mot de passe compromis (par phishing, fuite de données, réutilisation).

## 2. Défaillances d'authentification fréquentes

| Défaillance | Risque |
|---|---|
| Absence de limitation du nombre de tentatives | attaque par force brute ou par dictionnaire praticable |
| Messages d'erreur distinguant « utilisateur inconnu » de « mot de passe incorrect » | permet à un attaquant d'énumérer les comptes existants |
| Absence de MFA sur les comptes à privilèges | un mot de passe compromis suffit à compromettre un compte critique |
| Réinitialisation de mot de passe mal sécurisée | jeton de réinitialisation prévisible ou sans expiration, permettant une prise de contrôle de compte |
| **Credential stuffing** non contré | réutilisation automatisée de couples identifiant/mot de passe issus de fuites de données antérieures sur d'autres services |

## 3. Gestion de sessions

Après authentification, un **identifiant de session** (généralement transmis via un cookie) permet à l'application de reconnaître l'utilisateur sur les requêtes suivantes sans redemander ses identifiants à chaque fois.

### Exigences de sécurité d'un identifiant de session

- **Imprévisibilité** : généré par un générateur aléatoire cryptographiquement sûr, suffisamment long pour résister à une attaque par force brute.
- **Régénération après authentification** : un identifiant de session attribué avant authentification ne doit jamais rester valide après (prévention de la **fixation de session**, où un attaquant impose à sa victime un identifiant de session qu'il connaît à l'avance).
- **Expiration** : durée de vie limitée, avec expiration après une période d'inactivité et invalidation explicite à la déconnexion.
- **Attributs de cookie sécurisés** : `Secure` (transmission uniquement en HTTPS), `HttpOnly` (inaccessible au JavaScript côté client, limitant l'impact d'un XSS — chapitre 4), `SameSite` (limite l'envoi automatique du cookie lors de requêtes intersites, contre-mesure au CSRF — chapitre 4).

## 4. Authentification fédérée et jetons (aperçu)

De nombreuses applications modernes délèguent l'authentification à un fournisseur d'identité tiers (OAuth 2.0, OpenID Connect) plutôt que de gérer elles-mêmes des mots de passe. Ces protocoles introduisent leurs propres risques (mauvaise validation d'un jeton, absence de vérification de la signature d'un JWT, confusion entre autorisation et authentification) qui nécessitent une compréhension précise du protocole utilisé plutôt qu'une implémentation ad hoc.

### JWT (JSON Web Token) : points de vigilance

- Toujours vérifier la **signature** du jeton reçu, avec l'algorithme attendu explicitement fixé côté serveur (ne jamais faire confiance à l'algorithme annoncé dans l'en-tête du jeton lui-même — une classe de vulnérabilité réelle où un attaquant modifie l'algorithme annoncé en `none` ou substitue une clé publique en clé symétrique).
- Vérifier systématiquement l'expiration (`exp`) et l'émetteur attendu.
- Ne jamais stocker d'informations sensibles non chiffrées dans le contenu d'un JWT, qui n'est qu'encodé (Base64), pas chiffré, et donc lisible par quiconque intercepte le jeton.

## À retenir

- Une politique de mot de passe moderne privilégie la longueur et la vérification contre des mots de passe déjà compromis plutôt que des règles de composition contraignantes et peu efficaces.
- La gestion de session exige régénération après authentification, expiration maîtrisée, et attributs de cookie sécurisés (Secure, HttpOnly, SameSite).
- L'authentification fédérée (OAuth/OIDC/JWT) déplace le risque vers une bonne compréhension du protocole et une vérification rigoureuse des jetons, plutôt que de l'éliminer.
