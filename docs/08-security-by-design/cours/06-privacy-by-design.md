# Chapitre 6 — Privacy by design

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=08-security-by-design/slides/06-privacy-by-design.txt){ target=_blank }

## 1. Définition et origine

Le *privacy by design* (protection de la vie privée dès la conception) est un cadre formalisé par Ann Cavoukian, articulé autour de sept principes fondateurs, visant à intégrer la protection des données personnelles comme caractéristique intrinsèque d'un système, dès sa conception, plutôt que comme un ajout réglementaire tardif.

Ann Cavoukian, alors commissaire à l'information et à la protection de la vie privée de l'Ontario (Canada), a formalisé ce cadre à une époque où la protection de la vie privée était encore très largement traitée comme une question exclusivement juridique et réactive : un formulaire de consentement ajouté après coup, une politique de confidentialité rédigée pour satisfaire une obligation légale plutôt que pour refléter des choix de conception réellement protecteurs. L'apport principal de ce cadre a été de déplacer la question de la vie privée du seul terrain juridique vers le terrain de l'ingénierie et de la conception de systèmes, en affirmant qu'un système peut et doit être conçu, dès son architecture, pour minimiser structurellement les risques pour la vie privée des personnes concernées — une démarche directement parallèle à *security by design*, appliquée spécifiquement à la protection des données personnelles.

## 2. Les sept principes fondateurs

1. **Proactif, non réactif ; préventif, non correctif** : anticiper les risques pour la vie privée avant qu'un incident ne survienne.
2. **Protection de la vie privée par défaut** : les paramètres les plus protecteurs doivent être activés sans action requise de l'utilisateur.
3. **Protection intégrée à la conception** : composante structurelle du système, pas une fonctionnalité ajoutée après coup.
4. **Fonctionnalité intégrale (« somme positive », pas somme nulle)** : rechercher des solutions où sécurité/fonctionnalité et vie privée coexistent, plutôt que de présenter systématiquement un arbitrage l'un contre l'autre.
5. **Sécurité de bout en bout** : protection tout au long du cycle de vie de la donnée, de la collecte à la suppression.
6. **Visibilité et transparence** : les pratiques réelles doivent être vérifiables, pas seulement déclarées.
7. **Respect de la vie privée des utilisateurs** : conception centrée sur l'intérêt de la personne concernée, avec des interfaces et paramètres clairs et accessibles.

### Éclairage sur le principe de « somme positive »

Le quatrième principe mérite un développement particulier car il rectifie une opposition fréquemment présentée comme inévitable entre vie privée et fonctionnalité (ou entre vie privée et sécurité). L'exemple le plus souvent cité pour illustrer ce principe est celui de techniques permettant d'obtenir une information statistique utile (par exemple, une moyenne agrégée) sans jamais avoir besoin d'accéder aux données individuelles sous une forme directement identifiante (voir la confidentialité différentielle, section 5) : la finalité métier légitime est satisfaite sans sacrifier la protection des personnes concernées. Le principe invite le concepteur à chercher activement ce type de solution « somme positive » avant de conclure qu'un arbitrage strict entre fonctionnalité et vie privée est réellement inévitable dans un cas donné.

## 3. Traduction juridique : article 25 du RGPD

Le RGPD (Règlement Général sur la Protection des Données, Union européenne, largement pris en référence au-delà de sa portée juridique directe) reprend ce principe dans son article 25 : « protection des données dès la conception et par défaut » (*privacy by design and by default*), imposant concrètement au responsable de traitement de mettre en œuvre des mesures techniques et organisationnelles appropriées dès la conception d'un traitement de données personnelles.

