# Chapitre 4 — Pilotage, indicateurs et conformité réglementaire

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=04-gouvernance-gestion-risques-si/slides/04-pilotage-indicateurs-conformite.txt){ target=_blank }

## 1. Pourquoi piloter par les indicateurs

Une gouvernance SSI mature ne se limite pas à produire des documents (PSSI, analyses de risque) : elle démontre, dans la durée, une amélioration mesurable. Les indicateurs permettent de communiquer objectivement avec la direction, de détecter une dérive avant qu'elle ne devienne un incident, et de justifier les investissements.

## 2. Caractéristiques d'un bon indicateur

Un indicateur utile est généralement décrit par les critères **SMART** : Spécifique, Mesurable, Atteignable, Réaliste, Temporellement défini. Il doit surtout être **actionnable** : un indicateur qui ne débouche sur aucune décision possible n'a pas d'intérêt opérationnel.

## 3. Exemples d'indicateurs SSI

| Catégorie | Exemple d'indicateur |
|---|---|
| Vulnérabilités | délai moyen d'application des correctifs critiques (patch management) |
| Incidents | nombre d'incidents de sécurité déclarés par mois, temps moyen de détection (MTTD), temps moyen de résolution (MTTR) |
| Sensibilisation | taux de clics sur des campagnes de simulation de phishing |
| Conformité | pourcentage d'exigences ISO 27001 couvertes, nombre de non-conformités ouvertes |
| Continuité | résultat des derniers tests de PCA/PRA (RTO/RPO atteints ou non) |
| Accès | nombre de comptes à privilèges, délai de révocation des accès après départ d'un collaborateur |

## 4. Tableau de bord SSI

Un tableau de bord synthétise ces indicateurs pour différents publics :

- **Vision direction** : quelques indicateurs agrégés, tendance (amélioration/dégradation), niveau de risque global.
- **Vision RSSI/opérationnelle** : indicateurs détaillés par domaine, permettant d'identifier les points d'action prioritaires.

La fréquence de mise à jour (mensuelle, trimestrielle) et le format (comité de pilotage, revue de direction) doivent être définis dans la gouvernance elle-même.

## 5. Conformité réglementaire : panorama non exhaustif

| Cadre réglementaire | Portée | Point clé |
|---|---|---|
| **RGPD** (et lois nationales équivalentes de protection des données) | données à caractère personnel | notification des violations, analyse d'impact (AIPD) pour les traitements à risque |
| **Réglementations sectorielles** (banque, assurance, santé, télécoms) | secteurs régulés | exigences souvent renforcées (ex. résilience opérationnelle dans le secteur financier) |
| **Réglementations nationales de cybersécurité** (souvent inspirées de directives régionales) | opérateurs d'importance vitale ou essentiels | obligations de déclaration d'incident, exigences techniques minimales |
| **Exigences contractuelles** | relations clients/fournisseurs | souvent alignées sur ISO 27001 ou des questionnaires de sécurité spécifiques |

La conformité n'est pas un objectif en soi mais un sous-ensemble de la gestion des risques : une organisation conforme n'est pas nécessairement sécurisée, et inversement une organisation bien sécurisée doit néanmoins documenter sa conformité pour ses obligations légales et contractuelles.

## 6. Revue de direction et amélioration continue

L'ISO 27001 impose une revue de direction périodique du SMSI, qui doit examiner : les résultats des audits internes/externes, l'état d'avancement du traitement des risques, les indicateurs de performance, les retours des parties intéressées, et déboucher sur des décisions (allocation de ressources, ajustement des priorités). C'est le point de bouclage du cycle PDCA introduit au chapitre 1.

## À retenir

- Un indicateur SSI doit être actionnable, pas seulement descriptif.
- Le tableau de bord se décline selon le public (direction vs opérationnel) pour rester pertinent à chaque niveau.
- La conformité réglementaire est une composante de la gestion des risques, pas un objectif indépendant ; elle se pilote au même titre que les autres indicateurs.
