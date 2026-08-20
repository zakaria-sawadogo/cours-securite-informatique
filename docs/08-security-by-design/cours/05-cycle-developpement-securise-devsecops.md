# Chapitre 5 — Cycle de développement sécurisé (DevSecOps)

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/05-cycle-developpement-securise-devsecops.txt){ target=_blank }

## 1. Le cycle de développement sécurisé (Secure SDLC)

Intégrer la sécurité à chaque phase du cycle de vie du développement logiciel, plutôt que de la traiter comme une étape de validation finale isolée :

| Phase | Activités de sécurité |
|---|---|
| Exigences | exigences de sécurité explicites, classification de la sensibilité des données traitées |
| Conception | threat modeling (chapitre 2), revue d'architecture |
| Développement | bonnes pratiques de codage sécurisé, revue de code, analyse statique (SAST) |
| Tests | tests de sécurité automatisés, analyse dynamique (DAST), tests d'intrusion |
| Déploiement | durcissement de la configuration, gestion sécurisée des secrets |
| Exploitation | supervision, gestion des correctifs, réponse à incident |

Un Secure SDLC bien conçu se distingue d'un cycle de développement classique auquel on aurait simplement ajouté une phase de test de sécurité en fin de projet : dans un Secure SDLC, chaque phase produit des artefacts qui alimentent la suivante (les exigences de sécurité orientent le threat modeling, dont les contre-mesures identifiées deviennent des tâches de développement, dont la couverture est vérifiée par les tests). Cette continuité évite l'écueil fréquent où l'équipe sécurité découvre, au moment des tests finaux, des choix d'architecture profondément ancrés qu'il est devenu trop coûteux de remettre en cause — situation qui ramène directement à la discussion du chapitre 1 sur le coût croissant de la correction tardive.

## 2. « Shift left » : déplacer la sécurité en amont