L'article distingue explicitement deux volets complémentaires, qu'il est utile de ne pas confondre : la protection des données **dès la conception** (*by design*), qui impose d'intégrer des mesures de protection dans l'architecture même du système, et la protection des données **par défaut** (*by default*), qui impose que les paramètres appliqués par défaut, en l'absence de toute intervention de l'utilisateur, soient déjà les plus protecteurs (par exemple, un champ de profil optionnel visible uniquement par son propriétaire par défaut, plutôt que public par défaut avec une option de restriction que l'utilisateur devrait découvrir et activer lui-même). Cette distinction rejoint directement le principe de « valeurs par défaut sécurisées » vu au chapitre 3, appliqué spécifiquement au domaine de la protection des données personnelles.

## 4. Principes de minimisation

- **Minimisation de la collecte** : ne collecter que les données strictement nécessaires à la finalité poursuivie, pas « au cas où » elles seraient utiles plus tard.
- **Minimisation de la conservation** : définir une durée de conservation limitée et justifiée, avec suppression ou anonymisation effective au-delà.
- **Minimisation de l'accès** : appliquer le principe du moindre privilège (chapitre 3) spécifiquement aux données personnelles.
- **Minimisation de l'identifiabilité** : privilégier, quand la finalité le permet, des données pseudonymisées ou anonymisées plutôt que directement identifiantes.

### Exemple pédagogique de conception minimisatrice

Considérons, à titre d'illustration, une application fictive de gestion de présence en cours, « PresenceCampus ». Une conception naïve pourrait consister à enregistrer, pour chaque étudiant, l'horodatage précis de chaque entrée et sortie de salle, conservé indéfiniment, accessible à l'ensemble du personnel administratif. Une conception appliquant les principes de minimisation reviendrait plutôt à se poser systématiquement, pour chaque donnée envisagée, la question de sa nécessité réelle par rapport à la finalité déclarée (vérifier la présence à une séance donnée) :

| Donnée envisagée | Nécessaire à la finalité « vérifier la présence » ? | Choix de conception minimisateur |
|---|---|---|
| Présence/absence par séance (oui/non) | oui, directement | conservée |
| Horodatage précis à la seconde de l'entrée | non, une granularité par séance suffit | non collecté, ou agrégé au niveau de la séance |
| Historique complet conservé sans limite de durée | non, une durée limitée à l'année académique suffit généralement | suppression ou anonymisation automatique en fin d'année académique |
| Accès en lecture par l'ensemble du personnel administratif | non, seul le personnel pédagogique concerné par le cours en a besoin | accès restreint par le principe du moindre privilège |

## 5. Techniques de protection de la vie privée

| Technique | Principe |
|---|---|
| **Pseudonymisation** | remplacer un identifiant direct par un pseudonyme, la ré-identification restant possible avec une information additionnelle gardée séparément |
| **Anonymisation** | rendre la ré-identification impossible, y compris par recoupement avec d'autres sources (plus difficile à garantir réellement que ce que l'on croit souvent) |
| **Chiffrement** | protège la confidentialité des données au repos et en transit (lien direct avec le module *Cryptographie*) |
| **Confidentialité différentielle (differential privacy)** | ajout de bruit statistique calibré à des résultats d'agrégation pour empêcher la ré-identification d'un individu tout en préservant l'utilité statistique globale |
| **Contrôle d'accès et journalisation** | limiter et tracer qui accède à quelles données personnelles |

### Pourquoi l'anonymisation est plus difficile qu'il n'y paraît

Il est important d'insister, sur le plan pédagogique, sur la difficulté réelle d'obtenir une anonymisation effective et durable. Un jeu de données dont les identifiants directs (nom, numéro d'identification) ont simplement été retirés n'est pas nécessairement anonyme au sens strict : la combinaison de plusieurs attributs indirects, apparemment anodins pris séparément (par exemple, date de naissance approximative, code postal, profession), peut suffire à ré-identifier une personne de façon quasi unique au sein d'une population, en particulier si ce jeu de données est recoupé avec d'autres sources d'information publiques ou tierces. C'est pourquoi la pseudonymisation (qui conserve explicitement la possibilité théorique de ré-identification via une information additionnelle gardée séparément et protégée) et l'anonymisation véritable (qui doit résister à toute tentative de ré-identification, y compris par recoupement) sont deux notions à ne pas confondre : la réglementation applicable et les techniques de protection appropriées diffèrent sensiblement selon que l'on se situe dans l'un ou l'autre cas.

## 6. Analyse d'impact relative à la protection des données (AIPD/DPIA)

Pour un traitement de données présentant un risque élevé pour les droits des personnes, une analyse d'impact formelle est requise par de nombreuses réglementations : elle documente la finalité, la nécessité et la proportionnalité du traitement, évalue les risques pour les personnes concernées, et identifie les mesures pour les atténuer — une démarche directement apparentée à l'analyse de risque du module *Gouvernance et gestion des risques SI*, appliquée spécifiquement à la vie privée.

Une AIPD suit généralement une structure en plusieurs temps qu'il est utile de mettre en parallèle avec la démarche de threat modeling du chapitre 2 : description systématique du traitement envisagé et de ses finalités (comparable à « sur quoi travaillons-nous ? ») ; évaluation de la nécessité et de la proportionnalité du traitement au regard de sa finalité ; identification des risques pour les droits et libertés des personnes concernées (comparable à « qu'est-ce qui peut mal tourner ? ») ; définition des mesures envisagées pour traiter ces risques (comparable à « que faisons-nous à ce sujet ? »). Cette parenté méthodologique n'est pas fortuite : les deux démarches partagent la même logique d'anticipation structurée des risques avant la mise en œuvre effective d'un système ou d'un traitement.

