# Solutions — Niveau 1 (Bases)

## Exercice 1

```text
Navigateur
   │  GET /profil/42
   ▼
Load Balancer
   │  route vers un serveur applicatif disponible
   ▼
Serveur applicatif
   │  1. vérifie le cache (Redis) pour la clé "profil:42"
   │     - présent -> retourne directement au client
   │     - absent  -> continue
   ▼
Base de données (lecture du profil 42)
   │  résultat renvoyé au serveur applicatif
   │  le serveur écrit le résultat dans le cache
   ▼
Réponse HTML/JSON renvoyée au navigateur via le Load Balancer
```

## Exercice 2

Scalabilité verticale : augmenter le CPU/RAM du serveur applicatif ; augmenter les ressources de la base de données.
Scalabilité horizontale : ajouter des serveurs applicatifs derrière un load balancer ; ajouter des répliques de lecture pour la base de données.

Ordre recommandé : commencer par la scalabilité verticale (changement rapide, sans changement d'architecture) si la limite physique n'est pas encore atteinte et que la tolérance de panne n'est pas un enjeu immédiat ; passer à l'horizontale dès que la verticale atteint sa limite ou que la disponibilité (résister à la panne d'une machine) devient une exigence — l'horizontale demande plus de travail (état partagé, répartition) donc n'est introduite que quand elle apporte un bénéfice réel.

## Exercice 3

1. Le serveur applicatif reçoit la requête et calcule la clé de cache correspondante (ex. `"profil:42"`).
2. Il interroge le cache (Redis) avec cette clé — c'est un **cache miss** (donnée absente).
3. Il interroge la base de données pour récupérer la donnée réelle.
4. Il écrit le résultat dans le cache avec cette clé, avec une expiration (TTL) définie.
5. Il retourne la donnée au client. La prochaine requête sur la même clé sera un **cache hit**, servie directement depuis le cache sans repasser par la base.

## Exercice 4

**Problème** : un load balancer round-robin peut envoyer deux requêtes successives du même utilisateur vers deux serveurs différents ; si la session vit en mémoire locale sur le premier serveur, le deuxième serveur ne la connaît pas — l'utilisateur perd sa session (déconnexion, panier vidé) de façon aléatoire.

**Solutions** :
1. Externaliser la session dans un stockage partagé (Redis) accessible par tous les serveurs applicatifs, quel que soit celui qui traite la requête.
2. Configurer le load balancer en mode "sticky sessions" (même client → toujours le même serveur), au prix d'une perte de la répartition de charge si un serveur reçoit un utilisateur très actif, et d'un problème résiduel si ce serveur tombe.

## Exercice 5

Le "replication lag" est le délai entre l'écriture d'une donnée sur la base primaire et le moment où cette donnée est effectivement copiée et disponible sur une réplique — la réplication étant généralement asynchrone, ce délai n'est jamais strictement nul. Exemple de bug : un utilisateur poste un commentaire (écrit sur le primaire), la page se recharge immédiatement et lit depuis une réplique qui n'a pas encore reçu la réplication — le commentaire semble avoir disparu pendant quelques centaines de millisecondes.
