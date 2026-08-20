# Chapitre 3 — Formats d'échange : JSON, XML et SOAP

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=10-service-web/slides/03-formats-echange-json-xml-soap.txt){ target=_blank }

## 1. Le rôle d'un format d'échange

Le format d'échange définit la façon dont les données transportées par une requête ou une réponse HTTP (chapitre 1) sont structurées et sérialisées. Le choix du format conditionne la lisibilité, la taille des messages, la richesse de la validation possible, et l'écosystème d'outils disponible. Ce chapitre compare les trois formats les plus rencontrés dans les services web : **JSON**, **XML**, et le protocole historique **SOAP**, qui s'appuie sur XML.

## 2. JSON

**JSON** (*JavaScript Object Notation*) est aujourd'hui le format dominant des API REST modernes (chapitre 2), pour sa légèreté syntaxique et sa correspondance directe avec les structures de données natives de la plupart des langages de programmation (objets/dictionnaires, tableaux/listes, chaînes, nombres, booléens, valeur nulle).

```json
{
  "id": 42,
  "nom": "Awa Traoré",
  "actif": true,
  "role": "etudiant",
  "cours_suivis": ["Service web", "Cryptographie"],
  "adresse": {
    "ville": "Ouagadougou",
    "pays": "Burkina Faso"
  }
}
```

Avantages structurants de JSON par rapport à XML :

- **Concision** : absence de balises fermantes redondantes, ce qui réduit la taille des messages.
- **Correspondance directe** avec les types natifs des langages de programmation les plus courants, ce qui simplifie la sérialisation/désérialisation.
- **Lisibilité** pour un humain, tout en restant facile à analyser (*parser*) pour une machine.

### JSON Schema

