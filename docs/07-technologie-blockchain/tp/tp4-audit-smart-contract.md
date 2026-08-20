# TP4 — Audit de vulnérabilités d'un smart contract (2h)

## Cadre du TP

Manipulations exclusivement sur un environnement de test local (Remix VM), reproduisant des vulnérabilités connues à des fins strictement pédagogiques.

## Objectifs
- Identifier et exploiter, de façon contrôlée, une vulnérabilité de réentrance.
- Corriger le contrat et vérifier que l'exploitation échoue après correction.

## Exercice 1 — Déploiement d'un contrat vulnérable

L'enseignant fournit un contrat `CoffreVulnerable.sol` reproduisant le schéma de vulnérabilité de réentrance vu en cours (fonction `retirer()` effectuant l'appel externe avant la mise à jour de l'état). Le déployer sur Remix VM et alimenter le contrat avec des fonds de test depuis plusieurs comptes.

## Exercice 2 — Contrat attaquant

Écrire un second contrat `Attaquant.sol` dont la fonction de réception (`receive()` ou `fallback()`) rappelle `retirer()` sur le contrat cible tant que celui-ci contient des fonds. Déployer ce contrat et déclencher l'attaque : observer que le contrat attaquant parvient à retirer plus que son solde initial déposé.

## Exercice 3 — Analyse et documentation du constat

Rédiger un constat au format vu dans le module *Audit organisation et technique* : titre, description technique de la vulnérabilité, preuve de concept (trace des appels), impact (montant potentiellement siphonné), recommandation (patron checks-effects-interactions, ou utilisation d'un verrou de réentrance de type `nonReentrant`).

## Exercice 4 — Correction et re-test

Corriger `CoffreVulnerable.sol` en appliquant le patron checks-effects-interactions (mise à jour du solde avant l'appel externe). Redéployer et relancer l'attaque de l'exercice 2 : vérifier qu'elle échoue désormais.

## Exercice 5 — Analyse statique automatisée (si outil disponible)

Si l'environnement le permet, exécuter un outil d'analyse statique (ex. Slither) sur les deux versions du contrat (vulnérable et corrigée) et comparer les alertes remontées.

## À rendre

Les deux contrats (`.sol`), le contrat attaquant, le constat rédigé (exercice 3), et une capture montrant l'échec de l'attaque après correction.
