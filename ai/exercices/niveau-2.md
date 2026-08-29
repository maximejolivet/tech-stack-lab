# Exercices IA — Niveau 2 (Intermédiaire)

## Exercice 1 — Function calling

Décris (nom, description, paramètres en JSON Schema) un outil `rechercher_produit(nom: string, prix_max: number)` destiné à être proposé à un modèle via function calling, pour un assistant e-commerce.

## Exercice 2 — Pipeline RAG minimal

Décris étape par étape (5-6 points) le pipeline complet d'un système RAG, depuis l'indexation des documents jusqu'à la génération de la réponse finale.

## Exercice 3 — Recherche par similarité

Étant donné trois embeddings de documents (représentés comme de simples listes de 3 nombres pour simplifier) et l'embedding d'une question, écris en pseudocode/Python la logique qui calcule la similarité cosinus entre la question et chaque document, et retourne le document le plus proche.

## Exercice 4 — Fine-tuning, RAG ou prompt engineering ?

Pour chacun des trois besoins suivants, indique quel levier (prompt engineering, RAG, fine-tuning) est le plus adapté en premier réflexe, et justifie en une phrase :
1. Un chatbot qui doit répondre en se basant sur la documentation produit mise à jour chaque semaine.
2. Un assistant qui doit toujours répondre dans un format de rapport très spécifique et inhabituel, de façon parfaitement fiable, sur des milliers de requêtes par jour.
3. Un premier prototype qui doit résumer des articles de presse en 3 phrases.

## Exercice 5 — Gestion de la fenêtre de contexte

Un agent conversationnel accumule l'historique complet de la conversation à chaque appel. Après 50 échanges, l'appel commence à dépasser la limite de contexte du modèle. Propose deux stratégies différentes pour gérer ce problème, avec leurs compromis respectifs.
