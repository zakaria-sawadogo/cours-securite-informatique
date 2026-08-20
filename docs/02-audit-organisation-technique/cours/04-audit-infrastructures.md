# Chapitre 4 — Audit des infrastructures réseau et systèmes

!!! tip "Présentation de ce chapitre"
    [🖥 Ouvrir les diapositives](../../presentation.html?deck=02-audit-organisation-technique/slides/04-audit-infrastructures.txt){ target=_blank }

## 1. Audit de l'architecture réseau

- **Segmentation** : vérifier l'existence de zones réseau distinctes (DMZ, réseau interne, réseau d'administration) et que le filtrage entre zones suit le principe du moindre privilège.
- **Flux autorisés** : revue des règles de pare-feu (une règle `ANY-ANY` en sortie ou entrée est presque toujours une non-conformité à documenter).
- **Points d'entrée exposés** : recensement de tout ce qui est accessible depuis Internet (VPN, portails, services mal exposés par erreur).

## 2. Audit des équipements réseau

- Mots de passe par défaut non changés (routeurs, switchs, imprimantes réseau, caméras IP).
- Protocoles non chiffrés encore actifs (Telnet, HTTP d'administration, SNMP v1/v2 avec communauté `public`).
- Firmware obsolète et absence de politique de mise à jour.

## 3. Audit des systèmes d'exploitation (hardening)

Points de contrôle typiques, alignés sur les référentiels de durcissement (CIS Benchmarks) :

| Domaine | Points de contrôle |
|---|---|
| Comptes et authentification | comptes par défaut désactivés, politique de mots de passe, MFA pour les comptes à privilèges |
| Services | services inutiles désactivés (réduction de la surface d'attaque) |
| Correctifs | politique de patch management, délai moyen d'application des correctifs critiques |
| Journalisation | logs activés, centralisés, conservés une durée suffisante |
| Droits | principe du moindre privilège, séparation des comptes admin/utilisateur |

## 4. Cas particulier : audit d'un domaine Active Directory

L'annuaire Active Directory est une cible privilégiée car il centralise l'authentification. Points d'audit fréquents :

- comptes avec mot de passe n'expirant jamais ou marqués `PASSWD_NOTREQD` ;
- délégation Kerberos non contrainte mal maîtrisée (risque d'usurpation) ;
- groupes à privilèges (`Domain Admins`) surdimensionnés ;
- absence de séparation entre comptes d'administration et comptes utilisateurs quotidiens ;
- outils dédiés d'audit : `BloodHound` (cartographie des chemins d'attaque), `PingCastle` (score de maturité AD).

## 5. Audit du Cloud et des environnements virtualisés

- Vérification des configurations de stockage (buckets S3 ou équivalents en accès public non justifié).
- Gestion des identités et accès (IAM) : clés d'API non tournées, permissions trop larges.
- Isolation entre locataires (tenants) dans un environnement mutualisé.
- Journalisation et alerte (CloudTrail ou équivalent) activées et supervisées.

## 6. Restitution d'un audit d'infrastructure

Chaque constat technique doit être formulé selon la structure : **observation** (fait constaté et preuve), **risque** (ce que cela permet à un attaquant), **recommandation** (action corrective priorisée), **référence** (CIS Benchmark, ISO 27002, CVE le cas échéant).

## À retenir

- L'audit d'infrastructure combine architecture réseau, configuration système et, de plus en plus, environnements Cloud/AD.
- Les référentiels de durcissement (CIS Benchmarks) fournissent des points de contrôle réutilisables et mesurables.
- Chaque constat doit relier un fait technique à un risque métier compréhensible par la direction.
