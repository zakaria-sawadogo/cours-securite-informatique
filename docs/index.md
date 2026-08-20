# Programme — Master Sécurité Informatique

Dépôt pédagogique regroupant les supports de cours (CM), les présentations (diapositives), les travaux pratiques (TP) et les syllabus des 9 matières du programme.

## 🌐 Accès en ligne (site du cours)

Ce dépôt est publié comme site de cours via **GitHub Pages**. Une fois activé (voir instructions du dépôt), il est accessible à l'adresse `https://<votre-compte>.github.io/<nom-du-depot>/` et offre :

- un **sommaire par matière** dans le panneau latéral (cliquable, avec recherche) ;
- le **support de cours** de chaque chapitre, à lire directement dans le navigateur ;
- les **présentations** (diapositives) associées à chaque chapitre, pour un enseignement en amphi ou en visioconférence ;
- les **énoncés de TP**, qui font également office de travaux dirigés (aucune distinction TD/TP séparée dans ce programme : chaque séance de TP combine exercices dirigés et manipulation pratique).

Sans passer par le site, chaque ressource reste consultable directement dans ce dépôt (voir structure ci-dessous).

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

## Structure du dépôt

Chaque matière dispose de son propre dossier, avec le même schéma :

```
NN-nom-de-la-matiere/
├── README.md          # syllabus : objectifs, prérequis, plan de séances, évaluation, bibliographie
├── cours/              # support de cours magistral (un fichier Markdown par chapitre)
├── slides/             # diapositives associées à chaque chapitre (Markdown reveal.js)
└── tp/                 # énoncés de travaux pratiques / dirigés (un fichier par séance)
```

Deux fichiers à la racine du dépôt font fonctionner le site :

- `index.html` + `_sidebar.md` : page d'accueil et sommaire du site (docsify).
- `presentation.html` : visualiseur de diapositives (reveal.js). Chaque lien de présentation prend la forme `presentation.html?deck=<matiere>/slides/<chapitre>.md`.

## Liste des matières

1. [Algorithmique et Programmation en C](01-algorithmique-programmation-c/index.md)
2. [Audit organisation et technique](02-audit-organisation-technique/index.md)
3. [Cryptanalyse](03-cryptanalyse/index.md)
4. [Gouvernance et gestion des risques de la sécurité de l'information](04-gouvernance-gestion-risques-si/index.md)
5. [IA appliquée à la sécurité informatique](05-ia-appliquee-securite-informatique/index.md)
6. [Cryptographie](06-cryptographie/index.md)
7. [Technologie Blockchain](07-technologie-blockchain/index.md)
8. [Security by design](08-security-by-design/index.md)
9. [Sécurité des applications](09-securite-des-applications/index.md)

## Utiliser ce dépôt

```bash
git clone <url-de-votre-depot>
cd programme-cybersecurite
```

Chaque dossier de matière est autonome : le `README.md` sert de plan de cours, les fichiers de `cours/` peuvent être ouverts directement en Markdown, les fichiers de `slides/` sont les mêmes chapitres découpés pour la présentation en diapositives, et les fichiers de `tp/` contiennent les énoncés ainsi que, quand c'est pertinent, du code de démarrage ou des corrigés indicatifs.

## Activer l'accès étudiant via GitHub Pages

1. Poussez ce dépôt sur GitHub (voir instructions dans le message d'accompagnement).
2. Sur GitHub : **Settings → Pages**.
3. Sous « Build and deployment », choisissez **Source : Deploy from a branch**.
4. Sélectionnez la branche `main` et le dossier **`/ (root)`**, puis **Save**.
5. Après quelques minutes, le site est disponible à `https://<votre-compte>.github.io/<nom-du-depot>/`. Partagez ce lien avec les étudiants — aucune installation n'est nécessaire de leur côté, tout fonctionne dans le navigateur (le site n'a pas d'étape de compilation : docsify et reveal.js sont chargés directement).

Pour projeter une présentation en cours, ouvrez le lien de diapositives correspondant depuis le sommaire et utilisez les flèches du clavier ou la touche `F` pour le plein écran.
