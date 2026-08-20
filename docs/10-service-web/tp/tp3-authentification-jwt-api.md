# TP3 — Authentification par JWT d'une API (2h)

## Objectifs

- Ajouter une authentification par JWT à l'API développée au TP2.
- Distinguer et implémenter une route publique et une route protégée par authentification et autorisation.

## Préparation

Repartir du code source de l'API « livres » produite au TP2. Installer la bibliothèque `PyJWT` (`pip install pyjwt`).

## Exercice 1 — Route de connexion et émission d'un jeton

Ajouter une route `POST /connexion` acceptant un identifiant et un mot de passe (utilisateurs stockés en mémoire pour ce TP, mots de passe hachés — ne jamais stocker de mot de passe en clair, même dans un exercice pédagogique). En cas de succès, renvoyer un JWT signé contenant l'identifiant de l'utilisateur, son rôle, et une expiration courte (par exemple 30 minutes).

```python
import jwt
import datetime
from werkzeug.security import check_password_hash

CLE_SECRETE = "a-remplacer-par-une-valeur-aleatoire-longue"

@app.route("/connexion", methods=["POST"])
def connexion():
    donnees = request.get_json()
    utilisateur = utilisateurs.get(donnees.get("identifiant"))
    if utilisateur is None or not check_password_hash(utilisateur["mot_de_passe_hache"], donnees.get("mot_de_passe", "")):
        return jsonify({"erreur": "Identifiants invalides"}), 401

    jeton = jwt.encode(
        {"sub": utilisateur["id"], "role": utilisateur["role"],
         "exp": datetime.datetime.utcnow() + datetime.timedelta(minutes=30)},
        CLE_SECRETE, algorithm="HS256"
    )
    return jsonify({"jeton": jeton}), 200
```

## Exercice 2 — Middleware de vérification du jeton

Implémenter un décorateur Python vérifiant, sur les routes protégées, la présence et la validité d'un JWT dans l'en-tête `Authorization` (schéma `Bearer`). En l'absence de jeton valide, renvoyer `401 Unauthorized` ; en cas de jeton valide mais de rôle insuffisant pour l'action demandée, renvoyer `403 Forbidden`.

```python
from functools import wraps
from flask import request, jsonify

def authentification_requise(f):
    @wraps(f)
    def decoree(*args, **kwargs):
        en_tete = request.headers.get("Authorization", "")
        if not en_tete.startswith("Bearer "):
            return jsonify({"erreur": "Jeton manquant"}), 401
        jeton = en_tete.removeprefix("Bearer ")
        try:
            charge_utile = jwt.decode(jeton, CLE_SECRETE, algorithms=["HS256"])
        except jwt.ExpiredSignatureError:
            return jsonify({"erreur": "Jeton expiré"}), 401
        except jwt.InvalidTokenError:
            return jsonify({"erreur": "Jeton invalide"}), 401
        request.utilisateur_courant = charge_utile
        return f(*args, **kwargs)
    return decoree
```

## Exercice 3 — Protection des routes sensibles

Appliquer le décorateur de l'exercice 2 aux routes `POST`, `PUT`, `PATCH` et `DELETE` de l'API « livres », en laissant les routes `GET` accessibles sans authentification. Ajouter une vérification de rôle : seul un utilisateur de rôle `administrateur` peut supprimer un livre (`DELETE`), un utilisateur de rôle `bibliothecaire` pouvant créer et modifier.

## Exercice 4 — Vérification du comportement

Vérifier, avec `curl` ou Postman, les cas suivants et documenter le code de statut obtenu pour chacun :

- Accès à une route protégée sans jeton.
- Accès avec un jeton expiré (réduire temporairement la durée d'expiration pour le test).
- Accès avec un jeton valide mais un rôle insuffisant pour l'action demandée.
- Accès avec un jeton valide et un rôle suffisant.

## Exercice 5 — Analyse critique

Décoder manuellement (sans vérification de signature, par exemple avec un outil en ligne de décodage JWT ou un script Python isolé) un jeton généré à l'exercice 1, et vérifier que sa charge utile est bien lisible sans connaître la clé secrète. Rédiger un court paragraphe expliquant pourquoi aucune donnée sensible ne doit figurer dans la charge utile d'un JWT signé mais non chiffré.

## À rendre

Le code source complet de l'API mise à jour, le journal des vérifications de l'exercice 4, et le paragraphe d'analyse de l'exercice 5.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
