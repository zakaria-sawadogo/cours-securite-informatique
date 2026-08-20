# TP1 — Découverte du protocole HTTP et d'une API REST (2h)

## Objectifs

- Manipuler directement le protocole HTTP en ligne de commande, sans passer par un navigateur.
- Explorer une API REST publique ou pédagogique existante et en documenter le comportement observé.

## Préparation

Ce TP s'appuie sur une API REST publique gratuite et sans données sensibles (au choix de l'enseignant ou de l'étudiant, par exemple une API de démonstration listant des ressources factices), ou sur une API pédagogique fournie en local. Prévoir `curl` et un client HTTP graphique (Postman ou équivalent) installés au préalable.

## Exercice 1 — Requête GET et analyse de la réponse

À l'aide de `curl -v` (mode verbeux, affichant la requête et la réponse complètes), effectuer une requête `GET` vers un point de terminaison de l'API choisie. Relever et commenter :

- la ligne de requête complète envoyée par `curl` ;
- le code de statut de la réponse et sa signification exacte ;
- au moins quatre en-têtes de réponse différents et leur rôle ;
- le format et le contenu du corps de réponse.

## Exercice 2 — Méthodes et codes de statut

En vous appuyant sur la documentation de l'API choisie, identifier une ressource permettant de tester au moins trois méthodes HTTP différentes (`GET`, `POST`, éventuellement `PUT`/`PATCH`/`DELETE` si l'API le permet sans conséquence destructrice). Pour chaque méthode testée, noter dans un tableau : la requête envoyée, le code de statut obtenu, et une explication de ce code au regard du tableau des classes de statut vu en cours.

## Exercice 3 — En-têtes personnalisés et négociation de contenu

Envoyer la même requête en faisant varier l'en-tête `Accept` (par exemple `application/json` puis `application/xml` si l'API le supporte), et observer si la réponse change de format. Envoyer également une requête avec un en-tête `Accept` demandant un format non supporté par l'API et documenter le code de statut retourné.

## Exercice 4 — Reproduction avec un client graphique

Reproduire l'une des requêtes de l'exercice 2 avec Postman (ou équivalent), en construisant la requête via l'interface graphique. Comparer le résultat obtenu à celui de `curl` et exporter la requête sous forme de collection réutilisable.

## Exercice 5 — Script Python

Écrire un court script Python, à l'aide de la bibliothèque `requests`, qui effectue automatiquement les mêmes trois requêtes que l'exercice 2 et affiche, pour chacune, le code de statut et un extrait du corps de réponse.

```python
import requests

reponse = requests.get("https://api.exemple.edu/ressources")
print(reponse.status_code)
print(reponse.json())
```

## À rendre

Le tableau de l'exercice 2, les captures ou journaux `curl -v` des exercices 1 et 3, la collection Postman exportée de l'exercice 4, et le script Python de l'exercice 5.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
