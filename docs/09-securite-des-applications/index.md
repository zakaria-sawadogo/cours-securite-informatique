# Sécurité des applications

**Volume horaire :** 16h CM · 8h TP (24h)

## Objectifs pédagogiques

- Connaître en détail les principales classes de vulnérabilités applicatives (OWASP Top 10) et leurs mécanismes d'exploitation.
- Comprendre les vulnérabilités mémoire bas niveau et leur lien avec les compétences du module *Algorithmique et Programmation en C*.
- Savoir concevoir et développer des contre-mesures efficaces à chaque classe de vulnérabilité.
- Sécuriser une API REST selon les bonnes pratiques actuelles.

## Prérequis

Ce module s'appuie fortement sur *Algorithmique et Programmation en C* (chapitre vulnérabilités mémoire), *Audit organisation et technique* (méthodologie de test, OWASP Top 10 déjà introduit) et *Security by design* (principes de conception, à appliquer concrètement ici).

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction et panorama OWASP Top 10](cours/01-introduction-owasp-top10.md) | 2h |
| 2 | [Vulnérabilités d'injection](cours/02-vulnerabilites-injection.md) | 3h |
| 3 | [Authentification et gestion de sessions](cours/03-authentification-gestion-sessions.md) | 3h |
| 4 | [Vulnérabilités côté client : XSS, CSRF, clickjacking](cours/04-vulnerabilites-cote-client.md) | 3h |
| 5 | [Vulnérabilités mémoire bas niveau et exploitation](cours/05-vulnerabilites-memoire-exploitation.md) | 2h |
| 6 | [Sécurisation des API et bonnes pratiques](cours/06-securisation-api-bonnes-pratiques.md) | 3h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Injection SQL et contre-mesures](tp/tp1-injection-sql.md) | 2h |
| 2 | [XSS et CSRF sur application vulnérable](tp/tp2-xss-csrf.md) | 2h |
| 3 | [Exploitation contrôlée d'un buffer overflow en C](tp/tp3-buffer-overflow-c.md) | 2h |
| 4 | [Sécurisation d'une API REST](tp/tp4-securisation-api-rest.md) | 2h |

## Évaluation

- TP notés : 40 %
- Analyse de vulnérabilité (CVE réelle au choix, rapport écrit) : 20 %
- Examen final : 40 %

## Cadre légal

Comme pour le module *Audit organisation et technique*, toute manipulation d'exploitation s'effectue exclusivement sur des cibles pédagogiques déployées localement (DVWA, Juice Shop, binaires fournis pour le TP3). Aucune manipulation n'est autorisée en dehors de ce cadre.

## Bibliographie

- OWASP Top 10, OWASP Application Security Verification Standard (ASVS), OWASP Cheat Sheet Series.
- E. Foster et al., *Buffer Overflow Attacks*.
- Documentation OWASP API Security Top 10.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
