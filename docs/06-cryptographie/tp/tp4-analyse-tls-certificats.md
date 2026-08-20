# TP4 — Analyse d'une négociation TLS et des certificats (2h)

## Objectifs
- Observer concrètement une poignée de main TLS et un certificat X.509.
- Identifier des erreurs de configuration TLS courantes.

## Exercice 1 — Inspection d'un certificat

```bash
openssl s_client -connect exemple-fournit-par-enseignant.tld:443 -showcerts </dev/null
openssl x509 -in certificat.pem -text -noout
```

Identifier dans la sortie : le sujet, l'autorité de certification émettrice, la période de validité, l'algorithme de signature utilisé, la clé publique et sa taille.

## Exercice 2 — Chaîne de confiance

À partir de la sortie de l'exercice 1, reconstituer la chaîne de certificats (certificat serveur → intermédiaire(s) → racine) et expliquer par écrit comment un navigateur valide cette chaîne jusqu'à une ancre de confiance.

## Exercice 3 — Capture et analyse d'une poignée de main TLS 1.3 (Wireshark)

Sur un environnement de laboratoire fourni, capturer une connexion TLS 1.3 avec Wireshark. Identifier dans la capture : le `ClientHello`, le `ServerHello`, le certificat transmis, et le passage au trafic chiffré. Noter que le contenu applicatif n'est effectivement pas lisible en clair après la poignée de main.

## Exercice 4 — Audit d'une configuration TLS

À l'aide d'un outil de test de configuration TLS (ex. `testssl.sh` en local, ou une analyse manuelle des suites proposées par un serveur de laboratoire fourni par l'enseignant), identifier si le serveur accepte encore des protocoles ou suites obsolètes (TLS 1.0/1.1, RC4, absence de confidentialité persistante). Rédiger un mini-constat au format vu dans le module *Audit organisation et technique* (observation, risque, recommandation).

## Exercice 5 — Synthèse

Rédiger une synthèse (15 lignes) reliant explicitement les éléments observés dans ce TP aux chapitres du cours : où intervient l'échange de clé asymétrique (chapitre 4), où intervient la vérification de certificat (chapitre 5), où intervient le chiffrement symétrique de la session (chapitre 2), et pourquoi ce TP illustre le principe du chiffrement hybride.

## À rendre

Le certificat analysé, la capture Wireshark (ou une capture d'écran annotée), le constat d'audit de l'exercice 4, et la synthèse de l'exercice 5.
