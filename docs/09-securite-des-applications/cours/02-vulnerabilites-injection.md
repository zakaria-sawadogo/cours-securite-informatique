# Chapitre 2 — Vulnérabilités d'injection

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=09-securite-des-applications/slides/02-vulnerabilites-injection.txt){ target=_blank }

## 1. Principe général de l'injection

Une injection se produit lorsqu'une donnée fournie par un utilisateur est concaténée directement dans une commande, une requête ou un interpréteur, sans séparation stricte entre code et donnée, permettant à l'attaquant de modifier la structure ou la sémantique de la commande exécutée.

Ce mécanisme repose sur une confusion fondamentale : l'interpréteur cible (moteur SQL, shell, moteur de gabarit) ne dispose d'aucun moyen de distinguer, dans une chaîne de caractères reçue, ce qui relève de la structure voulue par le développeur et ce qui provient de l'utilisateur. Tant que code et donnée circulent mélangés dans le même flux textuel, cette ambiguïté reste exploitable. C'est précisément cette observation qui explique pourquoi la défense structurelle privilégiée, dans tous les cas étudiés ci-dessous, consiste à séparer physiquement le canal du code de celui de la donnée plutôt qu'à tenter de « nettoyer » la donnée après coup.

## 2. Injection SQL

### Exemple de code vulnérable

```sql
-- Requête construite par concaténation de chaînes
requete = "SELECT * FROM utilisateurs WHERE login = '" + login + "' AND motdepasse = '" + motdepasse + "'"
```

Si `login` reçoit la valeur `admin' -- `, la requête devient :

```sql
SELECT * FROM utilisateurs WHERE login = 'admin' -- ' AND motdepasse = '...'
```

Le `--` commente le reste de la requête (syntaxe SQL) : la vérification du mot de passe est neutralisée, permettant une connexion en tant qu'`admin` sans en connaître le mot de passe.

### Injection UNION-based

Une injection peut aussi être utilisée pour extraire des données d'autres tables via l'opérateur `UNION SELECT`, si le nombre et le type de colonnes sont devinés ou déduits par tâtonnement (technique pratiquée en TP1). Le principe général : l'attaquant ajoute une seconde requête `SELECT` combinée par `UNION` à la requête d'origine, dont le résultat vient s'afficher dans la même zone de la page que le résultat légitime attendu, révélant ainsi des données d'une table totalement différente (ex. une table de comptes administrateurs).

```sql
-- Requête légitime attendue : afficher le nom d'un produit à partir de son identifiant
SELECT nom, prix FROM produits WHERE id = 12

-- Entrée malveillante substituée à l'identifiant :
-- 12 UNION SELECT login, motdepasse FROM utilisateurs --
SELECT nom, prix FROM produits WHERE id = 12 UNION SELECT login, motdepasse FROM utilisateurs --
```

Pour que cette technique fonctionne, l'attaquant doit d'abord déterminer le nombre exact de colonnes de la requête d'origine (souvent par tâtonnement avec la clause `ORDER BY N`, en augmentant `N` jusqu'à provoquer une erreur), puis leur type compatible, avant de pouvoir aligner sa requête `UNION SELECT` sur cette structure.

### Injection en aveugle (blind SQL injection)

Lorsque l'application ne retourne pas directement le résultat de la requête mais seulement un comportement différent (page d'erreur, temps de réponse), un attaquant peut néanmoins extraire des données bit par bit en observant ces différences (injection booléenne ou temporelle).

