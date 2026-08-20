# Chapitre 3 — Principes de conception sécurisée

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/03-principes-conception-securisee.txt){ target=_blank }

## 1. Moindre privilège (least privilege)

Chaque composant, utilisateur ou processus ne doit disposer que des droits strictement nécessaires à l'accomplissement de sa tâche, ni plus. Application concrète : un service applicatif se connectant à une base de données ne doit pas utiliser un compte disposant de droits d'administration complets si seules des opérations de lecture/écriture sur des tables spécifiques sont nécessaires. Ce principe limite l'impact d'une compromission (un composant compromis ne peut nuire qu'à hauteur de ses propres privilèges).

## 2. Défense en profondeur (defense in depth)

Superposer plusieurs couches de contrôles indépendantes, de sorte que la défaillance d'une seule couche ne compromette pas l'ensemble du système. Exemple pour une application web : pare-feu réseau, pare-feu applicatif (WAF), validation des entrées côté serveur, requêtes préparées contre l'injection SQL, chiffrement des données sensibles au repos, journalisation et détection d'anomalies. Aucune couche seule n'est infaillible ; leur combinaison réduit fortement la probabilité qu'une seule faille suffise à compromettre le système entier.

## 3. Échec sécurisé (fail-safe / fail-secure)

En cas d'erreur ou de défaillance, un système doit basculer vers l'état le plus sûr, pas vers un état permissif. Exemple : si un mécanisme de vérification d'autorisation échoue techniquement (exception, timeout), le comportement par défaut doit être de **refuser** l'accès, jamais de l'accorder par défaut. Une erreur de logique inversant ce principe (`if erreur: autoriser_acces()` au lieu de refuser) est une classe de bug de sécurité récurrente.

## 4. Médiation complète (complete mediation)

Chaque accès à une ressource protégée doit être vérifié systématiquement, à chaque tentative, sans mise en cache non maîtrisée d'une autorisation antérieure qui pourrait être devenue caduque (ex. droit révoqué entre-temps). Une vérification effectuée une seule fois « à l'entrée » puis considérée valide pour toute la suite d'une session peut devenir obsolète si les droits changent en cours de session.

## 5. Séparation des tâches (separation of duties)

Répartir des opérations sensibles entre plusieurs personnes ou composants distincts, de sorte qu'aucune entité unique ne puisse, seule, réaliser une action critique de bout en bout sans contrôle croisé. Exemple organisationnel : la personne qui valide un paiement ne doit pas être la même que celle qui l'initie. Exemple technique : séparer les clés de signature de code des systèmes de build automatisés accessibles à de nombreux développeurs.

## 6. Simplicité de conception (economy of mechanism)

Un système simple est plus facile à analyser, auditer et prouver correct qu'un système complexe. La complexité est l'ennemie de la sécurité : elle multiplie les cas limites, les interactions imprévues entre composants et la surface d'attaque effective. Ce principe justifie de préférer des architectures et des dépendances minimales à des solutions sur-conçues « au cas où ».

## 7. Conception ouverte (open design) — rappel du principe de Kerckhoffs

Rappel direct du module *Cryptographie* : la sécurité d'un système ne doit pas reposer sur le secret de sa conception, mais sur des mécanismes robustes même connus de l'attaquant (clés, secrets d'authentification), le design lui-même pouvant être examiné publiquement.

## 8. Minimisation de la surface d'attaque

Réduire le nombre de points d'entrée exposés (fonctionnalités, ports réseau, API, dépendances tierces) au strict nécessaire. Chaque fonctionnalité, endpoint ou dépendance supplémentaire est une surface d'attaque potentielle additionnelle à sécuriser et maintenir — un principe qui rejoint la minimisation de complexité du point précédent.

## 9. Valeurs par défaut sécurisées (secure by default)

Un système livré ou installé doit être sécurisé « dès la sortie de la boîte », sans que l'utilisateur ait à activer explicitement les protections de base (ex. chiffrement activé par défaut, comptes par défaut désactivés, ports non essentiels fermés par défaut). Rappel du chapitre 1 : la sécurité facile et par défaut est plus efficace en pratique que la sécurité techniquement supérieure mais optionnelle.

## À retenir

- Ces principes (Saltzer et Schroeder, et compléments ultérieurs) forment un socle de conception réutilisable indépendamment de la technologie employée.
- Ils se combinent : la défense en profondeur suppose l'application cohérente du moindre privilège à chaque couche ; l'échec sécurisé suppose une médiation complète.
- Aucun principe pris isolément ne suffit ; c'est leur application systématique et cohérente à l'échelle d'un système qui constitue une véritable démarche *security by design*.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
