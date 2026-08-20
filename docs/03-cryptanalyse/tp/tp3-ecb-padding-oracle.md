# TP3 — Attaque sur le mode ECB et padding oracle (2h)

## Objectifs
- Visualiser concrètement la faiblesse du mode ECB.
- Comprendre et mettre en œuvre une attaque par padding oracle simplifiée sur CBC/PKCS#7.

## Exercice 1 — Faiblesse du mode ECB

Chiffrer une image bitmap simple (fournie, au format non compressé) avec AES en mode ECB, bloc par bloc, en utilisant une bibliothèque cryptographique standard. Afficher l'image chiffrée obtenue et observer que les motifs de l'image d'origine restent visibles. Répéter l'opération en mode CBC (avec IV aléatoire) et comparer visuellement.

## Exercice 2 — Blocs identiques, sorties identiques

Chiffrer en ECB un texte contenant des répétitions volontaires (ex. le même mot de passe répété plusieurs fois, aligné sur la taille de bloc). Observer que les blocs de texte chiffré correspondants sont identiques, et expliquer par écrit pourquoi cela peut permettre à un attaquant de déduire de l'information sans même déchiffrer.

## Exercice 3 — Padding oracle simplifié (guidée)

L'enseignant fournit un petit service (script local) qui déchiffre un texte chiffré CBC/PKCS#7 fourni par l'étudiant et renvoie uniquement `"padding valide"` ou `"padding invalide"`, sans révéler le contenu déchiffré.

1. Comprendre le fonctionnement du padding PKCS#7 (valeur de complément ajoutée pour aligner sur la taille de bloc).
2. En manipulant l'avant-dernier bloc chiffré (bloc « leurre ») octet par octet et en observant la réponse de l'oracle, retrouver l'octet en clair correspondant du dernier bloc.
3. Étendre la méthode pour déchiffrer un bloc complet.
4. Documenter, étape par étape, comment cette même logique permettrait en théorie de déchiffrer un message entier bloc par bloc.

## Exercice 4 — Contre-mesures

Rédiger une fiche de synthèse (15 lignes) expliquant comment ces deux attaques (ECB, padding oracle) auraient pu être évitées : choix du mode opératoire (éviter ECB), authentification du texte chiffré (AEAD tel qu'AES-GCM, qui empêche le déchiffrement d'un texte modifié), messages d'erreur uniformisés côté serveur.

## À rendre

Les deux images chiffrées (ECB et CBC) de l'exercice 1, le script de l'exercice 3, et la fiche de synthèse de l'exercice 4.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
