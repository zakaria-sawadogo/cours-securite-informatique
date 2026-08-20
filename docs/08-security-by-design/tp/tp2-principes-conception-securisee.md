# TP2 — Application des principes de conception sécurisée (2h)

## Objectif

Analyser un code et une architecture existants au regard des principes du chapitre 3 (moindre privilège, défense en profondeur, échec sécurisé, médiation complète, séparation des tâches, simplicité, valeurs par défaut sécurisées).

## Exercice 1 — Revue de code : échec sécurisé

L'enseignant fournit un extrait de code (pseudo-code ou langage au choix) contenant une fonction d'autorisation avec une gestion d'erreur permissive par défaut (ex. `try { verifier_droits() } catch { autoriser() }`). Identifier le problème, expliquer le scénario d'exploitation, et proposer une correction respectant le principe d'échec sécurisé.

## Exercice 2 — Revue d'architecture : moindre privilège

À partir d'un schéma d'architecture fourni (application web, base de données, service de fichiers, tous accédés avec un compte unique disposant de tous les droits), identifier les violations du principe de moindre privilège et proposer une répartition des comptes/rôles plus fine.

## Exercice 3 — Défense en profondeur

Pour l'application « GesFormation » du TP1, lister au moins 5 couches de contrôle indépendantes qui pourraient être mises en place pour protéger les données de paiement, en identifiant pour chacune la couche qu'elle protège (réseau, application, données) et ce qui se passerait si cette couche seule échouait.

## Exercice 4 — Minimisation de la surface d'attaque

À partir d'une liste de fonctionnalités et de ports/services fournie pour un serveur fictif, identifier ceux qui ne sont manifestement pas nécessaires au fonctionnement de l'application et devraient être désactivés, en justifiant chaque suppression proposée.

## Exercice 5 — Synthèse

Rédiger une fiche de synthèse (1 page) reliant chaque correction proposée dans les exercices précédents au principe de conception sécurisée correspondant.

## À rendre

Les réponses aux exercices 1 à 4 et la fiche de synthèse.
