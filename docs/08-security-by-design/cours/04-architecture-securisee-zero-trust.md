# Chapitre 4 — Architecture sécurisée : segmentation et zero trust

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/04-architecture-securisee-zero-trust.txt){ target=_blank }

## 1. Le modèle périmétrique traditionnel et ses limites

Le modèle de sécurité « château fort » repose sur un périmètre (pare-feu) séparant un réseau interne considéré de confiance d'un extérieur considéré hostile. Ce modèle, longtemps dominant, s'avère fragile face à des menaces qui n'ont plus besoin de franchir frontalement le périmètre : télétravail massif, usage du Cloud (données et applications hors du périmètre physique traditionnel), compromission initiale par hameçonnage d'un poste interne, prestataires tiers disposant d'accès internes.

Trois évolutions structurelles ont particulièrement affaibli la pertinence du modèle périmétrique au fil du temps :

- **La dissolution du périmètre physique** : lorsque les applications et les données d'une organisation sont hébergées chez un ou plusieurs fournisseurs Cloud, et que les utilisateurs y accèdent depuis des réseaux variés (domicile, déplacement, réseaux publics), la notion même de « frontière du réseau interne » perd une grande partie de son sens opérationnel : il n'existe plus un point unique par lequel canaliser et filtrer l'ensemble du trafic.
- **La sophistication des attaques d'ingénierie sociale** : un attaquant qui parvient à obtenir les identifiants d'un utilisateur légitime, par hameçonnage ciblé, contourne intégralement un pare-feu périmétrique aussi robuste soit-il — le pare-feu ne distingue pas une connexion légitime d'une connexion illégitime effectuée avec des identifiants volés mais valides.
- **La menace interne et les tiers de confiance** : un modèle périmétrique accorde implicitement une confiance élevée à tout ce qui se trouve « à l'intérieur », y compris des prestataires externes disposant d'accès distants, des équipements personnels connectés au réseau interne, ou des employés dont le comportement peut, volontairement ou non (poste compromis à leur insu), devenir une source de risque.

## 2. Segmentation réseau

Avant même le concept de zero trust, la segmentation réduit l'impact d'une compromission en divisant le réseau en zones distinctes avec un filtrage strict entre elles (rappel du module *Audit organisation et technique*, chapitre 4) : DMZ pour les services exposés, réseau interne cloisonné par fonction ou sensibilité, réseau d'administration isolé du réseau utilisateur. Une segmentation fine limite la **propagation latérale (lateral movement)** d'un attaquant ayant compromis un premier point d'entrée.

La propagation latérale décrit le scénario où un attaquant, après avoir obtenu un accès initial limité (par exemple, un poste utilisateur compromis par hameçonnage), cherche ensuite à étendre progressivement son emprise à d'autres systèmes du réseau interne — serveurs de fichiers, contrôleurs de domaine, bases de données — en exploitant l'absence de cloisonnement entre segments. Une segmentation bien conçue transforme un incident potentiellement catastrophique (compromission de l'ensemble du système d'information à partir d'un unique poste utilisateur) en un incident circonscrit (compromission limitée au segment initialement atteint), ce qui réduit considérablement le temps et les moyens nécessaires pour contenir et remédier à l'incident.

## 3. Principes du modèle Zero Trust

Formalisé notamment par le NIST (SP 800-207), le modèle zero trust part du principe qu'**aucune confiance implicite ne doit être accordée du simple fait qu'une requête provient du réseau interne** : chaque accès doit être authentifié, autorisé et vérifié en continu, indépendamment de sa localisation réseau d'origine.

Le document NIST SP 800-207 définit sept principes directeurs pour une architecture zero trust, dont les idées maîtresses peuvent se résumer ainsi : toute ressource de données ou de calcul est considérée comme une ressource à protéger indépendamment de son emplacement réseau ; toute communication doit être sécurisée quelle que soit la localisation du réseau ; l'accès à chaque ressource individuelle est accordé par session, sur la base d'une décision dynamique ; l'accès est déterminé par une politique tenant compte de multiples attributs (identité, application, état de l'appareil) et non uniquement par l'emplacement réseau ; l'organisation surveille et mesure en continu l'intégrité et la posture de sécurité de tous les actifs, propres ou associés ; l'authentification et l'autorisation sont strictes et dynamiques, appliquées avant que tout accès ne soit accordé.

### Principes clés

- **Vérification explicite** : authentifier et autoriser systématiquement, en s'appuyant sur toutes les données disponibles (identité, appareil, localisation, comportement).
- **Moindre privilège appliqué dynamiquement** : accès juste-à-temps et juste-suffisant (*just-in-time*, *just-enough-access*), plutôt que des droits larges accordés une fois pour toutes.
- **Présomption de compromission** (*assume breach*) : concevoir le système comme si un attaquant était déjà présent quelque part dans l'environnement, en minimisant le rayon d'action possible (segmentation fine, micro-segmentation) et en maximisant la détection.

Ces trois principes, mis ensemble, changent fondamentalement la question que pose une architecture zero trust par rapport à un modèle périmétrique. Le modèle périmétrique pose implicitement la question : « cette requête provient-elle de l'intérieur du réseau de confiance ? ». Le modèle zero trust pose systématiquement une question différente, réévaluée à chaque accès : « cette identité précise, sur cet appareil précis, dans ce contexte précis, est-elle autorisée à effectuer cette action précise sur cette ressource précise, maintenant ? ».

## 4. Composants typiques d'une architecture zero trust