Le principe du *shift left* consiste à déplacer les activités de sécurité le plus tôt possible dans le cycle de développement (« vers la gauche » d'une frise chronologique), en cohérence avec l'observation du chapitre 1 : plus une faille est détectée tôt, moins elle coûte à corriger.

Le shift left ne signifie pas seulement « exécuter des outils de scan plus tôt » ; il implique un changement de responsabilité : plutôt que la sécurité soit uniquement l'affaire d'une équipe spécialisée intervenant en fin de cycle, chaque développeur devient acteur de la sécurité du code qu'il produit, au moment même où il le produit — retour d'information immédiat (par exemple, un avertissement de l'éditeur de code ou du pipeline de build dès l'introduction d'un motif de code vulnérable) plutôt que différé de plusieurs semaines ou mois lors d'un audit externe.

## 3. DevSecOps : intégrer la sécurité dans DevOps

DevOps vise à accélérer et fiabiliser le cycle développement-déploiement par l'automatisation (intégration continue, déploiement continu — CI/CD) et une collaboration étroite entre équipes de développement et d'exploitation. **DevSecOps** étend cette approche en intégrant la sécurité comme responsabilité partagée et automatisée tout au long du pipeline, plutôt que comme une porte de validation manuelle et tardive gérée par une équipe séparée.

### Contrôles de sécurité automatisables dans un pipeline CI/CD

| Étape du pipeline | Contrôle de sécurité |
|---|---|
| Commit / pre-commit | détection de secrets codés en dur (clés API, mots de passe) avant qu'ils n'entrent dans l'historique du dépôt |
| Build | analyse statique du code (SAST) |
| Dépendances | analyse de composition logicielle (SCA) : vulnérabilités connues dans les bibliothèques tierces utilisées |
| Tests | tests de sécurité automatisés (cas de test dérivés du threat modeling) |
| Déploiement en environnement de test | analyse dynamique (DAST) contre l'application en fonctionnement |
| Image de conteneur | analyse de vulnérabilités de l'image avant publication |
| Infrastructure as Code | analyse de configuration (détection de règles de pare-feu trop permissives, de stockage mal configuré) avant provisionnement |

### Exemple pédagogique d'extrait de pipeline CI/CD

L'extrait suivant illustre, de façon simplifiée, comment plusieurs de ces contrôles peuvent être intégrés comme étapes successives d'un pipeline d'intégration continue :

```yaml
# Extrait pédagogique simplifié d'un pipeline CI/CD (syntaxe générique)
stages:
  - detection_secrets
  - analyse_statique
  - analyse_dependances
  - tests
  - analyse_image_conteneur
  - analyse_infrastructure_as_code
  - deploiement

detection_secrets:
  stage: detection_secrets
  script:
    - scanner-secrets --chemin ./src --echec-si-trouve

analyse_statique:
  stage: analyse_statique
  script:
    - outil-sast --chemin ./src --seuil-echec "critique,eleve"

analyse_dependances:
  stage: analyse_dependances
  script:
    - outil-sca --fichier-dependances requirements.txt --seuil-echec "critique"

tests:
  stage: tests
  script:
    - executer-tests-unitaires
    - executer-tests-securite-derives-threat-modeling

analyse_image_conteneur:
  stage: analyse_image_conteneur
  script:
    - construire-image -t app:${CI_COMMIT_SHA}
    - scanner-image app:${CI_COMMIT_SHA} --seuil-echec "critique"

analyse_infrastructure_as_code:
  stage: analyse_infrastructure_as_code
  script:
    - scanner-iac --chemin ./infrastructure --seuil-echec "critique,eleve"

deploiement:
  stage: deploiement
  script:
    - deployer-environnement-test
  when: on_success   # ne se déclenche que si toutes les étapes précédentes ont réussi
```

Ce type de pipeline matérialise concrètement le principe de shift left : une vulnérabilité critique introduite dans une dépendance, un secret oublié dans un fichier de configuration, ou une règle de pare-feu excessivement permissive dans un fichier d'infrastructure as code sont détectés automatiquement avant même que le code n'atteigne un environnement partagé, plutôt que découverts a posteriori par un audit ou, pire, par un incident réel en production.

## 4. Gestion des secrets

Les identifiants, clés d'API et certificats ne doivent jamais être codés en dur dans le code source ou versionnés dans un dépôt Git (même privé — un dépôt peut devenir public par erreur, ou son historique complet reste accessible à quiconque y a eu accès). Bonnes pratiques : gestionnaires de secrets dédiés (Vault et équivalents), variables d'environnement injectées au déploiement, rotation régulière des secrets, scan automatique des dépôts pour détecter des secrets déjà exposés.

```python
# Anti-pattern à éviter : secret codé en dur directement dans le code source
CLE_API_PAIEMENT = "<CLE_SECRETE_EN_CLAIR_A_NE_JAMAIS_VERSIONNER>"

# Bonne pratique : le secret est récupéré depuis un gestionnaire de secrets externe,
# jamais présent en clair dans le code source ni dans l'historique du dépôt
CLE_API_PAIEMENT = gestionnaire_secrets.recuperer("cle_api_paiement")
```

Un point souvent sous-estimé par les équipes de développement est la persistance d'un secret compromis dans l'**historique** d'un dépôt Git : même si un secret codé en dur est supprimé dans un commit ultérieur, il demeure récupérable par quiconque a accès à l'historique complet du dépôt (`git log`, clone antérieur, fork existant), tant qu'une réécriture explicite de l'historique n'a pas été effectuée — opération elle-même risquée sur un dépôt partagé. La bonne pratique en cas de secret exposé n'est donc jamais de se contenter de le supprimer dans un nouveau commit, mais de le considérer comme définitivement compromis et de procéder immédiatement à sa **révocation et son remplacement** (rotation), indépendamment du nettoyage de l'historique.

## 5. Gestion des vulnérabilités des dépendances tierces

La majorité du code d'une application moderne provient de dépendances tierces (bibliothèques open source). Une vulnérabilité découverte dans une dépendance largement utilisée peut affecter un très grand nombre d'applications simultanément (illustré par des incidents majeurs affectant des bibliothèques de journalisation ou de compression largement répandues). L'analyse de composition logicielle (SCA) automatisée en continu, couplée à une politique de mise à jour réactive, est devenue une pratique de sécurité de base incontournable.

Deux dimensions complémentaires structurent une gestion mature des vulnérabilités de dépendances : la **détection** (scan automatique et continu des dépendances directes et transitives — une dépendance transitive étant une dépendance de dépendance, souvent bien plus nombreuse et moins visible que les dépendances directes déclarées par l'équipe de développement elle-même), et la **réactivité** (délai entre la publication d'un correctif pour une vulnérabilité connue et son application effective dans le système). Un inventaire précis et à jour des dépendances utilisées (parfois appelé nomenclature logicielle ou *Software Bill of Materials*, SBOM) facilite considérablement cette seconde dimension : lorsqu'une nouvelle vulnérabilité est publiée pour un composant donné, disposer d'un inventaire exhaustif permet d'identifier immédiatement l'ensemble des systèmes concernés, plutôt que de devoir mener une investigation manuelle dans l'urgence.

## 6. Culture et organisation

DevSecOps est autant une question de culture que d'outillage : formation des développeurs à la sécurité (« security champions » au sein des équipes de développement), responsabilité partagée plutôt que reportée entièrement sur une équipe sécurité séparée en fin de cycle, et retour d'information rapide (un développeur informé d'une faille immédiatement après son introduction la corrige bien plus efficacement qu'informé plusieurs mois plus tard lors d'un audit externe).

Le modèle des « security champions » mérite d'être détaillé : il s'agit de désigner, au sein de chaque équipe de développement, une ou plusieurs personnes qui, sans être des experts sécurité à temps plein, reçoivent une formation approfondie sur les enjeux de sécurité pertinents pour leur périmètre, et servent de relais de proximité entre l'équipe sécurité centrale et les développeurs au quotidien. Ce modèle répond à un problème d'échelle réel : une petite équipe sécurité centrale ne peut matériellement pas réviser en détail chaque ligne de code produite par l'ensemble des équipes de développement d'une organisation de taille significative ; les security champions démultiplient la capacité de vigilance sécurité sans nécessiter que chaque développeur devienne un expert sécurité généraliste.

### Repère : maturité DevSecOps et référentiels d'auto-évaluation

Pour situer la maturité d'une organisation en matière d'intégration de la sécurité dans le développement logiciel, plusieurs référentiels reconnus peuvent être mobilisés, tel **OWASP SAMM** (Software Assurance Maturity Model), qui structure l'évaluation autour de plusieurs fonctions métier (gouvernance, conception, implémentation, vérification, opérations) et de niveaux de maturité croissants pour chacune. L'intérêt pédagogique d'un tel référentiel n'est pas tant d'obtenir un score chiffré que de fournir une grille de lecture structurée permettant à une organisation d'identifier ses pratiques déjà matures et ses angles morts (par exemple, une organisation peut avoir un processus de threat modeling très mature en phase de conception tout en négligeant presque totalement la gestion des vulnérabilités en phase d'exploitation).

## À retenir

- Le Secure SDLC intègre des activités de sécurité à chaque phase du développement, pas uniquement en validation finale, avec des artefacts qui se transmettent d'une phase à l'autre (exigences → threat modeling → tâches de développement → tests).
- Le shift left implique un changement de responsabilité, pas seulement un changement de calendrier : chaque développeur devient acteur de la sécurité du code qu'il produit, avec un retour d'information rapide plutôt que différé.
- DevSecOps automatise ces contrôles directement dans le pipeline CI/CD (détection de secrets, SAST, SCA, tests dérivés du threat modeling, DAST, scan d'image de conteneur, scan d'infrastructure as code).
- La gestion des secrets impose la rotation immédiate d'un secret exposé, indépendamment du nettoyage de l'historique du dépôt, qui à lui seul ne suffit pas à neutraliser la compromission.
- La gestion des dépendances tierces vulnérables repose sur deux dimensions complémentaires : la détection continue (y compris des dépendances transitives) et la réactivité de mise à jour, facilitées par un inventaire précis des composants utilisés (SBOM).
- DevSecOps est autant une question de culture que d'outillage : le modèle des security champions démultiplie la capacité de vigilance sécurité sans exiger que chaque développeur devienne expert sécurité généraliste ; des référentiels comme OWASP SAMM aident à situer la maturité d'une organisation.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
