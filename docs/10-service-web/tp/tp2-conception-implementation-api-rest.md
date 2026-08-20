# TP2 — Conception et implémentation d'une API REST (2h)

## Objectifs

- Concevoir les routes d'une API REST respectant les bonnes pratiques vues en cours.
- Implémenter une API REST simple avec Flask, réalisant les opérations CRUD sur une ressource.

## Préparation

Environnement Python avec Flask installé (`pip install flask`). L'API développée dans ce TP sera réutilisée et enrichie dans les TP3 et TP4 : conserver soigneusement le code produit.

## Exercice 1 — Conception des routes

Pour une ressource « livre » (attributs : identifiant, titre, auteur, année de publication, disponible), concevoir sur papier ou dans un fichier texte le tableau des routes de l'API en respectant les conventions vues en cours (chapitre 2) : nom de ressource au pluriel, verbes HTTP appropriés, codes de statut attendus pour chaque cas de succès et d'échec. Faire valider ce tableau avant de passer à l'implémentation.

## Exercice 2 — Implémentation du CRUD

Implémenter l'API conçue à l'exercice 1 avec Flask, en stockant les données en mémoire (structure de données Python, sans base de données pour ce TP). L'API doit exposer :

- `GET /livres` : lister tous les livres, avec pagination simple (`page`, `taille_page`).
- `GET /livres/<id>` : obtenir un livre précis (`404` si absent).
- `POST /livres` : créer un livre (`201` avec l'en-tête `Location` pointant vers la ressource créée).
- `PUT /livres/<id>` : remplacer intégralement un livre existant.
- `PATCH /livres/<id>` : modifier partiellement un livre (ex. changer sa disponibilité).
- `DELETE /livres/<id>` : supprimer un livre.

```python
from flask import Flask, jsonify, request, url_for

app = Flask(__name__)
livres = {}
prochain_id = 1

@app.route("/livres", methods=["POST"])
def creer_livre():
    global prochain_id
    donnees = request.get_json()
    livre = {"id": prochain_id, **donnees}
    livres[prochain_id] = livre
    prochain_id += 1
    return jsonify(livre), 201, {"Location": url_for("obtenir_livre", id_livre=livre["id"])}

@app.route("/livres/<int:id_livre>", methods=["GET"])
def obtenir_livre(id_livre):
    livre = livres.get(id_livre)
    if livre is None:
        return jsonify({"erreur": "Livre introuvable"}), 404
    return jsonify(livre), 200
```

## Exercice 3 — Validation des entrées

Ajouter une validation minimale des données reçues en `POST` et `PUT` (champs obligatoires présents, type correct pour l'année de publication) ; renvoyer un code `400 Bad Request` avec un message d'erreur explicite en cas de donnée invalide, en cohérence avec le principe de méfiance envers les entrées vu dans le module *Sécurité des applications*.

## Exercice 4 — Vérification avec `curl` et Postman

Vérifier manuellement, avec `curl` ou Postman, que chaque route se comporte conformément au tableau de conception de l'exercice 1, y compris les cas d'erreur (identifiant inexistant, donnée invalide).

## Exercice 5 — Pagination et filtrage

Ajouter à la route `GET /livres` un paramètre de requête `disponible` permettant de filtrer uniquement les livres disponibles ou indisponibles, en plus de la pagination déjà en place.

## À rendre

Le code source complet de l'API, le tableau de conception de l'exercice 1, et une capture ou un journal des vérifications de l'exercice 4 pour chaque route.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
