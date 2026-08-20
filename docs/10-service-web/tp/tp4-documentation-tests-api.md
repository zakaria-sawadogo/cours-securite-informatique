# TP4 — Documentation et tests automatisés d'une API (2h)

## Objectifs

- Documenter formellement l'API « livres » avec une spécification OpenAPI.
- Écrire une suite de tests automatisés couvrant les principaux scénarios de succès et d'échec de l'API.

## Préparation

Repartir du code source de l'API produite aux TP2 et TP3. Installer `pytest` (`pip install pytest`).

## Exercice 1 — Rédaction de la spécification OpenAPI

Rédiger un fichier `openapi.yaml` décrivant l'ensemble des routes de l'API « livres » : chemins, méthodes, paramètres, schémas de requête et de réponse (y compris le schéma de la ressource « livre » et celui d'une erreur), codes de statut possibles pour chaque opération, et mécanisme d'authentification (JWT via l'en-tête `Authorization`).

```yaml
openapi: 3.0.3
info:
  title: API Livres
  version: "1.0.0"
paths:
  /livres/{id}:
    get:
      summary: Obtenir un livre par son identifiant
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        "200":
          description: Livre trouvé
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Livre"
        "404":
          description: Livre introuvable
components:
  securitySchemes:
    JetonBearer:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    Livre:
      type: object
      required: [id, titre, auteur]
      properties:
        id:
          type: integer
        titre:
          type: string
        auteur:
          type: string
        annee_publication:
          type: integer
        disponible:
          type: boolean
```

## Exercice 2 — Documentation interactive

À l'aide de Swagger UI (déployé localement, ou intégré directement si l'implémentation utilise un framework le générant automatiquement), charger la spécification produite à l'exercice 1 et vérifier, pour au moins trois routes, que l'interface générée correspond bien au comportement réel de l'API.

## Exercice 3 — Tests unitaires et d'intégration

Écrire, avec `pytest`, une suite de tests couvrant au minimum les scénarios suivants pour l'API « livres » :

- Création d'un livre valide (`201`) et vérification du contenu de la réponse.
- Création d'un livre avec des données invalides (`400`).
- Obtention d'un livre existant (`200`) et d'un livre inexistant (`404`).
- Modification d'un livre par un utilisateur autorisé (`200`) et par un utilisateur non authentifié (`401`).
- Suppression d'un livre par un rôle insuffisant (`403`) et par un rôle suffisant (`204` ou `200`).

```python
def test_creation_livre_invalide(client, jeton_bibliothecaire):
    reponse = client.post(
        "/livres",
        json={"titre": ""},
        headers={"Authorization": f"Bearer {jeton_bibliothecaire}"}
    )
    assert reponse.status_code == 400
```

## Exercice 4 — Test de contrat

Écrire un test (ou utiliser un outil de validation de schéma existant) qui valide chaque réponse effectivement renvoyée par l'API contre le schéma déclaré dans la spécification OpenAPI de l'exercice 1, pour au moins deux routes. Documenter tout écart détecté entre l'implémentation réelle et la spécification, puis corriger l'un des deux.

## Exercice 5 — Intégration continue (optionnel)

Configurer l'exécution automatique de la suite de tests de l'exercice 3 à chaque modification du code (par exemple via un script exécuté localement simulant un pipeline d'intégration continue, ou une configuration réelle si l'environnement le permet).

## À rendre

Le fichier `openapi.yaml` complet, la suite de tests `pytest` avec son rapport d'exécution, et le compte rendu des écarts détectés et corrigés à l'exercice 4.

---

## Conclusion du module

Ce TP4 clôt le module *Service web* : il rassemble la conception REST (TP2), l'authentification (TP3) et la documentation/test (TP4) autour d'une même API construite progressivement au fil des séances, et prépare directement l'approfondissement de la sécurisation des API abordé dans le module *Sécurité des applications*.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
