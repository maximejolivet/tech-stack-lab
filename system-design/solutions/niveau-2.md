# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

**Choisir la Cohérence (C)** : pendant la partition, le système refuse ou met en attente les opérations sur le panier qui ne peuvent pas être confirmées comme cohérentes entre les deux datacenters — l'utilisateur peut voir une erreur "service temporairement indisponible" en essayant d'ajouter un article, mais ne verra jamais un état incohérent (ex. un article ajouté qui disparaît ensuite).

**Choisir la Disponibilité (A)** : le système continue d'accepter les modifications du panier sur chaque datacenter indépendamment pendant la partition, quitte à devoir réconcilier les deux versions du panier une fois la partition résorbée (ex. fusionner les articles ajoutés des deux côtés) — l'utilisateur ne voit jamais d'erreur, mais un état temporairement incohérent est possible.

## Exercice 2

```text
POST /commandes
   │
   ▼
Serveur applicatif
   │  1. crée la commande en base (opération synchrone, rapide)
   │  2. publie un message "commande.creee" avec l'ID de commande
   │  3. répond immédiatement 201 Created au client
   ▼
Queue "commande.creee"
   ├──> Worker Email       (consomme le message, envoie l'email de confirmation)
   └──> Worker Facturation (consomme le message, génère le PDF)
```

Le client reçoit une réponse rapide dès que la commande est enregistrée ; l'email et la facture sont traités de façon asynchrone par des workers séparés, sans bloquer la requête HTTP initiale.

## Exercice 3

Un "seau" (bucket) contient au maximum 100 jetons. Chaque requête du client consomme un jeton si le seau en contient au moins un ; sinon la requête est rejetée (HTTP 429). Le seau se reremplit à un débit constant (ici, un jeton toutes les 0,6 seconde, pour revenir à 100 jetons en 1 minute), jusqu'à sa capacité maximale — ce qui permet d'absorber un pic ponctuel de requêtes (tant que le seau n'est pas vide) tout en garantissant qu'en moyenne, le débit ne dépasse jamais 100 requêtes/minute.

## Exercice 4

Sans circuit breaker, chaque requête vers la page d'accueil attend jusqu'à 30 secondes le timeout du service de recommandations avant de pouvoir continuer, ce qui sature les threads/connexions du serveur applicatif et peut faire tomber la page d'accueil entière à cause d'une panne d'un service secondaire.

Avec un circuit breaker : après un nombre défini d'échecs consécutifs, le circuit passe à l'état "ouvert" — les appels suivants vers ce service échouent **immédiatement** (sans attendre le timeout) pendant une période donnée, avant de retester périodiquement (état "semi-ouvert") si le service est revenu. Pendant que le circuit est ouvert, la page d'accueil devrait afficher son contenu sans la section recommandations (ou un contenu par défaut/statique) plutôt que d'attendre ou d'échouer entièrement — c'est la dégradation gracieuse.

## Exercice 5

```text
Client --> Load Balancer --> Serveur applicatif
                                  │
                                  ├─ 1. vérifie le cache (Redis) pour l'URL longue -> code court existant ?
                                  ├─ 2. sinon, génère un identifiant court (compteur global encodé en base62)
                                  ├─ 3. écrit (code_court -> url_longue) en base
                                  └─ 4. écrit également dans le cache pour accélérer les prochaines redirections
```

Schéma de données minimal : table `urls (code_court PRIMARY KEY, url_longue, date_creation)`. Génération : un compteur auto-incrémenté (ou distribué via une plage réservée par serveur) encodé en base62 pour rester court et lisible. Le cache (Redis) est placé devant la base pour la lecture (`GET /{code_court}` — l'opération la plus fréquente, largement dominante face aux créations), avec la base comme source de vérité en cas de cache miss.