**JSON Schema** est une spécification permettant de décrire formellement la structure attendue d'un document JSON (types, champs obligatoires, contraintes de valeur), utile pour valider automatiquement une requête ou documenter précisément un contrat d'API (approfondi au chapitre 5, où OpenAPI s'appuie directement sur ce mécanisme).

```json
{
  "type": "object",
  "required": ["id", "nom"],
  "properties": {
    "id": {"type": "integer"},
    "nom": {"type": "string", "minLength": 1},
    "role": {"type": "string", "enum": ["etudiant", "enseignant", "administrateur"]}
  }
}
```

## 3. XML

**XML** (*eXtensible Markup Language*) est un format à balises, plus verbeux que JSON, mais offrant des capacités de validation et de description structurelle historiquement plus riches, ainsi qu'un support natif de mécanismes absents de JSON comme les espaces de noms (*namespaces*) et les commentaires.

```xml
<utilisateur>
  <id>42</id>
  <nom>Awa Traoré</nom>
  <actif>true</actif>
  <role>etudiant</role>
  <adresse>
    <ville>Ouagadougou</ville>
    <pays>Burkina Faso</pays>
  </adresse>
</utilisateur>
```

### XSD (XML Schema Definition)

Le pendant de JSON Schema pour XML est **XSD**, un langage permettant de définir formellement la structure, les types et les contraintes d'un document XML.

```xml
<xs:element name="utilisateur">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="id" type="xs:integer"/>
      <xs:element name="nom" type="xs:string"/>
      <xs:element name="role">
        <xs:simpleType>
          <xs:restriction base="xs:string">
            <xs:enumeration value="etudiant"/>
            <xs:enumeration value="enseignant"/>
          </xs:restriction>
        </xs:simpleType>
      </xs:element>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```

### JSON et XML côte à côte

| Critère | JSON | XML |
|---|---|---|
| Concision | Élevée | Plus verbeux |
| Espaces de noms | Absents | Supportés nativement |
| Schéma de validation | JSON Schema | XSD |
| Transformation de document | Limitée | Riche (XSLT) |
| Adoption dans les API REST modernes | Très majoritaire | Marginale, souvent héritée |
| Adoption dans SOAP | Non applicable | Systématique |

## 4. SOAP

**SOAP** (*Simple Object Access Protocol*) est un protocole d'échange de messages structuré, apparu à la fin des années 1990, reposant intégralement sur XML et pouvant être transporté sur plusieurs protocoles (le plus souvent HTTP, mais pas exclusivement). Contrairement à REST, SOAP définit un format d'enveloppe de message strict :

```xml
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <!-- métadonnées : authentification, transaction, etc. -->
  </soap:Header>
  <soap:Body>
    <ObtenirUtilisateur xmlns="http://exemple.edu/services">
      <id>42</id>
    </ObtenirUtilisateur>
  </soap:Body>
</soap:Envelope>
```

### WSDL

Un service SOAP est décrit par un document **WSDL** (*Web Services Description Language*), qui définit formellement et exhaustivement les opérations disponibles, les types de messages échangés (souvent via un schéma XSD associé) et l'adresse du service. Cette description rigide permet une génération automatique de code client dans de nombreux langages, ce qui a longtemps constitué l'un des attraits majeurs de SOAP dans les environnements d'entreprise fortement outillés (ex. Java/.NET).

### Pourquoi SOAP reste employé dans certains contextes

Bien que largement supplanté par REST pour les nouveaux développements, SOAP demeure présent dans des contextes précis, généralement pour des raisons de continuité historique et d'exigences propres à certains secteurs :

- **Systèmes hérités** (*legacy*) dans les grandes organisations (banques, administrations, compagnies aériennes), où le coût de migration d'une interface stable et fonctionnelle n'est pas justifié par les seuls avantages de REST.
- **Contrats formels stricts** : la rigueur du WSDL et la vérifiabilité poussée par schéma XSD sont parfois recherchées dans des échanges interentreprises à fort enjeu contractuel.
- **Fonctionnalités standardisées avancées** (transactions distribuées, fiabilité des messages, sécurité au niveau du message plutôt qu'au niveau du transport), définies par des extensions du protocole (WS-Security, WS-ReliableMessaging), utiles dans des contextes d'intégration d'entreprise complexes que REST ne couvre pas nativement de façon aussi normalisée.

## 5. Sérialisation et désérialisation

La **sérialisation** transforme une structure de données en mémoire (objet, dictionnaire) en un flux de texte ou d'octets transportable ; la **désérialisation** effectue l'opération inverse à la réception.

```python
import json

# Sérialisation : structure Python → chaîne JSON
utilisateur = {"id": 42, "nom": "Awa Traoré", "role": "etudiant"}
texte_json = json.dumps(utilisateur)

# Désérialisation : chaîne JSON → structure Python
donnees = json.loads(texte_json)
print(donnees["nom"])  # Awa Traoré
```

### Un point de vigilance : la désérialisation de données non fiables

Comme évoqué dans le module *Sécurité des applications* (catégorie « défaillances d'intégrité des données et logiciels » de l'OWASP Top 10), désérialiser une donnée provenant d'une source non fiable peut, selon le langage et la bibliothèque utilisés, exposer à une exécution de code arbitraire si le mécanisme de désérialisation est capable de reconstruire des objets complexes plutôt que de simples structures de données. JSON, dans son usage standard (`json.loads` en Python, `JSON.parse` en JavaScript), ne présente pas ce risque car il ne reconstruit que des types de données primitifs ; le risque apparaît surtout avec des mécanismes de sérialisation plus « riches » propres à certains langages (ex. `pickle` en Python), qu'il convient de ne jamais utiliser sur une donnée reçue d'un client non authentifié ou non fiable.

## À retenir

- JSON s'est imposé comme le format dominant des API REST modernes pour sa concision et sa correspondance directe avec les structures de données natives des langages courants.
- XML reste plus verbeux mais offre des capacités historiquement plus riches (espaces de noms, transformations, schémas XSD) ; JSON Schema en constitue l'équivalent côté JSON.
- SOAP est un protocole XML strict, décrit par un contrat WSDL, encore présent dans des systèmes hérités ou des contextes d'intégration d'entreprise à exigences formelles fortes.
- Sérialisation et désérialisation sont les opérations symétriques de conversion entre structures en mémoire et représentation transportable ; la désérialisation de données non fiables via des mécanismes « riches » est une source de vulnérabilité connue.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
