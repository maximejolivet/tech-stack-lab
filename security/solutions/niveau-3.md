# Solutions — Niveau 3 (Avancé)

## Exercice 1

Failles présentes :
1. **Injection SQL** — l'email est concaténé directement dans la requête.
2. **Mot de passe en clair** — comparaison directe `$user['password'] === $_POST['password']` implique un stockage non haché.
3. **Fixation de session** — `session_start()` sans régénérer l'ID de session après authentification (`session_regenerate_id`), ce qui laisse une fenêtre d'exploitation si un ID de session a été prédit/volé avant connexion.
4. **Pas de rate limiting** — aucune protection contre le brute force sur les tentatives de connexion.

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute(['email' => $_POST['email']]);
$user = $stmt->fetch();

if ($user && password_verify($_POST['password'], $user['password'])) {
    session_start();
    session_regenerate_id(true);
    $_SESSION['user_id'] = $user['id'];
    echo json_encode(['status' => 'ok']);
} else {
    http_response_code(401);
    echo json_encode(['status' => 'error']);
}
```

## Exercice 2

Risque principal : `localStorage` est accessible par n'importe quel script JavaScript exécuté sur la page — une faille XSS, même mineure, permet de voler directement le token. L'alternative recommandée est un cookie `HttpOnly` (inaccessible en JavaScript), envoyé automatiquement par le navigateur. Limite : un cookie envoyé automatiquement réintroduit une exposition au CSRF (contrairement à un token lu manuellement et envoyé en header par le JS), ce qui nécessite de le combiner avec une protection CSRF (token synchronisé, `SameSite=Strict`/`Lax`).

## Exercice 3

**Approche 1 — Liste de révocation (blocklist)** : maintenir en base/cache (Redis) une liste des tokens (ou des utilisateurs) révoqués, vérifiée à chaque requête authentifiée. Compromis : réintroduit un état côté serveur à consulter à chaque requête, ce qui annule en partie l'intérêt "stateless" du JWT, mais reste peu coûteux avec un cache rapide.

**Approche 2 — Durée de vie très courte + refresh token** : émettre des JWT d'accès de courte durée (ex. 5-15 min), renouvelés via un refresh token stocké côté serveur et révocable. Compromis : le bannissement met jusqu'à la durée de vie de l'access token pour prendre effet (fenêtre plus courte qu'avec un token de 24h, mais pas immédiate), et nécessite une infrastructure de refresh token additionnelle.

## Exercice 4

Un attaquant peut fournir une URL interne (`http://localhost:6379`, `http://169.254.169.254/latest/meta-data/` sur un cloud, ou une adresse du réseau privé de l'infrastructure) : le serveur, en confiance, effectue la requête pour son compte et peut exposer des services internes non censés être accessibles publiquement, voire des identifiants cloud.

Mitigations : (1) valider l'URL fournie contre une liste blanche de protocoles/hôtes autorisés et rejeter explicitement les plages d'adresses privées/loopback/métadonnées cloud avant d'effectuer la requête ; (2) exécuter la requête sortante depuis un réseau isolé sans accès aux ressources internes sensibles (segmentation réseau), en défense en profondeur si la validation applicative est contournée.

## Exercice 5

Le *dependency confusion* exploite le fait qu'un gestionnaire de paquets peut résoudre un nom de paquet identique entre un registre public et un registre privé interne à une organisation : si l'outillage n'est pas configuré pour prioriser explicitement le registre privé, il peut installer par erreur un paquet public malveillant portant le même nom (et une version supérieure) que le paquet interne légitime, exécutant du code arbitraire lors de l'installation. Mesures de protection : (1) réserver/publier systématiquement les noms de paquets internes également sur le registre public (paquet vide) pour empêcher un tiers de les squatter, et (2) configurer explicitement le gestionnaire de paquets (scope npm, repository Composer) pour ne résoudre les paquets internes que depuis le registre privé, sans fallback implicite vers le registre public.
