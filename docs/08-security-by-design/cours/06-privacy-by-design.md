# Chapitre 6 — Privacy by design

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/06-privacy-by-design.txt){ target=_blank }

## 1. Définition et origine

Le *privacy by design* (protection de la vie privée dès la conception) est un cadre formalisé par Ann Cavoukian, articulé autour de sept principes fondateurs, visant à intégrer la protection des données personnelles comme caractéristique intrinsèque d'un système, dès sa conception, plutôt que comme un ajout réglementaire tardif.

## 2. Les sept principes fondateurs

1. **Proactif, non réactif ; préventif, non correctif** : anticiper les risques pour la vie privée avant qu'un incident ne survienne.
2. **Protection de la vie privée par défaut** : les paramètres les plus protecteurs doivent être activés sans action requise de l'utilisateur.
3. **Protection intégrée à la conception** : composante structurelle du système, pas une fonctionnalité ajoutée après coup.
4. **Fonctionnalité intégrale (« somme positive », pas somme nulle)** : rechercher des solutions où sécurité/fonctionnalité et vie privée coexistent, plutôt que de présenter systématiquement un arbitrage l'un contre l'autre.
5. **Sécurité de bout en bout** : protection tout au long du cycle de vie de la donnée, de la collecte à la suppression.
6. **Visibilité et transparence** : les pratiques réelles doivent être vérifiables, pas seulement déclarées.
7. **Respect de la vie privée des utilisateurs** : conception centrée sur l'intérêt de la personne concernée, avec des interfaces et paramètres clairs et accessibles.

## 3. Traduction juridique : article 25 du RGPD

Le RGPD (Règlement Général sur la Protection des Données, Union européenne, largement pris en référence au-delà de sa portée juridique directe) reprend ce principe dans son article 25 : « protection des données dès la conception et par défaut » (*privacy by design and by default*), imposant concrètement au responsable de traitement de mettre en œuvre des mesures techniques et organisationnelles appropriées dès la conception d'un traitement de données personnelles.

## 4. Principes de minimisation

- **Minimisation de la collecte** : ne collecter que les données strictement nécessaires à la finalité poursuivie, pas « au cas où » elles seraient utiles plus tard.
- **Minimisation de la conservation** : définir une durée de conservation limitée et justifiée, avec suppression ou anonymisation effective au-delà.
- **Minimisation de l'accès** : appliquer le principe du moindre privilège (chapitre 3) spécifiquement aux données personnelles.
- **Minimisation de l'identifiabilité** : privilégier, quand la finalité le permet, des données pseudonymisées ou anonymisées plutôt que directement identifiantes.

## 5. Techniques de protection de la vie privée

| Technique | Principe |
|---|---|
| **Pseudonymisation** | remplacer un identifiant direct par un pseudonyme, la ré-identification restant possible avec une information additionnelle gardée séparément |
| **Anonymisation** | rendre la ré-identification impossible, y compris par recoupement avec d'autres sources (plus difficile à garantir réellement que ce que l'on croit souvent) |
| **Chiffrement** | protège la confidentialité des données au repos et en transit (lien direct avec le module *Cryptographie*) |
| **Confidentialité différentielle (differential privacy)** | ajout de bruit statistique calibré à des résultats d'agrégation pour empêcher la ré-identification d'un individu tout en préservant l'utilité statistique globale |
| **Contrôle d'accès et journalisation** | limiter et tracer qui accède à quelles données personnelles |

## 6. Analyse d'impact relative à la protection des données (AIPD/DPIA)

Pour un traitement de données présentant un risque élevé pour les droits des personnes, une analyse d'impact formelle est requise par de nombreuses réglementations : elle documente la finalité, la nécessité et la proportionnalité du traitement, évalue les risques pour les personnes concernées, et identifie les mesures pour les atténuer — une démarche directement apparentée à l'analyse de risque du module *Gouvernance et gestion des risques SI*, appliquée spécifiquement à la vie privée.

## 7. LINDDUN : threat modeling orienté vie privée

Complément du chapitre 2 : LINDDUN structure l'identification de menaces spécifiques à la vie privée (Linkability, Identifiability, Non-repudiation, Detectability, Disclosure of information, Unawareness, Non-compliance), sur le même principe que STRIDE mais orienté vers les atteintes à la vie privée plutôt que vers les propriétés de sécurité classiques.

## À retenir

- Le privacy by design intègre la protection des données personnelles comme caractéristique par défaut d'un système, formalisé juridiquement par l'article 25 du RGPD et des dispositions équivalentes ailleurs.
- La minimisation (collecte, conservation, accès, identifiabilité) est le principe opérationnel le plus directement actionnable en conception.
- LINDDUN transpose la logique de threat modeling (STRIDE) spécifiquement aux risques pour la vie privée, complétant l'analyse de sécurité classique.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