| Composant | Rôle |
|---|---|
| **Gestion des identités (IAM)** | authentification forte (MFA), gestion centralisée des identités et des droits |
| **Politique d'accès contextuelle** | décision d'accès basée sur de multiples signaux (identité, posture de l'appareil, localisation, sensibilité de la ressource), pas uniquement sur l'appartenance réseau |
| **Micro-segmentation** | contrôle fin des flux, y compris entre charges de travail internes (pas seulement à la frontière du réseau) |
| **Chiffrement systématique** | chiffrement des flux, y compris en interne, sans supposer un réseau interne intrinsèquement sûr |
| **Supervision continue** | journalisation et détection comportementale permettant de révoquer un accès en cours de session si un comportement anormal est détecté |

### Exemple illustratif d'une politique d'accès contextuelle

Le fragment ci-dessous illustre, de façon simplifiée et pédagogique, le type de règle qu'un moteur de politique d'accès zero trust peut évaluer avant d'accorder un accès à une ressource sensible, en combinant plusieurs signaux plutôt qu'une seule condition d'appartenance réseau :

```yaml
# Exemple pédagogique simplifié de politique d'accès contextuelle
politique:
  ressource: "application-notes-etudiants"
  regles_acces:
    - condition:
        identite_authentifiee: true
        facteur_authentification: "MFA"
        appareil_conforme: true        # ex. chiffrement disque actif, correctifs à jour
        role_utilisateur: ["enseignant", "administration"]
      decision: "autoriser"
      duree_session_max: "8h"
    - condition:
        localisation_reseau: "inconnue_ou_a_risque"
        role_utilisateur: ["enseignant", "administration"]
      decision: "autoriser_avec_verification_additionnelle"
      verification_additionnelle: "reauthentification_MFA"
    - condition: "default"
      decision: "refuser"
```

Ce fragment illustre un point essentiel du zero trust : la même identité (un enseignant authentifié) peut se voir accorder un accès direct depuis un contexte jugé habituel, ou au contraire soumise à une vérification additionnelle depuis un contexte jugé plus risqué (nouvel appareil, localisation inhabituelle) — la décision n'est jamais figée une fois pour toutes, elle dépend du contexte évalué à chaque tentative d'accès.

## 5. Zero trust n'est pas un produit unique

Zero trust est une architecture et une philosophie de conception, pas un produit acheté sur étagère : sa mise en œuvre combine plusieurs briques technologiques existantes (IAM, micro-segmentation, chiffrement, supervision) orchestrées selon les principes ci-dessus, et s'accompagne nécessairement d'une transformation organisationnelle (revue des processus d'accès, culture de vérification continue).

Cette précision est importante car le terme « zero trust » a fait l'objet d'un usage marketing important de la part de nombreux éditeurs de solutions de sécurité, chacun présentant son propre produit comme une solution zero trust complète. Un achat de produit, à lui seul, ne constitue pas une architecture zero trust : celle-ci résulte d'une démarche de conception cohérente, articulant plusieurs briques technologiques autour de politiques d'accès explicites et d'une gouvernance des identités robuste, bien plus que de l'installation d'un outil particulier.

## 6. Migration progressive

Une migration complète vers zero trust est rarement un projet « big bang » : elle se conduit généralement de façon incrémentale, en priorisant les actifs les plus critiques (identifiés par la démarche de gestion des risques du module *Gouvernance*), avec des architectures hybrides transitoires combinant éléments périmétriques existants et contrôles zero trust nouveaux.

Une démarche de migration progressive typique suit généralement les grandes étapes suivantes : (1) inventaire des actifs et cartographie des flux existants, condition préalable indispensable puisqu'on ne peut protéger correctement ce que l'on ne connaît pas ; (2) renforcement de la gestion des identités (authentification forte généralisée, centralisation des identités) comme fondation, car la vérification explicite de l'identité est le socle sur lequel reposent tous les autres contrôles zero trust ; (3) déploiement progressif de la micro-segmentation, en commençant par les segments hébergeant les actifs les plus critiques ou les plus exposés ; (4) mise en place de politiques d'accès contextuelles de plus en plus fines, au fur et à mesure que la maturité de supervision augmente ; (5) extension progressive du périmètre couvert, jusqu'à réduire autant que possible la dépendance résiduelle au modèle périmétrique traditionnel. Cette progressivité permet à l'organisation d'apprendre et d'ajuster ses politiques d'accès sans provoquer d'interruption massive de service, tout en démontrant une valeur mesurable dès les premières étapes.

## À retenir

- Le modèle périmétrique traditionnel suppose une confiance implicite au réseau interne, une hypothèse de moins en moins tenable (télétravail, Cloud, prestataires tiers, hameçonnage contournant frontalement le pare-feu).
- La segmentation réseau, préalable historique au zero trust, limite déjà fortement la propagation latérale d'un attaquant ayant obtenu un accès initial.
- Le zero trust (formalisé notamment par le NIST SP 800-207) remplace la confiance implicite par une vérification systématique et continue de chaque accès, fondée sur de multiples signaux contextuels et non sur la seule provenance réseau.
- La mise en œuvre combine des briques technologiques existantes (IAM, micro-segmentation, chiffrement, supervision) et se conduit généralement de façon progressive et priorisée, en commençant typiquement par le renforcement de la gestion des identités.
- Zero trust est une démarche de conception et de gouvernance, pas un produit unique acheté sur étagère, malgré un usage marketing fréquent du terme par les éditeurs de solutions.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
