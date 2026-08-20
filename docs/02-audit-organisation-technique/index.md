# Audit organisation et technique

**Volume horaire :** 16h CM · 8h TP (24h)

## Objectifs pédagogiques

- Distinguer audit organisationnel et audit technique, et situer chacun dans une démarche globale de gestion de la sécurité.
- Connaître les principaux référentiels et normes utilisés en audit (ISO 27001/27002, PCI-DSS, ANSSI, NIST).
- Mener une démarche d'audit technique (tests d'intrusion) selon une méthodologie structurée (reconnaissance, scan, exploitation, reporting).
- Rédiger un rapport d'audit exploitable par une direction et par des équipes techniques.

## Prérequis

Notions de base en réseaux (TCP/IP) et en systèmes d'exploitation ; le cours *Algorithmique et Programmation en C* n'est pas un prérequis strict mais la lecture de scripts est un atout.

## Plan de séances

### Cours magistraux (16h)

| Séance | Chapitre | Durée |
|---|---|---|
| 1 | [Introduction à l'audit de sécurité](cours/01-introduction-audit-securite.md) | 2h |
| 2 | [Audit organisationnel : normes et référentiels](cours/02-audit-organisationnel-referentiels.md) | 3h |
| 3 | [Audit technique : méthodologie des tests d'intrusion](cours/03-audit-technique-methodologie.md) | 3h |
| 4 | [Audit des infrastructures réseau et systèmes](cours/04-audit-infrastructures.md) | 3h |
| 5 | [Audit des applications et du code](cours/05-audit-applications.md) | 3h |
| 6 | [Rapport d'audit et plan de remédiation](cours/06-rapport-audit-remediation.md) | 2h |

### Travaux pratiques (8h)

| Séance | Sujet | Durée |
|---|---|---|
| 1 | [Audit organisationnel sur cas d'étude (ISO 27001)](tp/tp1-audit-organisationnel.md) | 2h |
| 2 | [Reconnaissance et scan réseau (Nmap)](tp/tp2-reconnaissance-scan.md) | 2h |
| 3 | [Test d'intrusion applicatif (OWASP Top 10)](tp/tp3-pentest-applicatif.md) | 2h |
| 4 | [Rédaction d'un rapport d'audit complet](tp/tp4-rapport-audit.md) | 2h |

## Évaluation

- Participation et livrables de TP : 30 %
- Étude de cas notée (analyse d'un référentiel) : 20 %
- Examen final (méthodologie + étude de cas d'audit) : 50 %

## Environnement technique des TP

Machine virtuelle Kali Linux ou équivalent, cible d'entraînement légale (Metasploitable2, OWASP Juice Shop, DVWA) déployée en local — **tout test d'intrusion hors de ce cadre pédagogique nécessite une autorisation écrite explicite**, rappelée en séance 1.

## Bibliographie

- ISO/IEC 27001:2022, ISO/IEC 27002:2022.
- ANSSI, *Guide d'hygiène informatique*.
- PTES (Penetration Testing Execution Standard) — [pentest-standard.org](http://www.pentest-standard.org).
- OWASP Testing Guide.
