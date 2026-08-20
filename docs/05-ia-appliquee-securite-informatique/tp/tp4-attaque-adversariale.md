# TP4 — Attaque adversariale sur un classifieur (2h)

## Objectifs
- Construire un classifieur d'images simple.
- Mettre en œuvre une attaque adversariale de base et observer son effet.
- Discuter les contre-mesures.

## Préparation

```bash
pip install torch torchvision scikit-learn matplotlib --break-system-packages
```

Jeu de données : MNIST (chiffres manuscrits) ou équivalent simple, suffisant pour illustrer le principe sans nécessiter de gros moyens de calcul.

## Exercice 1 — Entraînement d'un classifieur de référence

Entraîner un petit réseau de neurones (quelques couches denses ou un CNN simple) à classer les chiffres MNIST. Évaluer sa précision sur l'ensemble de test (viser > 95 %).

## Exercice 2 — Attaque FGSM (Fast Gradient Sign Method)

Implémenter (ou utiliser une bibliothèque comme `torchattacks` si disponible) l'attaque FGSM en boîte blanche : calculer le gradient de la perte par rapport à l'image d'entrée, et ajouter une perturbation `epsilon * sign(gradient)`. Appliquer cette perturbation à plusieurs images correctement classées et observer le taux de mauvaise classification obtenu.

## Exercice 3 — Visualisation

Afficher côte à côte, pour 5 exemples : l'image originale, la perturbation appliquée (amplifiée pour la visibilité), et l'image perturbée, avec les prédictions du modèle avant/après. Faire varier `epsilon` et observer le compromis entre imperceptibilité de la perturbation et taux de succès de l'attaque.

## Exercice 4 — Contre-mesure : entraînement adversarial

Ré-entraîner le modèle en incluant, pour une partie des exemples de chaque lot d'entraînement, une version perturbée par FGSM (entraînement adversarial simple). Comparer la robustesse du nouveau modèle face aux mêmes attaques de l'exercice 2.

## Exercice 5 — Synthèse

Rédiger une synthèse (15 lignes) reliant cette expérimentation au chapitre 6 du cours : transposer le raisonnement à un cas de sécurité réel (ex. classifieur de malware ou de phishing) et discuter pourquoi la boîte blanche est un cas plus favorable à l'attaquant que la boîte noire, et quelles limites pratiques cela impose à un attaquant réel.

## À rendre

Le notebook/script complet, les visualisations de l'exercice 3, et la synthèse de l'exercice 5.
