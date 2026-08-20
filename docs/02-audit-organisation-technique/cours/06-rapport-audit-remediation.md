# Chapitre 6 — Rapport d'audit et plan de remédiation

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/06-rapport-audit-remediation.txt){ target=_blank }

## 1. Objectifs du rapport

Le rapport est le principal livrable d'un audit : c'est lui qui sera lu, discuté et utilisé pour arbitrer des décisions et des budgets. Un audit techniquement excellent mais mal restitué perd une grande partie de sa valeur.

## 2. Structure type d'un rapport d'audit

1. **Synthèse à destination de la direction (executive summary)** : 1 à 2 pages, sans jargon technique, avec le niveau de risque global et les priorités.
2. **Contexte et périmètre** : rappel du mandat, dates, méthode, limites de l'audit.
3. **Méthodologie** : référentiel utilisé, mode d'accès (black/grey/white box), outils.
4. **Constats détaillés**, un par vulnérabilité ou non-conformité, avec pour chacun :
   - titre et catégorie (ex. « Injection SQL sur le formulaire de connexion ») ;
   - criticité (score CVSS ou échelle qualitative Critique/Élevé/Moyen/Faible) ;
   - description technique et preuve de concept (capture, extrait de log, commande) ;
   - impact métier ;
   - recommandation de remédiation, si possible priorisée et estimée en effort.
5. **Synthèse des recommandations** sous forme de tableau priorisé.
6. **Annexes** : détails techniques, sorties d'outils, méthodologie complète.

## 3. Qualifier la criticité

Une bonne pratique consiste à combiner **vraisemblance** (facilité d'exploitation, exposition) et **impact** (confidentialité, intégrité, disponibilité, conformité réglementaire) dans une matrice de risque, plutôt que d'utiliser un score technique brut (CVSS) sans contextualisation métier — une vulnérabilité CVSS 9.8 sur un serveur de test isolé n'a pas le même risque réel qu'une vulnérabilité CVSS 6.0 sur le système de paiement en production.

## 4. Plan de remédiation

Le plan de remédiation transforme chaque recommandation en action suivie :

| Constat | Priorité | Action | Responsable | Échéance | Statut |
|---|---|---|---|---|---|
| Mot de passe admin par défaut | Critique | Changer et appliquer une politique de mots de passe forts | Équipe infra | J+7 | À faire |
| Absence de MFA sur le VPN | Élevé | Déployer une authentification à double facteur | RSSI | J+30 | En cours |
| Composant obsolète (CVE connue) | Élevé | Mettre à jour la bibliothèque concernée | Équipe dev | J+15 | À faire |

## 5. Contre-audit et suivi

Un audit de suivi (souvent partiel, ciblé sur les constats précédents) permet de vérifier l'application effective des corrections. C'est une pratique attendue dans les cycles de certification (audits de surveillance ISO 27001) et une preuve de maturité pour l'organisation auditée.

## 6. Communication et postures

- Adapter le niveau de détail au public (direction vs équipes techniques) — d'où l'intérêt de séparer synthèse et annexes techniques.
- Rester factuel et non accusatoire : l'audit évalue un système, pas les personnes.
- Respecter la confidentialité du rapport, qui contient potentiellement des informations exploitables par un attaquant (diffusion strictement limitée aux parties autorisées, destruction des données brutes après la période contractuelle convenue).

## À retenir

- Un rapport d'audit efficace sépare clairement synthèse décisionnelle et détails techniques.
- Chaque constat doit être actionnable : criticité, preuve, impact, recommandation.
- Le plan de remédiation et le contre-audit transforment l'audit en amélioration continue plutôt qu'en photographie isolée.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
