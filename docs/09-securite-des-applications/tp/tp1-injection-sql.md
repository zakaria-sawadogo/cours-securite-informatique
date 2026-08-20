# TP1 — Injection SQL et contre-mesures (2h)

## Cadre du TP

Manipulations exclusivement sur DVWA ou OWASP Juice Shop déployés en local à des fins pédagogiques.

## Objectifs
- Exploiter une injection SQL simple et une injection UNION-based.
- Implémenter et vérifier une contre-mesure par requêtes préparées.

## Exercice 1 — Contournement d'authentification

Sur le formulaire de connexion de l'application cible, tester une charge utile de type `' OR '1'='1' -- ` dans le champ identifiant. Documenter le résultat obtenu et expliquer précisément, requête par requête, pourquoi cela fonctionne.

## Exercice 2 — Injection UNION-based

Sur un champ de recherche ou de filtre vulnérable de l'application cible, déterminer le nombre de colonnes de la requête sous-jacente (par tâtonnement avec `ORDER BY` ou `UNION SELECT NULL, NULL, ...`), puis extraire des données d'une autre table (ex. table des utilisateurs et leurs mots de passe hachés) via une injection `UNION SELECT`.

## Exercice 3 — Injection en aveugle (démonstration guidée)

Sous supervision, sur un point d'injection ne renvoyant aucune donnée directement visible, utiliser une charge utile booléenne (`AND 1=1` vs `AND 1=2`) pour observer une différence de comportement de l'application, illustrant le principe de l'extraction de données bit par bit sans affichage direct des résultats.

## Exercice 4 — Correction

À partir d'un extrait de code source fourni (vulnérable, langage au choix parmi ceux vus en cours ou en TP), réécrire la requête concernée en utilisant des requêtes préparées/paramétrées. Vérifier que la même charge utile d'attaque, rejouée contre le code corrigé, échoue désormais.

## Exercice 5 — Constat d'audit

Rédiger un constat au format vu en *Audit organisation et technique* (observation, criticité CVSS estimée, preuve, impact, recommandation) pour la vulnérabilité exploitée à l'exercice 1 ou 2.

## À rendre

Les preuves de concept des exercices 1 à 3 (captures/logs), le code corrigé de l'exercice 4, et le constat d'audit de l'exercice 5.
