# TP4 — Sécurisation d'une API REST (2h)

## Objectifs
- Auditer et corriger une API REST volontairement vulnérable, en appliquant la checklist du chapitre 6.

## Préparation

L'enseignant fournit le code source d'une petite API REST (ex. Flask/Express) de gestion de commandes, volontairement vulnérable : absence de vérification d'autorisation sur les objets, absence de limitation de débit, réponse trop verbeuse, CORS ouvert à toutes les origines avec identifiants autorisés.

## Exercice 1 — Exploitation d'un BOLA (contrôle d'accès défaillant au niveau des objets)

En s'authentifiant avec un compte utilisateur standard, tenter d'accéder aux commandes d'un autre utilisateur en modifiant simplement l'identifiant dans l'URL de la requête (`GET /api/commandes/{id}`). Documenter le résultat.

## Exercice 2 — Correction du contrôle d'accès

Modifier le code de l'API pour vérifier, à chaque appel, que l'identifiant authentifié correspond bien au propriétaire de la ressource demandée (ou dispose d'un rôle autorisé). Revérifier que l'exploitation de l'exercice 1 échoue désormais.

## Exercice 3 — Exposition excessive de données

Examiner la réponse brute d'un endpoint retournant des informations utilisateur et identifier des champs sensibles renvoyés sans nécessité (ex. hash de mot de passe, informations internes). Corriger l'API pour ne renvoyer que les champs strictement nécessaires.

## Exercice 4 — Limitation de débit

Ajouter un mécanisme de limitation de débit (middleware de rate limiting disponible dans l'écosystème du framework utilisé) sur l'endpoint d'authentification de l'API. Vérifier, par un script simple envoyant des requêtes répétées, que le mécanisme bloque bien au-delà du seuil configuré.

## Exercice 5 — Configuration CORS

Corriger la configuration CORS pour n'autoriser que l'origine légitime de l'application front-end (fournie), au lieu d'accepter toute origine, en particulier lorsque des identifiants (cookies/jetons) sont transmis.

## Exercice 6 — Synthèse

Compléter la checklist du chapitre 6 (8 points) en indiquant, pour chaque point, l'état de l'API avant et après ce TP.

## À rendre

Le code source corrigé de l'API, les preuves de l'exercice 1 (avant correction) et de sa non-reproductibilité après correction, et la checklist complétée.

---

## Conclusion du module

Ce TP4 clôt le module *Sécurité des applications* et, avec lui, l'ensemble du programme : il mobilise des notions vues dans la quasi-totalité des matières précédentes (programmation en C, audit, gouvernance, IA, cryptographie, blockchain, security by design), illustrant la nature transversale de la sécurité informatique.
