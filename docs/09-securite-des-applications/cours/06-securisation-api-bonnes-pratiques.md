# Chapitre 6 — Sécurisation des API et bonnes pratiques

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=09-securite-des-applications/slides/06-securisation-api-bonnes-pratiques.txt){ target=_blank }

## 1. Spécificités de la sécurité des API

Les API (notamment REST et GraphQL) sont devenues le principal mode d'intégration entre applications, exposant souvent directement une logique métier et des données sensibles, avec une surface d'attaque distincte des applications web traditionnelles (pas d'interface graphique intermédiaire filtrant implicitement certains usages).

Cette absence d'interface graphique intermédiaire mérite d'être bien comprise : dans une application web classique, un utilisateur navigue à travers des écrans qui, implicitement, ne proposent que certaines actions dans certains contextes (un bouton « supprimer » n'apparaît que si l'utilisateur a le droit de l'utiliser, par exemple). Une API, elle, est appelée directement par un programme client (application mobile, service tiers, script), qui peut en théorie construire n'importe quelle requête vers n'importe quel point d'entrée documenté ou deviné, sans passer par les garde-fous visuels d'une interface. Il en résulte que **toute** vérification de sécurité doit être portée par le serveur API lui-même, sans jamais supposer qu'un client « bien élevé » respectera un usage attendu.

## 2. OWASP API Security Top 10 (panorama)

Référentiel dédié, distinct du Top 10 applicatif général, structurant les risques spécifiques aux API :

- **Contrôle d'accès défaillant au niveau des objets (BOLA)** : un utilisateur accède aux données d'un autre utilisateur en modifiant simplement un identifiant dans la requête (ex. `GET /api/commandes/1234` accessible en changeant l'identifiant, sans vérification que la commande appartient bien à l'utilisateur authentifié) — variante API de l'IDOR déjà vu.
- **Authentification défaillante** : rappel du chapitre 3, appliqué spécifiquement aux jetons d'API.
- **Exposition excessive de données (excessive data exposure)** : l'API renvoie plus de champs que nécessaire, en s'appuyant sur le client pour filtrer l'affichage — un attaquant inspectant directement la réponse brute accède alors à des données qui n'auraient jamais dû être transmises.
- **Absence de limitation de ressources et de débit (rate limiting)** : permet des attaques par force brute, de déni de service, ou une extraction massive de données (scraping).
- **Contrôle d'accès défaillant au niveau des fonctions (BFLA)** : un utilisateur standard parvient à appeler un endpoint réservé aux administrateurs faute de vérification côté serveur (une restriction uniquement appliquée côté interface utilisateur n'est jamais suffisante).
- **Mauvaise configuration de sécurité** : en-têtes de sécurité manquants, messages d'erreur trop verbeux révélant des détails d'implémentation, CORS mal configuré (autorisant des origines non nécessaires).

### Illustration : BOLA (Broken Object Level Authorization)

```
# Requête légitime de l'utilisateur authentifié 42, consultant sa propre commande
GET /api/commandes/1001
Authorization: Bearer <jeton-utilisateur-42>

# Requête malveillante : le même utilisateur modifie simplement
# l'identifiant dans l'URL pour tenter d'accéder à une commande
# appartenant à un autre utilisateur
GET /api/commandes/1002
Authorization: Bearer <jeton-utilisateur-42>
```

```python
# Vulnérable : la commande est retournée dès qu'elle existe,
# sans vérifier qu'elle appartient à l'utilisateur authentifié
def obtenir_commande(id_commande, utilisateur_courant):
    commande = base_donnees.commandes.get(id_commande)
    return commande

# Corrigé : vérification explicite de la propriété de la ressource,
# systématique à chaque appel, indépendamment de ce que l'interface
# graphique aurait normalement permis ou non de faire
def obtenir_commande(id_commande, utilisateur_courant):
    commande = base_donnees.commandes.get(id_commande)
    if commande is None or commande.id_utilisateur != utilisateur_courant.id:
        raise AccesRefuse()
    return commande
```

Cette vérification doit être répétée pour **chaque** point d'entrée manipulant une ressource identifiée par un identifiant (commande, facture, document, message), et non supposée acquise une fois pour toutes ailleurs dans l'application.

### Approfondissement : l'assignation massive (*mass assignment*)

Une vulnérabilité fréquente et distincte des précédentes survient lorsqu'un point d'entrée d'API met à jour un objet en acceptant directement l'ensemble des champs fournis par le client, sans restreindre explicitement quels champs peuvent réellement être modifiés par cet appelant :

```python
# Vulnérable : tous les champs envoyés par le client sont appliqués
# directement à l'objet utilisateur, y compris des champs qui ne
# devraient jamais être modifiables par l'utilisateur lui-même
def mettre_a_jour_profil(id_utilisateur, donnees_recues):
    utilisateur = base_donnees.utilisateurs.get(id_utilisateur)
    utilisateur.update(donnees_recues)   # ex. { "nom": "...", "est_administrateur": true }
    utilisateur.sauvegarder()

# Corrigé : liste blanche explicite des champs autorisés à être
# modifiés par ce point d'entrée précis
CHAMPS_MODIFIABLES_PAR_UTILISATEUR = {"nom", "email", "biographie"}

def mettre_a_jour_profil(id_utilisateur, donnees_recues):
    utilisateur = base_donnees.utilisateurs.get(id_utilisateur)
    for champ, valeur in donnees_recues.items():
        if champ in CHAMPS_MODIFIABLES_PAR_UTILISATEUR:
            setattr(utilisateur, champ, valeur)
    utilisateur.sauvegarder()
```

Si un attaquant ajoute simplement le champ `est_administrateur: true` à sa requête de mise à jour de profil, la version vulnérable applique ce champ sans discernement, provoquant une élévation de privilèges. La liste blanche explicite des champs modifiables constitue ici, comme au chapitre 1, la défense structurellement fiable plutôt qu'une tentative de filtrage d'une liste noire de champs sensibles à exclure.

## 3. Authentification et autorisation des API

- **Clés d'API** : simples à mettre en œuvre mais offrant peu de granularité ; à transmettre exclusivement via un en-tête HTTP (jamais dans l'URL, qui peut être journalisée ou mise en cache) et sur une connexion chiffrée.
- **OAuth 2.0** : cadre standard de délégation d'autorisation, permettant à une application tierce d'accéder à des ressources au nom d'un utilisateur sans jamais connaître son mot de passe.
- **JWT** : rappel du chapitre 3, avec vérification stricte de la signature et de l'expiration.
- **Principe du moindre privilège appliqué aux portées (scopes)** : une clé/jeton d'API ne devrait donner accès qu'aux opérations et ressources strictement nécessaires à l'usage prévu, pas à l'ensemble de l'API par défaut.