## 7. LINDDUN : threat modeling orienté vie privée

Complément du chapitre 2 : LINDDUN structure l'identification de menaces spécifiques à la vie privée (Linkability, Identifiability, Non-repudiation, Detectability, Disclosure of information, Unawareness, Non-compliance), sur le même principe que STRIDE mais orienté vers les atteintes à la vie privée plutôt que vers les propriétés de sécurité classiques.

| Lettre | Menace (terme original) | Signification |
|---|---|---|
| **L** | Linkability | possibilité de relier deux éléments d'information ou deux actions à une même personne, sans que cela soit souhaité |
| **I** | Identifiability | possibilité d'identifier une personne à partir de données qui devraient rester non identifiantes |
| **N** | Non-repudiation | impossibilité pour une personne de nier avoir effectué une action, alors que l'anonymat ou le déni plausible serait légitimement attendu dans ce contexte |
| **D** | Detectability | possibilité de déduire l'existence d'une donnée ou d'un événement concernant une personne, même sans en connaître le contenu exact |
| **D** | Disclosure of information | divulgation non autorisée du contenu d'une donnée personnelle |
| **U** | Unawareness | la personne concernée n'a pas conscience de la collecte ou de l'usage fait de ses données |
| **N** | Non-compliance | le traitement ne respecte pas les obligations légales ou les engagements pris envers la personne concernée |

Il est instructif de noter que certaines catégories LINDDUN sont, en un sens, à l'opposé de catégories STRIDE portant un nom voisin : là où STRIDE cherche à garantir la non-répudiation (pouvoir prouver qu'une action a bien été effectuée par une personne donnée, propriété de sécurité recherchée), LINDDUN identifie au contraire la non-répudiation comme une **menace potentielle pour la vie privée** dans certains contextes (par exemple, un système de vote ou de consultation devant garantir l'anonymat de la personne ayant exprimé une opinion). Cette tension illustre bien pourquoi *security by design* et *privacy by design*, bien que complémentaires, ne poursuivent pas systématiquement les mêmes objectifs et peuvent, dans certains cas précis, entrer en tension l'un avec l'autre — un arbitrage de conception à traiter explicitement plutôt qu'à ignorer.

## À retenir

- Le privacy by design intègre la protection des données personnelles comme caractéristique par défaut d'un système, formalisé juridiquement par l'article 25 du RGPD (qui distingue explicitement la protection « dès la conception » et « par défaut ») et des dispositions équivalentes ailleurs.
- Le principe de « somme positive » invite à chercher activement des solutions où fonctionnalité et vie privée coexistent, plutôt que de présumer un arbitrage strict et inévitable entre les deux.
- La minimisation (collecte, conservation, accès, identifiabilité) est le principe opérationnel le plus directement actionnable en conception, comme l'illustre l'exemple d'une application de gestion de présence appliquant systématiquement le test de nécessité à chaque donnée envisagée.
- L'anonymisation véritable est plus difficile à garantir qu'il n'y paraît, en raison du risque de ré-identification par recoupement d'attributs indirects ; elle se distingue nettement de la pseudonymisation, réversible par nature.
- L'analyse d'impact relative à la protection des données (AIPD/DPIA) partage sa logique méthodologique avec le threat modeling du chapitre 2, appliquée spécifiquement aux risques pour les personnes concernées.
- LINDDUN transpose la logique de threat modeling (STRIDE) spécifiquement aux risques pour la vie privée, révélant au passage que certains objectifs de sécurité (comme la non-répudiation) peuvent, selon le contexte, entrer en tension avec les objectifs de protection de la vie privée plutôt que les renforcer systématiquement.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
