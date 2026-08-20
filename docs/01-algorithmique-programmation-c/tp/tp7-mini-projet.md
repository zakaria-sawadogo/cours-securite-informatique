# TP7 — Mini-projet : gestionnaire de contacts (2h)

## Objectif

Consolider l'ensemble des notions du module (structures, tableaux/listes, pointeurs, fichiers, fonctions) sur un projet complet.

## Cahier des charges

Développer en C un gestionnaire de contacts en ligne de commande, avec :

1. **Structure de données** : chaque contact est une `struct Contact { char nom[50]; char telephone[20]; char email[50]; }`.
2. **Stockage** : liste chaînée de contacts en mémoire.
3. **Fonctionnalités (menu interactif avec `switch`)** :
   - Ajouter un contact (validation basique des champs : pas de champ vide, longueur bornée avec `snprintf`/`strncpy` sécurisés).
   - Lister tous les contacts.
   - Rechercher un contact par nom (recherche séquentielle).
   - Supprimer un contact.
   - Sauvegarder l'ensemble des contacts dans un fichier texte (`contacts.txt`).
   - Charger les contacts depuis ce fichier au démarrage du programme.
   - Quitter (en libérant proprement toute la mémoire allouée).

## Contraintes techniques

- Compilation sans aucun avertissement (`-Wall -Wextra -std=c11`).
- Aucune fuite mémoire ni débordement détecté par `valgrind` ou `-fsanitize=address,leak`.
- Toute saisie utilisateur doit être bornée (pas de `scanf("%s", ...)` sans limite, pas de `gets`).
- Le code est découpé en fonctions cohérentes, chacune avec un prototype et un commentaire d'en-tête (rôle, paramètres, valeur de retour).

## Barème indicatif

| Critère | Points |
|---|---:|
| Fonctionnalités complètes et correctes | 8 |
| Absence de bug mémoire (vérifié à l'outil) | 6 |
| Qualité du code (découpage, nommage, gestion d'erreurs) | 4 |
| Robustesse face aux entrées invalides | 2 |

## À rendre

Le code source complet (`.c`/`.h`), un `README.md` court expliquant comment compiler et utiliser le programme, et la sortie de `valgrind` (ou d'AddressSanitizer) montrant l'absence de fuite.