## 4. Validation systématique côté serveur

Principe absolu : **toute validation effectuée uniquement côté client (JavaScript) doit être considérée comme inexistante du point de vue de la sécurité**, car un attaquant peut appeler directement l'API en contournant totalement l'interface utilisateur. Toute règle de sécurité (autorisation, validation de format, limites métier) doit être appliquée et vérifiée côté serveur, la validation côté client n'étant qu'un confort d'expérience utilisateur, jamais un contrôle de sécurité.

Ce principe s'étend aux **limites métier**, souvent oubliées : par exemple, une API de commande en ligne doit revérifier côté serveur qu'une quantité commandée reste positive et raisonnable, qu'un prix n'a pas été altéré entre l'affichage côté client et la soumission de la commande, ou qu'une remise appliquée correspond bien à une règle métier valide — autant de contrôles qu'un client malveillant peut contourner en construisant directement sa requête API sans jamais passer par l'interface prévue.

## 5. Configuration CORS (Cross-Origin Resource Sharing)

Le mécanisme CORS définit quelles origines web sont autorisées à effectuer des requêtes cross-origin vers une API depuis un navigateur. Une configuration `Access-Control-Allow-Origin: *` combinée à l'autorisation d'envoi de cookies/identifiants est une mauvaise pratique fréquente, pouvant faciliter des attaques exploitant le navigateur d'un utilisateur authentifié depuis un site tiers malveillant.

```
# Configuration risquée : autorise n'importe quelle origine ET
# les identifiants (cookies), une combinaison normalement rejetée
# par les navigateurs conformes à la spécification mais parfois
# reconstituée par erreur via une réflexion dynamique de l'origine
Access-Control-Allow-Origin: https://site-quelconque-demande.example
Access-Control-Allow-Credentials: true

# Configuration recommandée : liste explicite et restreinte des
# origines réellement autorisées à effectuer des appels authentifiés
Access-Control-Allow-Origin: https://application-officielle.example
Access-Control-Allow-Credentials: true
```

Un piège fréquent consiste à réfléchir dynamiquement l'en-tête `Origin` de la requête entrante dans la réponse `Access-Control-Allow-Origin` (pour éviter de maintenir une liste statique), ce qui revient en pratique à accepter n'importe quelle origine tout en semblant restrictif : cette pratique doit être évitée au profit d'une liste explicite et contrôlée des origines légitimes.

## 6. Limitation de débit et quotas (rate limiting)

Mesure de défense en profondeur essentielle, souvent négligée : limiter le nombre de requêtes qu'un client (identifié par clé d'API, jeton ou adresse IP) peut effectuer dans une fenêtre de temps donnée, contrant à la fois les attaques par force brute sur l'authentification et l'extraction massive de données.

