# Exercices IA — Niveau 1 (Bases)

## Exercice 1 — Premier appel API

Écris le code Python d'un appel à une API de chat completion qui envoie un message système demandant au modèle de répondre uniquement par "oui" ou "non", et un message utilisateur "Le ciel est-il vert ?".

## Exercice 2 — Où stocker la clé API

Explique en 2-3 phrases pourquoi une clé API de LLM ne doit jamais être placée dans du code frontend (JavaScript exécuté dans le navigateur), et où elle doit vivre à la place.

## Exercice 3 — Few-shot prompting

Écris une liste de messages (`system` + 2 exemples `user`/`assistant` + 1 nouvelle question `user`) pour une tâche de classification d'emails en "spam" ou "légitime".

## Exercice 4 — Sortie structurée

Décris (en JSON Schema, format `type: object` avec `properties`) le schéma attendu pour extraire d'un texte libre une structure `{ "titre": string, "date": string, "lieu": string }` décrivant un événement.

## Exercice 5 — Tokens et coût

Un modèle facture 0,15 $ par million de tokens en entrée et 0,60 $ par million de tokens en sortie. Pour un appel avec un prompt de 2000 tokens et une réponse de 500 tokens, calcule le coût de cet appel en dollars.
