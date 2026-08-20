# TP4 — Intégration de contrôles de sécurité en CI/CD (DevSecOps) (2h)

## Objectifs
- Mettre en place un pipeline CI/CD simple intégrant des contrôles de sécurité automatisés.

## Préparation

Un dépôt Git simple (fourni par l'enseignant) contenant une petite application (ex. script Python ou application web basique) avec, volontairement, une dépendance vulnérable connue et un secret codé en dur dans un fichier de configuration.

## Exercice 1 — Détection de secrets

Installer et exécuter un outil de détection de secrets (ex. `gitleaks` ou `truffleHog`) sur le dépôt fourni. Identifier le secret exposé, l'extraire du code, le remplacer par une variable d'environnement, et vérifier que l'outil ne le détecte plus.

## Exercice 2 — Analyse de composition logicielle (SCA)

Exécuter un outil d'analyse de dépendances (ex. `pip-audit` pour Python, `npm audit` pour Node.js, ou OWASP Dependency-Check) sur le projet. Identifier la ou les vulnérabilités connues (CVE) dans les dépendances utilisées, mettre à jour la dépendance concernée, et vérifier que l'alerte disparaît.

## Exercice 3 — Analyse statique (SAST)

Exécuter un outil d'analyse statique adapté au langage du projet (ex. `bandit` pour Python, `semgrep` de façon générique). Examiner les alertes remontées, corriger au moins un problème identifié, et documenter pourquoi il constituait un risque.

## Exercice 4 — Intégration dans un pipeline CI

Créer un fichier de configuration de pipeline (GitHub Actions, GitLab CI ou équivalent disponible) exécutant automatiquement, à chaque `push`, les trois contrôles précédents (secrets, SCA, SAST), et faisant échouer le pipeline (`exit code` non nul) si une vulnérabilité critique est détectée.

## Exercice 5 — Synthèse

Rédiger une fiche de synthèse (15 lignes) reliant ce TP au chapitre 5 du cours : à quelle étape du Secure SDLC correspond chacun des contrôles mis en œuvre, et en quoi cette automatisation illustre concrètement le principe de *shift left*.

## À rendre

Le fichier de configuration du pipeline CI, une capture d'écran d'une exécution réussie du pipeline (après corrections), et la fiche de synthèse.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
