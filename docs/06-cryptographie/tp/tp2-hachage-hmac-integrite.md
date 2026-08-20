# TP2 — Hachage, HMAC et intégrité de fichiers (2h)

## Objectifs
- Utiliser des fonctions de hachage pour vérifier l'intégrité de fichiers.
- Implémenter et vérifier un HMAC.
- Comparer le coût de calcul de fonctions de hachage génériques et de fonctions dédiées au mot de passe.

## Exercice 1 — Empreintes et vérification d'intégrité

```bash
sha256sum fichier.txt > empreinte.sha256
# Après transfert/modification éventuelle du fichier :
sha256sum -c empreinte.sha256
```

Modifier volontairement un octet du fichier et vérifier que la commande de contrôle détecte l'altération.

## Exercice 2 — Démonstration de collision MD5 (guidée)

À partir de deux fichiers fournis par l'enseignant, illustrant une collision MD5 connue (deux fichiers différents, même empreinte MD5), calculer et comparer leurs empreintes MD5 et SHA-256. Observer que MD5 échoue à distinguer les deux fichiers tandis que SHA-256 les distingue correctement.

## Exercice 3 — HMAC

En Python (bibliothèque `hmac`) ou avec OpenSSL (`openssl dgst -sha256 -hmac "cle"`), calculer le HMAC-SHA256 d'un message avec une clé donnée. Vérifier qu'une modification d'un seul caractère du message ou de la clé change complètement le HMAC produit (effet avalanche).

## Exercice 4 — Vulnérabilité d'une construction naïve `H(clé || message)`

Reprendre la démonstration d'extension de longueur du module *Cryptanalyse* (TP2, exercice 4) et comparer avec HMAC : tenter la même attaque sur un HMAC-SHA256 et constater qu'elle échoue. Expliquer par écrit, en s'appuyant sur la construction vue en cours, pourquoi HMAC résiste là où `H(clé || message)` ne résiste pas.

## Exercice 5 — Coût de calcul : SHA-256 vs Argon2

Mesurer le temps de calcul de 10 000 hachages SHA-256 puis de 10 (!) hachages Argon2 (paramètres par défaut d'une bibliothèque standard). Commenter l'écart et relier à la notion de fonction volontairement lente pour le hachage de mots de passe.

## À rendre

Les scripts/commandes de chaque exercice et les réponses écrites aux exercices 2 et 4.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
