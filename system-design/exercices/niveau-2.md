# Exercices System Design — Niveau 2 (Intermédiaire)

## Exercice 1 — CAP theorem appliqué

Un système de panier d'achat e-commerce distribué subit une partition réseau entre deux datacenters. Explique ce que signifierait choisir la Cohérence (C) plutôt que la Disponibilité (A) dans ce contexte précis, avec un exemple concret de comportement utilisateur pour chaque choix.

## Exercice 2 — Message queue : découpler un traitement lent

Un endpoint `POST /commandes` doit, en plus de créer la commande, envoyer un email de confirmation et générer une facture PDF. Actuellement, tout se fait de façon synchrone dans la requête HTTP, ce qui la rend lente. Propose une architecture avec une queue pour découpler ces traitements, avec un schéma texte du flux.

## Exercice 3 — Rate limiting

Décris en pseudocode (ou en français structuré) le fonctionnement de l'algorithme "token bucket" pour limiter un client à 100 requêtes par minute.

## Exercice 4 — Circuit breaker

Un service de recommandations produit est appelé par la page d'accueil, mais tombe en panne et met 30 secondes à timeout par requête. Explique comment un circuit breaker améliore la situation, et ce que la page d'accueil devrait afficher pendant que le circuit est "ouvert".

## Exercice 5 — Raccourcisseur d'URL : première conception

Esquisse (schéma + 5-6 phrases) une architecture simple pour un raccourcisseur d'URL supportant 1 million de créations par jour : schéma de données, méthode de génération de l'identifiant court, et emplacement du cache.
