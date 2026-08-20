# Chapitre 1 — Introduction à l'audit de sécurité

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/01-introduction-audit-securite.txt){ target=_blank }

## 1. Définition

Un audit de sécurité est une évaluation systématique et indépendante du niveau de sécurité d'un système d'information, visant à vérifier sa conformité à un référentiel (normatif, réglementaire ou contractuel) et à identifier les écarts entre l'état constaté et l'état attendu.

## 2. Pourquoi auditer ?

- **Conformité réglementaire** : obligations légales (protection des données, secteurs régulés — banque, santé).
- **Exigence contractuelle** : un client ou partenaire exige un niveau de sécurité prouvé (ex. certification ISO 27001, conformité PCI-DSS pour le paiement en ligne).
- **Gestion des risques** : identifier les vulnérabilités avant qu'un attaquant ne les exploite.
- **Amélioration continue** : mesurer les progrès entre deux audits successifs.

## 3. Les grandes familles d'audit

| Type d'audit | Objet | Exemple de livrable |
|---|---|---|
| **Audit organisationnel** | politiques, procédures, gouvernance | rapport de conformité ISO 27001 |
| **Audit technique** | configuration des systèmes, réseaux, applications | rapport de test d'intrusion |
| **Audit de conformité** | respect d'une réglementation précise | rapport PCI-DSS, RGPD |
| **Audit de code** | revue du code source | rapport SAST (analyse statique) |
| **Audit physique** | sécurité des locaux, contrôle d'accès | rapport d'inspection physique |

Ce module couvre principalement les deux premières familles, qui se complètent : un système peut être « techniquement » bien configuré mais dépourvu de procédures (pas de gestion des correctifs formalisée), ou disposer de belles procédures jamais appliquées en pratique.

## 4. Les acteurs et postures

- **Auditeur interne** : appartient à l'organisation auditée, connaît le contexte, indépendance relative.
- **Auditeur externe** : société tierce, plus grande indépendance, souvent exigée par les référentiels de certification.
- **Boîte noire (black box)** : l'auditeur ne dispose d'aucune information préalable, simule un attaquant externe.
- **Boîte grise (grey box)** : informations partielles (comptes utilisateurs, documentation), simule une menace interne ou un attaquant ayant déjà un premier accès.
- **Boîte blanche (white box)** : accès complet (code source, architecture, comptes admin), maximise la couverture de l'audit.

## 5. Cadre légal et éthique

Tout audit technique — en particulier un test d'intrusion — implique des actions qui, sans autorisation, seraient qualifiées d'accès et de maintien frauduleux dans un système de traitement automatisé de données. Un audit ne peut être mené que dans le cadre d'un **mandat écrit** précisant :

- le périmètre exact autorisé (adresses IP, applications, plages horaires) ;
- les techniques autorisées et interdites (ex. déni de service exclu par défaut) ;
- les responsables à contacter en cas d'incident pendant l'audit ;
- les modalités de confidentialité et de destruction des données collectées.

## 6. Le cycle de vie d'un audit

1. **Cadrage** : définition du périmètre, des objectifs, du mandat.
2. **Collecte d'information / reconnaissance**.
3. **Analyse** (organisationnelle et/ou technique).
4. **Identification et qualification des écarts/vulnérabilités**.
5. **Rédaction du rapport** (constats, criticité, recommandations).
6. **Restitution** et plan d'action.
7. **Suivi** (contre-audit / vérification de la remédiation).

## À retenir

- Un audit combine une dimension organisationnelle et une dimension technique.
- Le mandat écrit est un prérequis non négociable à toute activité d'audit technique.
- Le mode d'accès (black/grey/white box) détermine la couverture et le réalisme de l'audit.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
