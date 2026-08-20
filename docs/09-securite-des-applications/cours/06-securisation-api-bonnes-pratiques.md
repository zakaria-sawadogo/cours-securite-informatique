# Chapitre 6 — Sécurisation des API et bonnes pratiques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=09-securite-des-applications/slides/06-securisation-api-bonnes-pratiques.txt){ target=_blank }

## 1. Spécificités de la sécurité des API

Les API (notamment REST et GraphQL) sont devenues le principal mode d'intégration entre applications, exposant souvent directement une logique métier et des données sensibles, avec une surface d'attaque distincte des applications web traditionnelles (pas d'interface graphique intermédiaire filtrant implicitement certains usages).

## 2. OWASP API Security Top 10 (panorama)

Référentiel dédié, distinct du Top 10 applicatif général, structurant les risques spécifiques aux API :

- **Contrôle d'accès défaillant au niveau des objets (BOLA)** : un utilisateur accède aux données d'un autre utilisateur en modifiant simplement un identifiant dans la requête (ex. `GET /api/commandes/1234` accessible en changeant l'identifiant, sans vérification que la commande appartient bien à l'utilisateur authentifié) — variante API de l'IDOR déjà vu.
- **Authentification défaillante** : rappel du chapitre 3, appliqué spécifiquement aux jetons d'API.
- **Exposition excessive de données (excessive data exposure)** : l'API renvoie plus de champs que nécessaire, en s'appuyant sur le client pour filtrer l'affichage — un attaquant inspectant directement la réponse brute accède alors à des données qui n'auraient jamais dû être transmises.
- **Absence de limitation de ressources et de débit (rate limiting)** : permet des attaques par force brute, de déni de service, ou une extraction massive de données (scraping).
- **Contrôle d'accès défaillant au niveau des fonctions (BFLA)** : un utilisateur standard parvient à appeler un endpoint réservé aux administrateurs faute de vérification côté serveur (une restriction uniquement appliquée côté interface utilisateur n'est jamais suffisante).
- **Mauvaise configuration de sécurité** : en-têtes de sécurité manquants, messages d'erreur trop verbeux révélant des détails d'implémentation, CORS mal configuré (autorisant des origines non nécessaires).

## 3. Authentification et autorisation des API

- **Clés d'API** : simples à mettre en œuvre mais offrant peu de granularité ; à transmettre exclusivement via un en-tête HTTP (jamais dans l'URL, qui peut être journalisée ou mise en cache) et sur une connexion chiffrée.
- **OAuth 2.0** : cadre standard de délégation d'autorisation, permettant à une application tierce d'accéder à des ressources au nom d'un utilisateur sans jamais connaître son mot de passe.
- **JWT** : rappel du chapitre 3, avec vérification stricte de la signature et de l'expiration.
- **Principe du moindre privilège appliqué aux portées (scopes)** : une clé/jeton d'API ne devrait donner accès qu'aux opérations et ressources strictement nécessaires à l'usage prévu, pas à l'ensemble de l'API par défaut.

## 4. Validation systématique côté serveur

Principe absolu : **toute validation effectuée uniquement côté client (JavaScript) doit être considérée comme inexistante du point de vue de la sécurité**, car un attaquant peut appeler directement l'API en contournant totalement l'interface utilisateur. Toute règle de sécurité (autorisation, validation de format, limites métier) doit être appliquée et vérifiée côté serveur, la validation côté client n'étant qu'un confort d'expérience utilisateur, jamais un contrôle de sécurité.

## 5. Configuration CORS (Cross-Origin Resource Sharing)

Le mécanisme CORS définit quelles origines web sont autorisées à effectuer des requêtes cross-origin vers une API depuis un navigateur. Une configuration `Access-Control-Allow-Origin: *` combinée à l'autorisation d'envoi de cookies/identifiants est une mauvaise pratique fréquente, pouvant faciliter des attaques exploitant le navigateur d'un utilisateur authentifié depuis un site tiers malveillant.

## 6. Limitation de débit et quotas (rate limiting)

Mesure de défense en profondeur essentielle, souvent négligée : limiter le nombre de requêtes qu'un client (identifié par clé d'API, jeton ou adresse IP) peut effectuer dans une fenêtre de temps donnée, contrant à la fois les attaques par force brute sur l'authentification et l'extraction massive de données.

## 7. Journalisation et supervision des API

Journaliser les tentatives d'accès refusées, les erreurs d'authentification répétées, et les schémas d'usage anormaux (rappel du module *Gouvernance*, chapitre 4, sur les indicateurs de sécurité), pour permettre la détection d'une exploitation en cours et alimenter une réponse à incident.

## 8. Synthèse : une checklist de sécurisation d'API

1. Authentification forte et jetons correctement vérifiés (chapitre 3).
2. Autorisation vérifiée systématiquement à chaque appel, au niveau objet et fonction (jamais déléguée au client).
3. Validation stricte des entrées côté serveur, quelle que soit la validation déjà présente côté client.
4. Exposition minimale des données (ne renvoyer que les champs nécessaires).
5. Limitation de débit et quotas.
6. Configuration CORS restrictive et justifiée.
7. Chiffrement systématique des communications (TLS, module *Cryptographie*).
8. Journalisation et supervision des accès et anomalies.

## À retenir

- L'OWASP API Security Top 10 recense des risques spécifiques aux API (BOLA, BFLA, exposition excessive de données, absence de rate limiting), complémentaires du Top 10 applicatif général.
- Aucune validation ni autorisation effectuée uniquement côté client ne doit être considérée comme un contrôle de sécurité : tout doit être revérifié côté serveur.
- La sécurisation d'une API repose sur la combinaison systématique de plusieurs contrôles (authentification, autorisation, validation, limitation de débit, configuration, journalisation), en cohérence directe avec la défense en profondeur vue au module *Security by design*.
