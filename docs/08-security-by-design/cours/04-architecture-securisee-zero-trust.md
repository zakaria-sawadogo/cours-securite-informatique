# Chapitre 4 — Architecture sécurisée : segmentation et zero trust

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/04-architecture-securisee-zero-trust.txt){ target=_blank }

## 1. Le modèle périmétrique traditionnel et ses limites

Le modèle de sécurité « château fort » repose sur un périmètre (pare-feu) séparant un réseau interne considéré de confiance d'un extérieur considéré hostile. Ce modèle, longtemps dominant, s'avère fragile face à des menaces qui n'ont plus besoin de franchir frontalement le périmètre : télétravail massif, usage du Cloud (données et applications hors du périmètre physique traditionnel), compromission initiale par hameçonnage d'un poste interne, prestataires tiers disposant d'accès internes.

## 2. Segmentation réseau

Avant même le concept de zero trust, la segmentation réduit l'impact d'une compromission en divisant le réseau en zones distinctes avec un filtrage strict entre elles (rappel du module *Audit organisation et technique*, chapitre 4) : DMZ pour les services exposés, réseau interne cloisonné par fonction ou sensibilité, réseau d'administration isolé du réseau utilisateur. Une segmentation fine limite la **propagation latérale (lateral movement)** d'un attaquant ayant compromis un premier point d'entrée.

## 3. Principes du modèle Zero Trust

Formalisé notamment par le NIST (SP 800-207), le modèle zero trust part du principe qu'**aucune confiance implicite ne doit être accordée du simple fait qu'une requête provient du réseau interne** : chaque accès doit être authentifié, autorisé et vérifié en continu, indépendamment de sa localisation réseau d'origine.

### Principes clés

- **Vérification explicite** : authentifier et autoriser systématiquement, en s'appuyant sur toutes les données disponibles (identité, appareil, localisation, comportement).
- **Moindre privilège appliqué dynamiquement** : accès juste-à-temps et juste-suffisant (*just-in-time*, *just-enough-access*), plutôt que des droits larges accordés une fois pour toutes.
- **Présomption de compromission** (*assume breach*) : concevoir le système comme si un attaquant était déjà présent quelque part dans l'environnement, en minimisant le rayon d'action possible (segmentation fine, micro-segmentation) et en maximisant la détection.

## 4. Composants typiques d'une architecture zero trust

| Composant | Rôle |
|---|---|
| **Gestion des identités (IAM)** | authentification forte (MFA), gestion centralisée des identités et des droits |
| **Politique d'accès contextuelle** | décision d'accès basée sur de multiples signaux (identité, posture de l'appareil, localisation, sensibilité de la ressource), pas uniquement sur l'appartenance réseau |
| **Micro-segmentation** | contrôle fin des flux, y compris entre charges de travail internes (pas seulement à la frontière du réseau) |
| **Chiffrement systématique** | chiffrement des flux, y compris en interne, sans supposer un réseau interne intrinsèquement sûr |
| **Supervision continue** | journalisation et détection comportementale permettant de révoquer un accès en cours de session si un comportement anormal est détecté |

## 5. Zero trust n'est pas un produit unique

Zero trust est une architecture et une philosophie de conception, pas un produit acheté sur étagère : sa mise en œuvre combine plusieurs briques technologiques existantes (IAM, micro-segmentation, chiffrement, supervision) orchestrées selon les principes ci-dessus, et s'accompagne nécessairement d'une transformation organisationnelle (revue des processus d'accès, culture de vérification continue).

## 6. Migration progressive

Une migration complète vers zero trust est rarement un projet « big bang » : elle se conduit généralement de façon incrémentale, en priorisant les actifs les plus critiques (identifiés par la démarche de gestion des risques du module *Gouvernance*), avec des architectures hybrides transitoires combinant éléments périmétriques existants et contrôles zero trust nouveaux.

## À retenir

- Le modèle périmétrique traditionnel suppose une confiance implicite au réseau interne, une hypothèse de moins en moins tenable (télétravail, Cloud, menaces internes).
- Le zero trust remplace cette confiance implicite par une vérification systématique et continue de chaque accès, quelle que soit sa provenance.
- La mise en œuvre combine des briques technologiques existantes (IAM, micro-segmentation, chiffrement, supervision) et se conduit généralement de façon progressive et priorisée.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
