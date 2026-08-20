# TP3 — Détection de phishing par NLP (2h)

## Objectifs
- Construire un classifieur de détection de phishing à partir du contenu textuel d'e-mails.
- Comprendre les limites d'un modèle NLP simple face aux techniques d'évasion.

## Préparation

Corpus public fourni par l'enseignant (échantillon anonymisé d'e-mails légitimes et de phishing, usage strictement pédagogique).

## Exercice 1 — Représentation TF-IDF

Charger le corpus, effectuer un nettoyage basique (minuscules, suppression de la ponctuation). Construire une représentation TF-IDF du contenu des messages avec `TfidfVectorizer` de `scikit-learn`.

## Exercice 2 — Entraînement et évaluation

Entraîner un classifieur (régression logistique ou Naive Bayes multinomial, adapté au texte) sur la détection phishing/légitime. Évaluer avec matrice de confusion, précision, rappel, F1-score.

## Exercice 3 — Mots les plus discriminants

Identifier les mots/tokens ayant le plus de poids dans la décision du modèle (coefficients de la régression logistique). Discuter si ces mots correspondent à une intuition raisonnable (urgence, demande d'action, termes financiers).

## Exercice 4 — Robustesse aux techniques d'évasion (démonstration)

Prendre 3 e-mails de phishing correctement détectés par le modèle. Leur appliquer manuellement de légères modifications (fautes volontaires, remplacement de lettres par des caractères visuellement proches, ajout de texte neutre) et vérifier si le modèle continue à les détecter. Commenter par écrit ce que cela révèle sur la robustesse d'un modèle NLP simple, en lien avec le chapitre 6 du cours (attaques adversariales sur le texte).

## Exercice 5 — Ajout de caractéristiques techniques

Ajouter au moins une caractéristique non textuelle simple (ex. présence d'une URL dans le message, nombre de liens) et évaluer si elle améliore les performances du modèle.

## À rendre

Le notebook/script complet et les réponses écrites aux exercices 3 et 4.

---

*Dr. Zakaria Sawadogo — École Polytechnique de Ouagadougou*
