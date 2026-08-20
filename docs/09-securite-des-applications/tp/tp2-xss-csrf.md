# TP2 — XSS et CSRF sur application vulnérable (2h)

## Cadre du TP

Manipulations exclusivement sur DVWA ou OWASP Juice Shop déployés en local.

## Objectifs
- Exploiter un XSS réfléchi et un XSS stocké.
- Comprendre et démontrer le principe d'une attaque CSRF.
- Mettre en œuvre les contre-mesures correspondantes.

## Exercice 1 — XSS réfléchi

Identifier un paramètre reflété sans encodage dans une page de l'application cible. Construire une URL contenant une charge `<script>alert(document.cookie)</script>` et vérifier son exécution. Documenter l'URL complète et le résultat.

## Exercice 2 — XSS stocké

Identifier un champ persistant (commentaire, profil utilisateur) vulnérable. Injecter une charge XSS stockée et vérifier qu'elle s'exécute pour tout utilisateur consultant le contenu affecté par la suite (recharger la page avec un autre compte si possible).

## Exercice 3 — Contre-mesure XSS

Sur un extrait de code fourni reproduisant la vulnérabilité de l'exercice 1 ou 2, appliquer un encodage contextuel approprié (fonction d'échappement HTML du langage utilisé) et vérifier que la même charge utile est désormais affichée comme texte inerte plutôt qu'exécutée.

## Exercice 4 — CSRF (démonstration guidée)

Sous supervision, à partir d'une fonctionnalité de l'application cible ne disposant pas de jeton anti-CSRF (ex. changement d'e-mail ou de mot de passe), construire une page HTML piégée hors de l'application (formulaire auto-soumis) reproduisant l'exemple du cours. En étant connecté à l'application dans le même navigateur, ouvrir la page piégée et observer l'effet obtenu.

## Exercice 5 — Contre-mesure CSRF et CSP

Documenter comment un jeton anti-CSRF aurait empêché l'exercice 4 de réussir. Ajouter (sur l'extrait de code de l'exercice 3, ou un extrait fourni) un en-tête Content Security Policy restrictif et vérifier qu'une charge XSS externe (chargement d'un script depuis un domaine tiers) est désormais bloquée par le navigateur.

## À rendre

Les preuves de concept (URL, captures, code de la page piégée) des exercices 1, 2 et 4, le code corrigé des exercices 3 et 5, et une explication écrite (10 lignes) de la différence de nature entre XSS et CSRF.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
