# Master Sécurité Informatique — site de cours (MkDocs Material)

Site du programme Master Sécurité Informatique : support de cours, présentations (diapositives) et travaux pratiques des 9 matières, publié via GitHub Pages.

**Note :** ce fichier est le README technique du dépôt (visible sur GitHub). La page d'accueil du site lui-même se trouve dans `docs/index.md`.

## Structure

```
mkdocs.yml              # configuration du site (thème, navigation, extensions)
requirements.txt         # dépendances Python (mkdocs-material)
.github/workflows/       # publication automatique à chaque push sur main
docs/
├── index.md             # page d'accueil du site
├── presentation.html    # visualiseur de diapositives (reveal.js), commun à toutes les matières
├── assets/
└── NN-nom-de-la-matiere/
    ├── index.md          # syllabus de la matière
    ├── cours/             # chapitres de cours (Markdown, avec lien vers la présentation associée)
    ├── slides/             # sources des diapositives (.txt, format Markdown reveal.js) + index.md
    └── tp/                 # énoncés de travaux pratiques
```

## Développement local

```bash
pip install -r requirements.txt
mkdocs serve
```

Le site est alors disponible sur `http://127.0.0.1:8000/`, avec rechargement automatique à chaque modification.

## Publication (GitHub Pages)

La publication est automatisée par `.github/workflows/deploy.yml` : à chaque push sur `main`, le site est construit puis poussé sur la branche `gh-pages`.

### Activation (une seule fois)

1. Poussez ce dépôt sur GitHub (dépôt `cours-securite-informatique`, ou le nom de votre choix — pensez à ajuster `site_url` et `repo_url` dans `mkdocs.yml` si vous changez le nom).
2. Le premier push sur `main` déclenche l'action, qui crée automatiquement la branche `gh-pages`.
3. Sur GitHub : **Settings → Pages**. Sous « Build and deployment » → **Source : Deploy from a branch**. Choisissez la branche **`gh-pages`**, dossier **`/ (root)`**, puis **Save**.
4. Après quelques minutes, le site est disponible à `https://<votre-compte>.github.io/<nom-du-depot>/`.

Les mises à jour suivantes sont automatiques : il suffit de modifier un fichier Markdown dans `docs/` et de pousser sur `main`.

## Ajouter un nouveau chapitre ou une nouvelle matière

1. Ajoutez le fichier Markdown au bon endroit sous `docs/<matiere>/cours/` (ou `tp/`).
2. Déclarez-le dans la section `nav:` de `mkdocs.yml`, au même format que les entrées existantes.
3. Si le chapitre a une présentation associée, ajoutez le fichier `.txt` correspondant (Markdown reveal.js, séparateur `---` isolé) dans `docs/<matiere>/slides/`, et un lien dans `docs/<matiere>/slides/index.md` ainsi qu'un encart en tête du chapitre (voir un chapitre existant comme modèle).

## Intégration avec le portfolio

Une entrée pointant vers ce site a été ajoutée à `data/courses.json` du portfolio (`zs`), affichée dans sa section *Teaching* — voir le patch fourni séparément.
