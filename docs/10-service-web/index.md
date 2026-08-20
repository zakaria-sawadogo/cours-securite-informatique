# Service web

**Volume horaire :** 16h CM · 8h TP (24h)

## Présentation de la matière

Ce module aborde la conception, le développement et l'exploitation des services web, qui constituent aujourd'hui le mode d'interopérabilité dominant entre applications, qu'il s'agisse d'applications web, mobiles ou de services internes à une organisation. Il part du protocole HTTP lui-même pour aller jusqu'aux architectures distribuées à base de microservices, en couvrant en chemin la conception d'API REST, les formats d'échange, l'authentification, et la documentation/test d'API. Il constitue un prérequis technique naturel pour aborder en profondeur la sécurisation des API, déjà introduite dans le module *Sécurité des applications*.

## Objectifs pédagogiques

- Comprendre le fonctionnement du protocole HTTP et son rôle de socle des services web modernes.
- Concevoir une API REST respectant les principes du style architectural REST et les bonnes pratiques de conception reconnues par la profession.
- Comparer et choisir un format d'échange adapté (JSON, XML, SOAP) selon le contexte technique et historique d'un projet.
- Mettre en œuvre des mécanismes d'authentification et d'autorisation modernes (clés d'API, OAuth 2.0, JWT) et comprendre le rôle de CORS.
- Documenter formellement une API avec OpenAPI et mettre en place une stratégie de test adaptée (unitaire, intégration, contrat).
- Situer les architectures orientées services et microservices par rapport à l'architecture monolithique, et identifier les défis qu'elles introduisent.

## Prérequis

Une aisance de base en programmation (module *Algorithmique et Programmation en C*, transposable aux exemples Python utilisés dans ce module) est attendue. Ce module prépare directement le chapitre « Sécurisation des API » du module *Sécurité des applications* ; il est recommandé, mais non strictement obligatoire, de le suivre en amont ou en parallèle de ce dernier.

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction aux services web et au protocole HTTP](cours/01-introduction-services-web-http.md) | 2h |
| 2 | [Architecture REST et conception d'API](cours/02-architecture-rest-conception-api.md) | 3h |
| 3 | [Formats d'échange : JSON, XML et SOAP](cours/03-formats-echange-json-xml-soap.md) | 3h |
| 4 | [Authentification et autorisation des services web](cours/04-authentification-autorisation-api.md) | 3h |
| 5 | [Documentation et test des API](cours/05-documentation-test-api.md) | 2h |
| 6 | [Architectures orientées services et microservices](cours/06-architectures-orientees-services-microservices.md) | 3h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Découverte du protocole HTTP et d'une API REST](tp/tp1-decouverte-http-api-rest.md) | 2h |
| 2 | [Conception et implémentation d'une API REST](tp/tp2-conception-implementation-api-rest.md) | 2h |
| 3 | [Authentification par JWT d'une API](tp/tp3-authentification-jwt-api.md) | 2h |
| 4 | [Documentation et tests automatisés d'une API](tp/tp4-documentation-tests-api.md) | 2h |

## Évaluation

- TP notés : 40 %
- Mini-projet (extension de l'API développée en TP2-TP4, avec soutenance courte) : 20 %
- Examen final : 40 %

## Environnement technique

Les travaux pratiques utilisent Python (bibliothèque `requests` pour les appels HTTP, `Flask` pour l'implémentation du serveur), `curl` pour l'exploration manuelle du protocole HTTP, ainsi que des outils de documentation et de test d'API (Swagger UI, `pytest`).

## Bibliographie

- IETF, RFC 9110 — *HTTP Semantics* ; RFC 9112 — *HTTP/1.1*.
- IETF, RFC 6749 — *The OAuth 2.0 Authorization Framework* ; RFC 7519 — *JSON Web Token (JWT)*.
- IETF, RFC 9457 — *Problem Details for HTTP APIs*.
- R. Fielding, *Architectural Styles and the Design of Network-based Software Architectures* (thèse de doctorat, 2000).
- OpenAPI Initiative — spécification OpenAPI, [spec.openapis.org](https://spec.openapis.org/).
- L. Richardson, M. Amundsen, *RESTful Web APIs*, O'Reilly.
- S. Newman, *Building Microservices*, O'Reilly.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
