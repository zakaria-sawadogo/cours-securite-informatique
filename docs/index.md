# Les Cours du Dr. Zakaria Sawadogo

**Dr Zakaria Sawadogo** — École Polytechnique de Ouagadougou

Support de cours, présentations (diapositives) et travaux pratiques des 9 matières enseignées, réunis sur un seul site.

## Les matières

<div class="grid cards" markdown>

-   :material-code-braces:{ .lg .middle } **Algorithmique et Programmation en C**

    ---

    15h CM · 15h TP — Bases du C, structures de contrôle, tableaux et chaînes, pointeurs, gestion mémoire, fichiers.

    [:octicons-arrow-right-24: Accéder au cours](01-algorithmique-programmation-c/index.md)

-   :material-clipboard-search-outline:{ .lg .middle } **Audit organisation et technique**

    ---

    16h CM · 8h TP — Méthodologie d'audit, référentiels (ISO 27001/27002), tests d'intrusion, rapport d'audit.

    [:octicons-arrow-right-24: Accéder au cours](02-audit-organisation-technique/index.md)

-   :material-lock-search:{ .lg .middle } **Cryptanalyse**

    ---

    16h CM · 8h TP — Attaques sur les chiffrements classiques et modernes, fonctions de hachage, RSA, canaux auxiliaires.

    [:octicons-arrow-right-24: Accéder au cours](03-cryptanalyse/index.md)

-   :material-shield-account-outline:{ .lg .middle } **Gouvernance et gestion des risques SI**

    ---

    10h CM · 8h TP — Gouvernance SSI, méthodes d'analyse de risques (EBIOS RM), PSSI, pilotage.

    [:octicons-arrow-right-24: Accéder au cours](04-gouvernance-gestion-risques-si/index.md)

-   :material-robot-outline:{ .lg .middle } **IA appliquée à la sécurité informatique**

    ---

    16h CM · 8h TP — Détection d'intrusions et de malwares par ML, NLP pour le phishing, sécurité de l'IA.

    [:octicons-arrow-right-24: Accéder au cours](05-ia-appliquee-securite-informatique/index.md)

-   :material-key-variant:{ .lg .middle } **Cryptographie**

    ---

    16h CM · 8h TP — Chiffrement symétrique/asymétrique, hachage, signatures numériques, PKI, TLS.

    [:octicons-arrow-right-24: Accéder au cours](06-cryptographie/index.md)

-   :material-link-variant:{ .lg .middle } **Technologie Blockchain**

    ---

    16h CM · 8h TP — Structures de données, mécanismes de consensus, smart contracts, sécurité et limites.

    [:octicons-arrow-right-24: Accéder au cours](07-technologie-blockchain/index.md)

-   :material-shield-check-outline:{ .lg .middle } **Security by design**

    ---

    16h CM · 8h TP — Threat modeling, principes de conception sécurisée, zero trust, DevSecOps, privacy by design.

    [:octicons-arrow-right-24: Accéder au cours](08-security-by-design/index.md)

-   :material-application-cog-outline:{ .lg .middle } **Sécurité des applications**

    ---

    16h CM · 8h TP — OWASP Top 10, injections, authentification, XSS/CSRF, vulnérabilités mémoire, API.

    [:octicons-arrow-right-24: Accéder au cours](09-securite-des-applications/index.md)

</div>

## Volume horaire

| # | Matière | CM (h) | TP (h) | Total (h) |
|---|---------|-------:|-------:|----------:|
| 1 | Algorithmique et Programmation en C | 15 | 15 | 30 |
| 2 | Audit organisation et technique | 16 | 8 | 24 |
| 3 | Cryptanalyse | 16 | 8 | 24 |
| 4 | Gouvernance et gestion des risques de la sécurité de l'information | 10 | 8 | 18 |
| 5 | IA appliquée à la sécurité informatique | 16 | 8 | 24 |
| 6 | Cryptographie | 16 | 8 | 24 |
| 7 | Technologie Blockchain | 16 | 8 | 24 |
| 8 | Security by design | 16 | 8 | 24 |
| 9 | Sécurité des applications | 16 | 8 | 24 |
| | **Total** | **137** | **79** | **216** |

## Comment naviguer

- Le **panneau latéral** (cliquable, avec recherche) liste, pour chaque matière : le syllabus, les chapitres de cours, les présentations et les TP.
- Chaque **chapitre de cours** comporte un encart en tête de page pour ouvrir directement la présentation associée.
- Les **énoncés de TP** font également office de travaux dirigés (aucune distinction TD/TP séparée dans ce programme : chaque séance combine exercices dirigés et manipulation pratique).

Pour projeter une présentation en cours, ouvrez le lien de diapositives correspondant et utilisez les flèches du clavier ou la touche `F` pour le plein écran.

## Structure du dépôt

```
NN-nom-de-la-matiere/
├── index.md            # syllabus : objectifs, prérequis, plan de séances, évaluation, bibliographie
├── cours/                # support de cours magistral (un fichier Markdown par chapitre)
├── slides/                # sources des diapositives (.txt, Markdown reveal.js) + page d'index
└── tp/                     # énoncés de travaux pratiques / dirigés (un fichier par séance)
```

`presentation.html`, à la racine du site, est le visualiseur de diapositives (reveal.js) commun à toutes les matières.

Pour les modalités de développement local, de mise à jour du contenu et de publication du site, voir le [README](https://github.com/zakaria-sawadogo/cours-securite-informatique#readme) du dépôt.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
