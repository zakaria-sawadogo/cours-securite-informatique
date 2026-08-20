# Chapitre 6 — Applications, enjeux et limites

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=07-technologie-blockchain/slides/06-applications-enjeux-limites.txt){ target=_blank }

## 1. Panorama des applications au-delà des cryptomonnaies

- **Finance décentralisée (DeFi)** : prêts, échanges, produits dérivés sans intermédiaire financier centralisé — avec les risques de sécurité vus au chapitre 5 et une volatilité/régulation encore incertaine.
- **Traçabilité de chaîne logistique** : suivi de provenance de produits (agroalimentaire, textile, minerais), où la blockchain garantit l'intégrité des enregistrements une fois saisis, sans garantir la véracité de la saisie initiale (problème de l'oracle, rappel chapitre 4).
- **Identité numérique décentralisée (self-sovereign identity)** : permettre à un individu de contrôler et prouver sélectivement des attributs de son identité sans dépendre d'un registre central unique.
- **Certification et notarisation** : preuve d'existence et d'horodatage d'un document à une date donnée (ancrage de l'empreinte du document sur une blockchain publique) — usage simple et robuste, sans manipuler de valeur financière.
- **Vote électronique** : cas d'usage souvent évoqué mais particulièrement délicat en pratique, car il combine des exigences difficilement conciliables (anonymat de l'électeur, vérifiabilité publique du résultat, résistance à la coercition) — objet de recherche active plutôt que de déploiements matures à grande échelle.

## 2. Trilemme de la blockchain (scalabilité, sécurité, décentralisation)

Les concepteurs de blockchains publiques font face à un compromis reconnu : il est difficile d'optimiser simultanément la **scalabilité** (nombre de transactions traitées par seconde), la **sécurité** et la **décentralisation**. Les architectures existantes privilégient généralement deux de ces trois propriétés au détriment de la troisième (ex. Bitcoin privilégie sécurité et décentralisation au prix d'une faible scalabilité ; certaines blockchains à permission privilégient scalabilité et sécurité au prix d'une décentralisation réduite).

### Approches de mise à l'échelle (aperçu)

- **Solutions de couche 2 (Layer 2)** : traiter la majorité des transactions hors chaîne principale, en n'ancrant sur la chaîne principale qu'un résumé périodique (ex. rollups sur Ethereum).
- **Sharding** : répartir l'état et le traitement entre plusieurs sous-réseaux (« shards ») traités en parallèle.

## 3. Enjeux réglementaires

- **Lutte contre le blanchiment (AML) et connaissance du client (KYC)** : tension entre pseudonymat des blockchains publiques et obligations réglementaires imposées aux plateformes d'échange.
- **Qualification juridique des cryptoactifs** : selon les juridictions, un jeton peut être qualifié différemment (monnaie, valeur mobilière, actif numérique sui generis), avec des conséquences fiscales et réglementaires très différentes.
- **Protection des consommateurs** : volatilité, risque de perte totale et irréversible en cas d'erreur de manipulation (perte de clé privée) ou d'escroquerie (absence de recours équivalent à un système bancaire traditionnel).
- **Encadrement progressif** : plusieurs juridictions ont mis en place ou développent des cadres réglementaires dédiés aux actifs numériques et aux prestataires de services associés ; la situation évolue rapidement et diffère fortement d'un pays à l'autre.

## 4. Limites techniques et environnementales

- Consommation énergétique significative pour les réseaux en preuve de travail (chapitre 3), bien que la transition vers la preuve d'enjeu réduise fortement cet impact pour les réseaux concernés.
- Irréversibilité des transactions : un avantage en termes de finalité, mais un risque important en cas d'erreur humaine ou de vol (pas de « bouton d'annulation »).
- Complexité d'usage pour l'utilisateur final (gestion de clés privées, absence de recours en cas de perte), un frein réel à l'adoption grand public.

## 5. Regard critique : quand la blockchain est-elle pertinente ?

Une grille de questions utile avant d'adopter une solution blockchain pour un projet donné : a-t-on réellement besoin de décentralisation, ou une base de données classique avec un tiers de confiance suffit-elle ? Plusieurs parties mutuellement méfiantes doivent-elles partager un registre sans intermédiaire ? L'immutabilité est-elle un avantage réel pour ce cas d'usage, ou une contrainte gênante (ex. droit à l'effacement en matière de données personnelles) ? De nombreux projets « blockchain » annoncés ces dernières années auraient pu être résolus plus simplement par une architecture centralisée classique.

## À retenir

- Les applications de la blockchain dépassent les cryptomonnaies mais restent souvent confrontées au problème de l'oracle et au trilemme scalabilité/sécurité/décentralisation.
- Le cadre réglementaire des cryptoactifs est encore en construction et varie fortement selon les juridictions.
- La pertinence d'une solution blockchain doit être évaluée par rapport à un besoin réel de décentralisation, pas adoptée par principe.
