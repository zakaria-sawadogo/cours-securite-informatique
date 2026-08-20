# TP2 — Reconnaissance et scan réseau (Nmap) (2h)

## Cadre légal du TP

Ce TP s'exécute exclusivement contre des machines de laboratoire déployées localement pour l'exercice (ex. Metasploitable2 dans une VM isolée du réseau de l'établissement). **Aucun scan ne doit être effectué contre une cible hors de ce périmètre.**

## Objectifs
- Pratiquer la reconnaissance passive et active.
- Utiliser Nmap pour la découverte d'hôtes, le scan de ports et le fingerprinting.

## Exercice 1 — Reconnaissance passive (démonstration guidée)

À partir d'un nom de domaine fictif fourni par l'enseignant, illustrer les techniques d'OSINT : WHOIS, recherche de sous-domaines, informations exposées publiquement (sans interaction directe avec la cible).

## Exercice 2 — Découverte d'hôtes

```bash
nmap -sn 192.168.56.0/24
```

Identifier les hôtes actifs du réseau de laboratoire et documenter leurs adresses IP et adresses MAC.

## Exercice 3 — Scan de ports et détection de versions

```bash
nmap -sV -sC -p- 192.168.56.101
```

Pour chaque port ouvert identifié, documenter dans un tableau : numéro de port, service, version détectée, et rechercher si une CVE connue correspond à cette version (base NVD).

## Exercice 4 — Scan furtif et implications

Comparer `nmap -sS` (SYN scan) à `nmap -sT` (connect scan) : expliquer par écrit la différence de fonctionnement et pourquoi l'un est plus discret que l'autre vis-à-vis d'un système de détection d'intrusion.

## Exercice 5 — Rapport de reconnaissance

Rédiger une fiche de synthèse (1 page) listant les hôtes découverts, les services exposés jugés à risque, et une première hypothèse de vecteurs d'attaque à approfondir (sans exploitation à ce stade).

## À rendre

Le tableau de résultats du scan et la fiche de synthèse.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
