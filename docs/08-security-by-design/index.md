# Security by design

**Volume horaire :** 16h CM · 8h TP (24h)

## Objectifs pédagogiques

- Comprendre les principes fondamentaux de conception sécurisée, à intégrer dès la phase de conception plutôt qu'ajoutés a posteriori.
- Maîtriser une démarche de modélisation des menaces (threat modeling).
- Comprendre les architectures de sécurité modernes (défense en profondeur, zero trust).
- Intégrer la sécurité dans le cycle de développement logiciel (DevSecOps).
- Comprendre les principes de *privacy by design*.

## Prérequis

Ce module s'appuie sur les notions vues en *Audit organisation et technique* (OWASP Top 10, méthodologie d'audit) et bénéficie d'une bonne compréhension du développement logiciel (module *Algorithmique et Programmation en C*).

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction : principes de security by design et privacy by design](cours/01-introduction-security-by-design.md) | 2h |
| 2 | [Modélisation des menaces (threat modeling)](cours/02-modelisation-menaces.md) | 3h |
| 3 | [Principes de conception sécurisée](cours/03-principes-conception-securisee.md) | 3h |
| 4 | [Architecture sécurisée : segmentation et zero trust](cours/04-architecture-securisee-zero-trust.md) | 3h |
| 5 | [Cycle de développement sécurisé (DevSecOps)](cours/05-cycle-developpement-securise-devsecops.md) | 2h |
| 6 | [Privacy by design](cours/06-privacy-by-design.md) | 3h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Threat modeling STRIDE sur une application](tp/tp1-threat-modeling-stride.md) | 2h |
| 2 | [Application des principes de conception sécurisée](tp/tp2-principes-conception-securisee.md) | 2h |
| 3 | [Conception d'une architecture zero trust](tp/tp3-architecture-zero-trust.md) | 2h |
| 4 | [Intégration de contrôles de sécurité en CI/CD (DevSecOps)](tp/tp4-devsecops-cicd.md) | 2h |

## Évaluation

- TP notés : 40 %
- Étude de cas de threat modeling notée : 20 %
- Examen final : 40 %

## Bibliographie

- A. Shostack, *Threat Modeling: Designing for Security*.
- OWASP Top 10, OWASP Application Security Verification Standard (ASVS).
- NIST SP 800-207 — Zero Trust Architecture.
- CNIL / autorités de protection des données — guides *Privacy by Design*.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
