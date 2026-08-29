# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

C'est une faille de Broken Access Control (IDOR — Insecure Direct Object Reference) : n'importe quel utilisateur connecté peut incrémenter l'`id` dans l'URL et consulter les commandes de n'importe qui d'autre.

```
Si commande.user_id !== utilisateur_connecte.id :
    retourner 403 Forbidden
Sinon :
    retourner les détails de la commande
```

## Exercice 2

Attributs manquants : `HttpOnly` (empêche la lecture du cookie en JavaScript, protège du vol par XSS), `Secure` (n'envoie le cookie qu'en HTTPS), `SameSite` (limite l'envoi cross-site, réduit la surface CSRF).

```php
setcookie('session_id', $sessionId, [
    'path' => '/',
    'httponly' => true,
    'secure' => true,
    'samesite' => 'Lax',
]);
```

## Exercice 3

Clé de comptage : combinaison IP + email soumis (comptabiliser par IP seule permettrait de bloquer un utilisateur légitime derrière un NAT partagé ; par email seul permettrait un déni de service ciblé sur un compte via des IP multiples — combiner les deux limite les deux risques). Fenêtre de temps : ex. 5 tentatives par 15 minutes. Une fois la limite atteinte : refuser les tentatives suivantes avec un délai de verrouillage croissant (ou CAPTCHA), et journaliser l'événement pour détection d'attaque en cours.

## Exercice 4

```
Content-Security-Policy: script-src 'self' https://cdn.exemple.com
```

## Exercice 5

D'abord vérifier si la vulnérabilité est réellement exploitable dans le contexte du projet (une faille dans une fonction jamais appelée par le code a un risque pratique nul, même si elle reste listée). Ensuite regarder si un correctif existe (`npm audit fix`) et s'il s'agit d'une mise à jour mineure/patch (généralement sûre) ou majeure (peut casser une API utilisée par le projet, nécessite des tests). Si aucun correctif n'existe encore, évaluer une solution de contournement temporaire (éviter la fonction concernée, isoler la dépendance) plutôt que de rester exposé en attendant un correctif amont.
