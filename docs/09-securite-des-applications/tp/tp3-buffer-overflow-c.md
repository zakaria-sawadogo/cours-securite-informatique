# TP3 — Exploitation contrôlée d'un buffer overflow en C (2h)

## Cadre du TP

Manipulations exclusivement sur des binaires fournis par l'enseignant, compilés spécifiquement pour l'exercice (protections désactivées volontairement à des fins pédagogiques : `-fno-stack-protector -z execstack -no-pie`), exécutés dans une machine virtuelle de laboratoire isolée. Ce TP est encadré et progressif.

## Objectifs
- Observer concrètement un débordement de tampon sur la pile.
- Comprendre l'effet des protections modernes (stack canary, ASLR, DEP/NX) en les réactivant.

## Exercice 1 — Provoquer un plantage contrôlé

À partir du code source fourni (fonction avec `strcpy` non bornée dans un buffer local), compiler sans protections et exécuter le programme avec une entrée progressivement plus longue jusqu'à provoquer un plantage (segmentation fault). Observer avec `gdb` l'état des registres au moment du plantage.

## Exercice 2 — Localiser précisément le débordement

À l'aide d'un motif de caractères unique (technique du « cyclic pattern », outil fourni ou motif construit manuellement), déterminer précisément le décalage (offset) entre le début du buffer et l'écrasement de l'adresse de retour.

## Exercice 3 — Redirection contrôlée du flot d'exécution

Construire une entrée qui écrase précisément l'adresse de retour par l'adresse d'une fonction interne du programme non normalement atteignable (ex. une fonction `secret()` présente dans le code mais jamais appelée), et vérifier avec `gdb` que l'exécution est bien redirigée vers cette fonction.

## Exercice 4 — Effet des protections modernes

Recompiler le même programme avec les protections activées par défaut (`-fstack-protector-all`, sans les options désactivant ASLR/DEP). Retenter l'exploitation de l'exercice 3 et documenter précisément ce qui empêche désormais l'attaque de réussir (détection du canary, ou échec dû à l'ASLR).

## Exercice 5 — Correction du code source

Réécrire le code source original en remplaçant `strcpy` par une fonction bornée (`snprintf` ou équivalent avec vérification de taille). Recompiler et vérifier qu'aucune entrée, même très longue, ne provoque plus de plantage.

## À rendre

Le code source original et corrigé, les sorties `gdb` documentant chaque étape (exercices 1 à 4), et une synthèse écrite (15 lignes) reliant cette expérience aux chapitres 5 et 6 du module *Algorithmique et Programmation en C* et au chapitre 5 de ce module.