- **Injection booléenne** : l'attaquant formule une condition qui rend la requête vraie ou fausse (ex. « le premier caractère du mot de passe de l'administrateur est-il supérieur à `m` ? ») et observe une différence de comportement (page affichée normalement contre page d'erreur, ou contenu légèrement différent), permettant de reconstruire l'information caractère par caractère par dichotomie.
- **Injection temporelle** : en l'absence de toute différence observable dans la réponse elle-même, l'attaquant insère une instruction provoquant un délai conditionnel côté base de données (ex. une fonction de mise en pause exécutée uniquement si la condition testée est vraie) et déduit la réponse du temps de réponse observé plutôt que du contenu de la page.

Ces deux variantes illustrent un point pédagogique important : l'absence de message d'erreur explicite révélant le contenu de la base de données **ne suffit pas** à protéger une application contre l'extraction de données via injection SQL ; seule l'élimination structurelle de la vulnérabilité (requêtes préparées) constitue une protection fiable.

### Contre-mesure de référence : requêtes préparées (paramétrées)

```
requete = "SELECT * FROM utilisateurs WHERE login = ? AND motdepasse = ?"
executer(requete, [login, motdepasse])
```

La donnée est transmise séparément de la structure de la requête au moteur de base de données, qui ne l'interprète jamais comme du code SQL, quel que soit son contenu. C'est la défense structurelle recommandée en priorité, à préférer systématiquement à un simple échappement de caractères spéciaux (plus fragile et sujet à contournement).

### Pièges fréquents avec les ORM (*Object-Relational Mapping*)

L'utilisation d'un ORM (mécanisme traduisant automatiquement des objets du langage applicatif en requêtes SQL) ne protège pas automatiquement contre l'injection SQL si le développeur contourne son mode d'utilisation sûr par défaut :

```python
# Vulnérable : construction manuelle d'une clause brute passée à l'ORM
resultats = Utilisateur.objects.raw(
    "SELECT * FROM utilisateurs WHERE nom = '" + nom_recherche + "'"
)

# Sûr : utilisation de l'API de requête paramétrée de l'ORM
resultats = Utilisateur.objects.filter(nom=nom_recherche)
```

La plupart des ORM modernes proposent, en usage standard, un paramétrage automatique des requêtes ; le risque réapparaît dès qu'un développeur revient à une requête « brute » construite par concaténation de chaînes pour un besoin non couvert par l'API de haut niveau — situation fréquente en pratique et donc à surveiller particulièrement en revue de code.

## 3. Injection de commande système

```c
char commande[100];
snprintf(commande, sizeof(commande), "ping -c 1 %s", entree_utilisateur);
system(commande);
```

Si `entree_utilisateur` vaut `8.8.8.8; rm -rf /donnees`, la commande système exécutée exécute en réalité deux commandes distinctes, la seconde étant totalement contrôlée par l'attaquant. **Contre-mesure** : éviter autant que possible d'appeler un interpréteur de commande shell avec des données utilisateur ; si indispensable, utiliser des API d'exécution de processus qui séparent explicitement la commande de ses arguments (sans passer par un shell interprétant les métacaractères), et valider strictement le format attendu (liste blanche).

```c
// Approche préférable : appel direct du programme, arguments séparés,
// sans passer par un interpréteur shell intermédiaire.
char *arguments[] = {"ping", "-c", "1", entree_utilisateur, NULL};
execv("/bin/ping", arguments);
```

Même dans ce second exemple, une validation stricte du format de `entree_utilisateur` (ex. une adresse IP ou un nom d'hôte conforme à une syntaxe attendue) reste nécessaire : l'absence d'interpréteur shell élimine l'enchaînement de commandes via `;`, `&&` ou `|`, mais ne garantit pas à elle seule que la donnée transmise au programme cible est inoffensive dans tous les cas (le programme appelé peut lui-même interpréter certains arguments de façon dangereuse selon son propre comportement).

## 4. Autres formes d'injection

| Type | Contexte | Exemple |
|---|---|---|
| **LDAP injection** | annuaires d'authentification | manipulation d'un filtre de recherche LDAP |
| **XPath injection** | requêtes sur documents XML | équivalent de l'injection SQL pour XPath |
| **NoSQL injection** | bases de données non relationnelles | injection d'opérateurs de requête dans un objet JSON mal validé |
| **Injection de gabarit côté serveur (SSTI)** | moteurs de templates web | exécution de code via une syntaxe de template non neutralisée |
| **Injection de code (command/eval injection)** | fonctions d'évaluation dynamique de code | passage direct d'une entrée utilisateur à une fonction `eval()` ou équivalente |

### Approfondissement : injection NoSQL

Contrairement à l'injection SQL, l'injection NoSQL n'exploite généralement pas une concaténation de chaînes de caractères mais une confusion de *type* de donnée transmise à une requête structurée en objet (JSON par exemple). Si une application construit sa requête à partir directement de l'objet JSON reçu du client, sans en valider la structure attendue, un attaquant peut substituer une valeur simple par un opérateur de requête :

```javascript
// Requête attendue par le développeur : { "motdepasse": "azerty123" }
// Entrée malveillante envoyée par l'attaquant :
{ "login": "admin", "motdepasse": { "$ne": null } }

// Si l'application transmet directement cet objet au moteur de requête
// sans vérifier que motdepasse est bien une chaîne de caractères simple,
// l'opérateur "$ne" (différent de) est interprété par le moteur NoSQL :
// la condition devient "le mot de passe est différent de null",
// vraie pour tout compte possédant un mot de passe, contournant ainsi
// totalement la vérification attendue.
```

**Contre-mesure** : valider strictement le *type* attendu de chaque champ (imposer explicitement une chaîne de caractères pour un mot de passe, rejeter tout objet ou tableau reçu là où une valeur simple est attendue), en complément du paramétrage lorsque le moteur NoSQL utilisé le propose.

### Approfondissement : injection de gabarit côté serveur (SSTI)

Les moteurs de gabarits (templates) permettent d'insérer dynamiquement du contenu dans un document (page HTML, e-mail) à partir d'une syntaxe dédiée. Une SSTI survient lorsqu'une entrée utilisateur est insérée non pas comme *donnée* affichée par le gabarit, mais comme *code de gabarit lui-même*, évalué par le moteur :

```
<!-- Vulnérable : le nom saisi par l'utilisateur est directement
     interprété comme faisant partie du gabarit -->
Bonjour {{ nom_utilisateur }}

<!-- Si nom_utilisateur vaut "{{ 7*7 }}", un moteur de gabarit vulnérable
     affichera "49" au lieu du texte littéral : la donnée a été évaluée
     comme du code de gabarit. Selon le moteur utilisé, cette même
     confusion peut être poussée jusqu'à l'exécution de code arbitraire
     côté serveur, un impact bien plus grave qu'un simple XSS. -->
```

La SSTI illustre bien le principe unificateur du chapitre 1 : la même confusion entre code et donnée, appliquée cette fois au contexte d'un moteur de gabarit plutôt qu'à un moteur SQL ou un shell système. **Contre-mesure** : ne jamais construire dynamiquement la chaîne de gabarit elle-même à partir d'une entrée utilisateur (seul le contenu inséré *dans* les emplacements prévus du gabarit doit provenir de l'utilisateur, jamais la structure du gabarit).

## 5. Principe de défense commun

Quelle que soit la variante, la défense structurelle privilégiée est la **séparation stricte entre code et donnée** (requêtes paramétrées, API d'exécution sans interprétation shell, désactivation des fonctionnalités dynamiques dangereuses des moteurs de template), complétée par une validation stricte des entrées en liste blanche et le principe du moindre privilège pour le compte utilisé par l'application (un compte de base de données en lecture seule limite l'impact même en cas d'injection réussie).

### Détection : outils et pratiques standards

- **Analyse statique (SAST)** : détection de motifs de concaténation de chaînes utilisés directement dans un appel de requête ou de commande, sans passer par une API paramétrée reconnue.
- **Analyse dynamique (DAST) et pentest** : envoi automatisé de charges de test connues (caractères spéciaux, séquences typiques d'injection) sur chaque point d'entrée, avec observation des réponses et des temps de réponse — principe déjà évoqué avec l'injection en aveugle.
- **Revue de code ciblée** : toute utilisation d'une API de requête « brute » (contournant le mode paramétré par défaut d'un ORM ou d'un pilote de base de données) mérite une attention particulière en revue de code, comme évoqué en section 2.
- **Pare-feu applicatif (WAF)** : mesure de défense complémentaire (et non substitutive) filtrant certains motifs d'attaque connus au niveau du trafic HTTP, utile en couche supplémentaire mais insuffisante seule car contournable par des variantes non répertoriées.

## À retenir

- L'injection résulte du mélange entre code et donnée non fiable dans un même flux interprété, quel que soit l'interpréteur cible (SQL, shell, LDAP, XPath, NoSQL, moteur de gabarit).
- Les requêtes préparées constituent la défense de référence contre l'injection SQL, structurellement plus robuste qu'un simple échappement ; ce même principe de paramétrage doit être recherché pour chaque variante d'injection rencontrée.
- L'injection en aveugle rappelle que l'absence de message d'erreur explicite ne suffit pas à protéger une application : seule l'élimination structurelle de la vulnérabilité est fiable.
- L'usage d'un ORM ne protège pas automatiquement contre l'injection SQL dès lors qu'un développeur revient à une requête construite manuellement par concaténation.
- Le principe de moindre privilège du compte applicatif limite l'impact même en cas de contournement des autres défenses ; un WAF constitue une couche complémentaire utile mais jamais suffisante seule.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
