# Exercices System Design — Niveau 1 (Bases)

## Exercice 1 — Schéma client-serveur

Dessine (en texte/ASCII ou description) le chemin complet d'une requête "afficher le profil d'un utilisateur" depuis le navigateur jusqu'à la base de données et retour, en incluant un load balancer et un cache.

## Exercice 2 — Vertical vs horizontal

Une application commence à ralentir sous la charge. Liste deux options de scalabilité verticale et deux options de scalabilité horizontale envisageables, puis explique en 2-3 phrases dans quel ordre tu les essaierais et pourquoi.

## Exercice 3 — Cache-aside pas à pas

Décris étape par étape (en 4-5 points) ce qui se passe quand un utilisateur demande une donnée qui n'est pas encore dans le cache, avec le pattern cache-aside.

## Exercice 4 — Sticky sessions ou session externalisée ?

Une application stocke la session utilisateur en mémoire locale sur chaque serveur applicatif, derrière un load balancer round-robin. Identifie le problème que cela pose, et propose deux solutions différentes.

## Exercice 5 — Réplication : lag

Explique en 2-3 phrases ce qu'est le "replication lag" entre une base primaire et une réplique, et donne un exemple concret de bug utilisateur que cela peut provoquer (ex. après avoir posté un commentaire).
