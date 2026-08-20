# Chapitre 1 — Introduction : principes de security by design et privacy by design

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/01-introduction-security-by-design.txt){ target=_blank }

## 1. Définition

*Security by design* désigne l'approche consistant à intégrer les exigences de sécurité **dès la phase de conception** d'un système, plutôt que de les ajouter a posteriori sous forme de correctifs après un incident ou un audit. C'est un changement de posture : la sécurité devient une contrainte de conception au même titre que la performance ou l'ergonomie, pas une couche additionnelle optionnelle.

## 2. Pourquoi corriger après coup coûte plus cher

Le coût de correction d'une faille de conception augmente très fortement à mesure que le projet avance : une faille identifiée en phase de conception coûte typiquement un ordre de grandeur de moins à corriger qu'une faille découverte en production, en particulier si elle nécessite de repenser une architecture déjà déployée et adoptée par des utilisateurs. Cette observation motive l'intégration précoce de la sécurité (« shift left »), reprise au chapitre 5 (DevSecOps).

## 3. Security by design vs sécurité périmétrique

Un modèle de sécurité historique reposait sur une **frontière protégée** (pare-feu, périmètre réseau) protégeant un intérieur considéré comme de confiance. Ce modèle s'avère fragile dès qu'un attaquant franchit le périmètre (par hameçonnage, compromission d'un poste, prestataire tiers compromis) : à l'intérieur, peu de contrôles supplémentaires ralentissent sa progression. *Security by design* pousse à concevoir des systèmes résilients même en présence d'une compromission partielle, préfigurant le modèle zero trust (chapitre 4).

## 4. Les grands principes, aperçu

Ce module détaille au chapitre 3 une liste de principes de conception sécurisée reconnus (moindre privilège, défense en profondeur, échec sécurisé, séparation des tâches, simplicité de conception, médiation complète), formalisés notamment par Saltzer et Schroeder dès 1975 et toujours d'actualité.

## 5. Privacy by design

Concept complémentaire, formalisé par Ann Cavoukian (fin des années 1990-2000) et aujourd'hui explicitement exigé par plusieurs réglementations de protection des données (ex. article 25 du RGPD, « protection des données dès la conception et par défaut ») : la protection des données personnelles doit être une caractéristique par défaut d'un système, pas une option activable. Ce concept est développé au chapitre 6.

## 6. Sécurité et fonctionnalité : un arbitrage, pas une opposition

Une idée reçue oppose sécurité et facilité d'usage. Une conception mature cherche plutôt à rendre le **comportement sécurisé le comportement par défaut et le plus simple** pour l'utilisateur (ex. authentification à double facteur activée par défaut plutôt que reléguée à un réglage avancé rarement découvert). Une mesure de sécurité qui dégrade excessivement l'expérience utilisateur est souvent contournée en pratique (mots de passe notés sur un post-it, désactivation d'un contrôle jugé trop contraignant) — un échec de conception, pas seulement un problème de discipline des utilisateurs.

## 7. Articulation avec le reste du programme

Ce module synthétise et opérationnalise des notions vues ailleurs dans le programme : vulnérabilités identifiées en *Audit organisation et technique*, gestion des risques du module *Gouvernance*, primitives cryptographiques à intégrer correctement (*Cryptographie*). *Security by design* est la discipline qui transforme ces connaissances en pratiques de conception concrètes, en amont du développement.

## À retenir

- *Security by design* intègre la sécurité dès la conception, réduisant fortement le coût de correction par rapport à une approche corrective a posteriori.
- Le modèle de sécurité périmétrique seul est insuffisant face à des attaquants capables de franchir la frontière ; une conception résiliente à la compromission partielle est nécessaire.
- Une mesure de sécurité qui dégrade excessivement l'usage est souvent contournée : la sécurité par défaut et simple d'usage est un objectif de conception, pas un vœu pieux.