### Où et comment appliquer cette limitation

La limitation de débit doit généralement être appliquée à plusieurs niveaux complémentaires : globalement par adresse IP (contre les attaques automatisées non authentifiées), par compte ou clé d'API (contre l'abus d'un compte légitime compromis ou détourné), et parfois par point d'entrée spécifique (un point d'entrée d'authentification ou de réinitialisation de mot de passe justifie une limite bien plus stricte qu'un point d'entrée de simple consultation). Une réponse HTTP standardisée (code `429 Too Many Requests`, en-tête indiquant le délai avant nouvelle tentative) permet aux clients légitimes de s'adapter, tandis qu'un dépassement répété peut alimenter une détection d'abus plus large.

### Approfondissement : risques spécifiques à GraphQL

Les API GraphQL, qui laissent le client composer librement la forme exacte de sa requête (quels champs, quelle profondeur d'imbrication de relations), introduisent des risques spécifiques qui n'ont pas d'équivalent direct dans une API REST classique :

- **Requêtes profondément imbriquées ou récursives** : un client peut construire une requête demandant des relations imbriquées sur plusieurs niveaux (ex. un utilisateur, ses commandes, les produits de chaque commande, les avis sur chaque produit, etc.), générant côté serveur un coût de calcul et de base de données disproportionné par rapport à la taille apparente de la requête — un vecteur de déni de service applicatif propre à GraphQL, à limiter par une profondeur ou une complexité de requête maximale imposée côté serveur.
- **Introspection en production** : GraphQL propose nativement un mécanisme d'introspection permettant à un client de découvrir l'intégralité du schéma de l'API (tous les types, champs et relations disponibles). Laisser cette fonctionnalité active en environnement de production facilite la reconnaissance par un attaquant, qui découvre ainsi la surface d'attaque complète sans travail de rétro-ingénierie ; il est recommandé de la désactiver hors des environnements de développement.

## 7. Journalisation et supervision des API

Journaliser les tentatives d'accès refusées, les erreurs d'authentification répétées, et les schémas d'usage anormaux (rappel du module *Gouvernance*, chapitre 4, sur les indicateurs de sécurité), pour permettre la détection d'une exploitation en cours et alimenter une réponse à incident.

Un point de vigilance propre au contexte API : les journaux ne doivent jamais contenir en clair les jetons d'authentification, clés d'API ou autres identifiants sensibles transmis dans les en-têtes de requête, sous peine de transformer le système de journalisation lui-même en cible attractive pour un attaquant ayant compromis un accès en lecture à ces journaux.

## 8. Synthèse : une checklist de sécurisation d'API

1. Authentification forte et jetons correctement vérifiés (chapitre 3).
2. Autorisation vérifiée systématiquement à chaque appel, au niveau objet et fonction (jamais déléguée au client), y compris pour les champs modifiables (prévention de l'assignation massive).
3. Validation stricte des entrées et des limites métier côté serveur, quelle que soit la validation déjà présente côté client.
4. Exposition minimale des données (ne renvoyer que les champs nécessaires).
5. Limitation de débit et quotas, à plusieurs niveaux (IP, compte, point d'entrée sensible).
6. Configuration CORS restrictive et explicitement listée, sans réflexion dynamique non contrôlée de l'origine.
7. Chiffrement systématique des communications (TLS, module *Cryptographie*).
8. Journalisation et supervision des accès et anomalies, sans exposer d'identifiants sensibles dans les journaux eux-mêmes.
9. Pour les API GraphQL spécifiquement : limitation de la profondeur/complexité des requêtes et désactivation de l'introspection en production.

## À retenir

- L'OWASP API Security Top 10 recense des risques spécifiques aux API (BOLA, BFLA, assignation massive, exposition excessive de données, absence de rate limiting), complémentaires du Top 10 applicatif général.
- Aucune validation ni autorisation effectuée uniquement côté client ne doit être considérée comme un contrôle de sécurité : tout doit être revérifié côté serveur, y compris les limites métier et les champs réellement modifiables par un appelant donné.
- Une configuration CORS restrictive et explicite évite qu'un site tiers malveillant n'exploite le navigateur d'un utilisateur authentifié ; la réflexion dynamique non contrôlée de l'origine annule l'intérêt de la restriction.
- GraphQL introduit des risques spécifiques (requêtes imbriquées coûteuses, introspection exposée) qui nécessitent des contre-mesures dédiées, absentes d'une API REST classique.
- La sécurisation d'une API repose sur la combinaison systématique de plusieurs contrôles (authentification, autorisation, validation, limitation de débit, configuration, journalisation), en cohérence directe avec la défense en profondeur vue au module *Security by design*.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
