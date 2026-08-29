# Exercices Security — Niveau 2 (Intermédiaire)

## Exercice 1 — Broken Access Control

Un endpoint `GET /api/orders/{id}` retourne les détails d'une commande sans vérifier que l'utilisateur connecté en est bien le propriétaire. Décris l'attaque possible, puis écris (en pseudo-code) la vérification à ajouter.

## Exercice 2 — Cookies de session

Voici la configuration d'un cookie de session :

```php
setcookie('session_id', $sessionId, ['path' => '/']);
```

Identifie les attributs de sécurité manquants et réécris la configuration en les ajoutant.

## Exercice 3 — Rate limiting sur le login

Décris, en pseudo-code ou en français structuré, comment implémenter un rate limiting simple sur `POST /login` : quelle donnée utiliser comme clé de comptage, quelle fenêtre de temps, et quelle action une fois la limite atteinte.

## Exercice 4 — Content-Security-Policy

Un site charge des scripts depuis son propre domaine et depuis `https://cdn.exemple.com`, mais aucun autre. Écris l'en-tête `Content-Security-Policy` correspondant pour la directive `script-src`.

## Exercice 5 — Dépendance vulnérable

Une alerte `npm audit` signale une vulnérabilité "high" sur une dépendance transitive de ton projet. Décris en 3-4 phrases la démarche à suivre pour la traiter (ce qu'il faut vérifier avant de mettre à jour, et pourquoi on ne peut pas toujours corriger immédiatement).
