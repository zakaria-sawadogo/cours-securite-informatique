# Chapitre 2 — Audit organisationnel : normes et référentiels

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/02-audit-organisationnel-referentiels.txt){ target=_blank }

## 1. ISO/IEC 27001 et 27002

- **ISO/IEC 27001** : norme certifiable définissant les exigences d'un Système de Management de la Sécurité de l'Information (SMSI). Structurée autour du cycle **PDCA** (Plan-Do-Check-Act).
- **ISO/IEC 27002** : catalogue de mesures de sécurité (bonnes pratiques) organisées en thèmes (organisationnel, humain, physique, technologique) que l'ISO 27001 référence.

### Clauses principales de l'ISO 27001 (4 à 10)

| Clause | Contenu |
|---|---|
| 4 | Contexte de l'organisation |
| 5 | Leadership (engagement de la direction, politique SSI) |
| 6 | Planification (appréciation et traitement des risques) |
| 7 | Support (ressources, compétences, sensibilisation) |
| 8 | Fonctionnement (mise en œuvre opérationnelle) |
| 9 | Évaluation des performances (audit interne, revue de direction) |
| 10 | Amélioration (non-conformités, actions correctives) |

Un audit organisationnel vérifie, pour chaque clause, l'existence de **preuves documentaires** (politiques, procédures) et de **preuves de mise en œuvre effective** (entretiens, échantillons de tickets, journaux d'activité).

## 2. Autres référentiels usuels

| Référentiel | Portée | Usage typique |
|---|---|---|
| **PCI-DSS** | données de cartes bancaires | commerçants et prestataires de paiement |
| **NIST Cybersecurity Framework** | gestion du risque cyber (Identify, Protect, Detect, Respond, Recover) | référence internationale, souvent utilisée hors certification |
| **ANSSI (guides et référentiels)** | administrations et OIV en France, largement repris en Afrique francophone | hygiène informatique, PASSI (prestataires d'audit) |
| **RGPD** | protection des données personnelles | conformité juridique, articulé avec la sécurité technique |
| **CIS Controls** | mesures techniques priorisées | audit technique et durcissement (*hardening*) |

## 3. Méthodologie d'un audit organisationnel

1. **Revue documentaire** : politiques de sécurité, chartes, procédures de gestion des incidents, plan de continuité d'activité (PCA/PRA).
2. **Entretiens** avec les parties prenantes (RSSI, DSI, RH, métiers) pour vérifier l'application réelle des procédures.
3. **Échantillonnage** : contrôle de tickets, de comptes utilisateurs, de journaux, de contrats prestataires.
4. **Analyse d'écart (gap analysis)** entre l'état constaté et les exigences du référentiel.
5. **Notation de maturité**, souvent sur une échelle (ex. modèle CMMI adapté : 0 = inexistant à 5 = optimisé).

## 4. Exemple de grille d'évaluation simplifiée

| Domaine | Exigence | Constat | Niveau de maturité (0-5) |
|---|---|---|---|
| Gestion des accès | Revue périodique des droits | Aucune revue formalisée depuis 18 mois | 1 |
| Gestion des correctifs | Politique de patch management documentée | Politique existe, appliquée à 60 % des serveurs | 2 |
| Sensibilisation | Formation annuelle du personnel | Formation dispensée à l'embauche uniquement | 2 |

## 5. Risques d'un audit organisationnel mal mené

- Se limiter à la lecture des documents sans vérifier l'application réelle (« conformité de façade »).
- Confondre existence d'une procédure et efficacité de la procédure.
- Absence de priorisation : toutes les non-conformités ne présentent pas le même risque.

## À retenir

- L'audit organisationnel évalue des processus et de la documentation, pas uniquement des configurations techniques.
- ISO 27001/27002 est le référentiel le plus largement utilisé ; il se combine souvent avec des référentiels sectoriels (PCI-DSS) ou nationaux (ANSSI).
- Une non-conformité doit toujours être qualifiée par un niveau de criticité et une recommandation actionnable.
