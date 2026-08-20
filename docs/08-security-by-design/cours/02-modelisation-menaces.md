# Chapitre 2 — Modélisation des menaces (threat modeling)

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/02-modelisation-menaces.txt){ target=_blank }

## 1. Qu'est-ce que le threat modeling ?

Le threat modeling est une démarche structurée visant à identifier, de façon systématique et le plus en amont possible, les menaces plausibles pesant sur un système, afin de concevoir des mesures de sécurité proportionnées avant l'implémentation. C'est l'un des outils centraux de *security by design*.

Il s'agit d'un exercice d'anticipation plutôt que de détection : contrairement à un test d'intrusion, qui évalue un système déjà existant en cherchant à l'exploiter concrètement, le threat modeling s'exerce idéalement sur une représentation encore abstraite du système (schéma d'architecture, diagramme de flux) avant même que le code ne soit écrit. Cette différence de nature explique pourquoi threat modeling et tests d'intrusion sont complémentaires plutôt que substituables : le premier balaie systématiquement l'espace des menaces plausibles à moindre coût, le second valide empiriquement la résistance effective d'un système réel, mais seulement sur les chemins que le testeur choisit d'explorer dans le temps imparti.

## 2. La démarche générale en quatre questions (Adam Shostack)

1. **Sur quoi travaillons-nous ?** (modéliser le système : diagramme de flux de données, composants, frontières de confiance).
2. **Qu'est-ce qui peut mal tourner ?** (identification systématique des menaces).
3. **Que faisons-nous à ce sujet ?** (définir des contre-mesures).
4. **Avons-nous fait du bon travail ?** (valider la démarche, itérer).

Cette formulation en quatre questions, popularisée par Adam Shostack (auteur de référence sur le sujet, ancien responsable du programme de threat modeling chez Microsoft), a l'avantage de rester indépendante de tout outillage ou méthodologie spécifique : elle peut se mener sur un tableau blanc avec des post-it, comme dans un atelier outillé avec un logiciel dédié de modélisation. La quatrième question est souvent la plus négligée en pratique : un threat modeling mené une seule fois, au lancement du projet, puis jamais révisé, perd rapidement sa pertinence à mesure que l'architecture évolue.

## 3. Diagramme de flux de données (DFD) et frontières de confiance

Avant d'identifier des menaces, il faut représenter le système : processus, flux de données, magasins de données, entités externes, et surtout les **frontières de confiance** (points où le niveau de confiance change, ex. entre un client non authentifié et un serveur, entre une DMZ et un réseau interne). Les menaces se concentrent typiquement sur ces frontières.

Un diagramme de flux de données utilise conventionnellement quatre types d'éléments :

| Élément | Symbole usuel | Exemple |
|---|---|---|
| Entité externe | rectangle | navigateur d'un utilisateur, système tiers partenaire |
| Processus | cercle ou ellipse | serveur d'application, service d'authentification |
| Magasin de données | deux traits parallèles | base de données, système de fichiers, cache |
| Flux de données | flèche | requête HTTP, appel API interne, écriture en base |

Les frontières de confiance sont représentées par une ligne en pointillés traversant le diagramme, séparant des zones de niveaux de confiance différents (ex. Internet public / DMZ / réseau interne / zone d'administration). En pratique, l'expérience montre que la majorité des menaces exploitables se situent précisément au croisement d'un flux de données et d'une frontière de confiance — c'est-à-dire aux points où une donnée ou une requête passe d'un contexte de confiance à un autre, et où une vérification (authentification, autorisation, validation d'entrée) doit impérativement avoir lieu.

## 4. STRIDE : classification des menaces

Modèle développé par Microsoft, six catégories de menaces à examiner systématiquement pour chaque composant/flux du diagramme :

| Lettre | Menace | Propriété de sécurité visée | Exemple |
|---|---|---|---|
| **S** | Spoofing (usurpation d'identité) | Authenticité | se faire passer pour un autre utilisateur |
| **T** | Tampering (falsification) | Intégrité | modifier des données en transit ou au repos |
| **R** | Repudiation (répudiation) | Non-répudiation | nier avoir effectué une action, en l'absence de traçabilité |
| **I** | Information disclosure (divulgation d'information) | Confidentialité | accéder à des données sans autorisation |
| **D** | Denial of Service (déni de service) | Disponibilité | rendre un service indisponible |
| **E** | Elevation of Privilege (élévation de privilège) | Autorisation | obtenir des droits supérieurs à ceux accordés |

Pour chaque flux/composant du diagramme, on se demande systématiquement : ce composant est-il vulnérable à S ? à T ? etc.

### Exemple d'application de STRIDE

Considérons, à titre pédagogique, un flux simple : un formulaire de connexion d'une application fictive « GestionCours » envoyant les identifiants saisis par l'utilisateur vers un service d'authentification, qui interroge ensuite une base de données de comptes. L'application systématique de STRIDE à ce flux unique peut donner :

| Catégorie STRIDE | Menace identifiée sur ce flux | Contre-mesure envisageable |
|---|---|---|
| Spoofing | un attaquant intercepte des identifiants valides et se connecte à la place de l'utilisateur légitime | authentification à double facteur, protection contre les attaques par force brute |
| Tampering | les identifiants sont modifiés en transit entre le navigateur et le serveur | chiffrement du canal (TLS) |
| Repudiation | un utilisateur nie s'être connecté à une date donnée | journalisation horodatée et intègre des tentatives de connexion |
| Information disclosure | un message d'erreur de connexion révèle si un identifiant existe (« mot de passe incorrect » vs « compte inexistant ») | messages d'erreur génériques ne distinguant pas les deux cas |
| Denial of Service | un attaquant sature le service d'authentification par un grand nombre de tentatives | limitation de débit (*rate limiting*), verrouillage temporaire progressif |
| Elevation of Privilege | une faille dans le service d'authentification permet d'obtenir directement un jeton avec des droits d'administrateur | validation stricte côté serveur des droits attribués, indépendamment de toute donnée fournie par le client |

Cet exemple illustre un point méthodologique important : STRIDE ne remplace pas l'expertise du concepteur, il structure et systématise son raisonnement pour éviter d'oublier une catégorie entière de menaces sur un composant donné — l'erreur la plus fréquente en l'absence de méthode étant de se concentrer intuitivement sur une ou deux catégories familières (souvent la divulgation d'information) en négligeant les autres (répudiation, déni de service, en particulier).

## 5. DREAD : évaluation de la sévérité

Modèle complémentaire (moins utilisé aujourd'hui de façon isolée, souvent remplacé par une matrice de risque classique ou CVSS) pour scorer une menace identifiée selon cinq critères : **D**amage (dommage potentiel), **R**eproducibility (reproductibilité), **E**xploitability (facilité d'exploitation), **A**ffected users (utilisateurs affectés), **D**iscoverability (facilité de découverte).

Chaque critère est généralement noté sur une petite échelle (par exemple de 1 à 3, ou de 1 à 10 selon les variantes), puis les scores sont combinés (souvent par une moyenne) pour obtenir un score global permettant de prioriser les menaces identifiées les unes par rapport aux autres. DREAD a été critiqué pour la subjectivité de la notation (deux évaluateurs peuvent attribuer des scores très différents à la même menace) et pour l'absence de méthodologie publique consensuelle sur la pondération des critères, ce qui explique que de nombreuses organisations lui préfèrent aujourd'hui des référentiels de scoring plus standardisés, tel le **CVSS** (Common Vulnerability Scoring System) pour les vulnérabilités déjà caractérisées, ou une simple matrice de risque probabilité × impact (vue en *Gouvernance et gestion des risques SI*) adaptée au contexte du threat modeling.

## 6. Arbres d'attaque (attack trees)

Représentation hiérarchique où le nœud racine est l'objectif de l'attaquant (ex. « compromettre le compte administrateur »), décomposé en sous-objectifs (nœuds enfants) reliés par des relations logiques ET/OU, jusqu'à des actions concrètes réalisables. Cette structure aide à visualiser les différents chemins d'attaque possibles et à identifier où une contre-mesure unique bloque plusieurs branches.

### Exemple simplifié d'arbre d'attaque

```text
Objectif : Compromettre le compte administrateur (racine)
├── (OU) Deviner ou obtenir le mot de passe
│   ├── (OU) Attaque par force brute sur le formulaire de connexion
│   ├── (OU) Réutilisation d'un mot de passe divulgué lors d'une fuite tierce
│   └── (OU) Hameçonnage ciblé de l'administrateur
├── (OU) Exploiter une vulnérabilité applicative pour contourner l'authentification
│   ├── (ET) Trouver un point d'injection ET obtenir une élévation de privilège
│   └── (OU) Exploiter une session administrateur non expirée sur un poste partagé
└── (OU) Compromettre un poste de travail de l'administrateur
    ├── (OU) Logiciel malveillant via une pièce jointe
    └── (OU) Vulnérabilité non corrigée du système d'exploitation du poste
```

La lecture d'un tel arbre permet d'identifier des contre-mesures à fort effet de levier : dans l'exemple ci-dessus, l'authentification à double facteur affaiblit simultanément plusieurs branches liées à l'obtention du mot de passe (force brute, réutilisation, hameçonnage), sans nécessiter de traiter chaque branche isolément par une mesure dédiée. C'est un des intérêts pédagogiques majeurs des arbres d'attaque : ils rendent visibles les contre-mesures qui protègent contre plusieurs scénarios à la fois, information moins immédiate dans une simple liste de menaces STRIDE traitées composant par composant.

## 7. PASTA et autres méthodologies

D'autres méthodologies existent, orientées différemment : **PASTA** (Process for Attack Simulation and Threat Analysis) adopte une perspective centrée sur le risque métier et simule des scénarios d'attaque complets ; **LINDDUN** est une méthodologie spécifiquement dédiée aux menaces sur la vie privée (à mettre en lien avec le chapitre 6, *privacy by design*).

PASTA se distingue de STRIDE par son point de départ : alors que STRIDE part du système (diagramme technique) pour remonter vers les menaces, PASTA part explicitement des objectifs métier et des impacts business, en sept étapes (définition des objectifs, définition du périmètre technique, décomposition de l'application, analyse des menaces, identification des vulnérabilités, modélisation des attaques, analyse des risques et de l'impact résiduel). Cette orientation en fait une méthodologie plus lourde mais souvent mieux adaptée à des discussions avec des parties prenantes non techniques (direction, propriétaires métier d'une application), pour qui un scénario d'attaque exprimé en termes d'impact financier ou réputationnel est plus parlant qu'une liste technique de catégories STRIDE.

## 8. Quand et comment mener un threat modeling

- Idéalement en phase de conception, avant l'écriture du code, et **révisé** à chaque évolution architecturale significative.
- En atelier collaboratif, associant développeurs, architectes et personnel sécurité — le threat modeling gagne à ne pas être l'exercice isolé d'un seul expert sécurité déconnecté de l'équipe de développement.
- Documenté et priorisé, pour alimenter directement le backlog de développement (contre-mesures = user stories/tâches techniques).

### Intégration au cycle de vie du projet

Dans une organisation mature, le threat modeling n'est pas un événement isolé mais un point de contrôle récurrent, déclenché par des événements précis plutôt que par un calendrier arbitraire : ajout d'une nouvelle fonctionnalité traitant des données sensibles, modification significative de l'architecture (ajout d'un nouveau composant externe, changement de fournisseur Cloud), ou constat d'un incident de sécurité nécessitant une réévaluation. Cette approche évite deux écueils symétriques : le threat modeling réalisé une seule fois au lancement puis jamais mis à jour (qui devient rapidement obsolète), et le threat modeling refait intégralement à chaque itération mineure (qui consomme un temps disproportionné par rapport au bénéfice).

## À retenir

- Le threat modeling identifie systématiquement les menaces à partir d'une représentation du système et de ses frontières de confiance, avant l'implémentation ; il est complémentaire, et non substituable, aux tests d'intrusion menés a posteriori.
- La démarche en quatre questions d'Adam Shostack (sur quoi travaillons-nous, qu'est-ce qui peut mal tourner, que faisons-nous, avons-nous bien fait) structure l'exercice indépendamment de tout outillage spécifique.
- STRIDE structure l'identification des menaces par catégorie, en miroir des propriétés de sécurité visées ; son application systématique, composant par composant et flux par flux, évite l'oubli intuitif de catégories entières de menaces.
- Les arbres d'attaque rendent visibles les contre-mesures à fort effet de levier, capables de neutraliser plusieurs chemins d'attaque simultanément.
- Les méthodologies alternatives (PASTA, LINDDUN) offrent des perspectives complémentaires selon le contexte : PASTA pour ancrer l'analyse dans le risque métier, LINDDUN pour les risques spécifiques à la vie privée.
- Le threat modeling doit être révisé à chaque évolution architecturale significative, pas figé au lancement du projet.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
