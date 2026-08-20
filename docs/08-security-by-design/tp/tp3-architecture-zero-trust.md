# TP3 — Conception d'une architecture zero trust (2h)

## Objectif

Concevoir, pour l'application « GesFormation » (introduite au TP1), une évolution de son architecture réseau/accès vers un modèle zero trust, en partant d'une architecture périmétrique classique fournie par l'enseignant.

## Exercice 1 — Analyse de l'architecture existante

À partir du schéma fourni (réseau interne « de confiance », VPN d'accès distant, pare-feu périmétrique unique), identifier les hypothèses de confiance implicite qui posent problème au regard des principes zero trust du chapitre 4.

## Exercice 2 — Identification des actifs et flux à protéger

Lister les actifs sensibles de « GesFormation » (base de données utilisateurs, documents justificatifs, service de paiement) et les flux d'accès associés (qui accède à quoi, depuis où, avec quel niveau de sensibilité).

## Exercice 3 — Conception de la politique d'accès contextuelle

Pour 3 scénarios d'accès distincts (ex. un administrateur depuis le réseau interne, un formateur en télétravail depuis son domicile, un prestataire tiers de maintenance), définir une politique d'accès zero trust incluant : méthode d'authentification requise (précisant le niveau de MFA), critères contextuels pris en compte (appareil, localisation), et niveau d'accès accordé.

## Exercice 4 — Micro-segmentation

Proposer un schéma de micro-segmentation pour les composants internes de « GesFormation » (serveur applicatif, base de données, service de stockage de documents, service de notification), en précisant les flux strictement nécessaires entre chaque composant et ceux qui doivent être explicitement bloqués.

## Exercice 5 — Plan de migration

Rédiger un plan de migration en 3 étapes priorisées (partant de l'actif le plus critique) pour faire évoluer l'architecture actuelle vers le modèle conçu, en tenant compte du principe de migration progressive vu en cours.

## À rendre

Le schéma d'architecture zero trust cible, les politiques d'accès des 3 scénarios, le schéma de micro-segmentation, et le plan de migration.
