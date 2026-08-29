# Exercices Security — Niveau 3 (Avancé)

## Exercice 1 — Audit d'une API de login vulnérable

Voici une API de connexion volontairement vulnérable :

```php
// login.php
$stmt = $pdo->query("SELECT * FROM users WHERE email = '{$_POST['email']}'");
$user = $stmt->fetch();

if ($user && $user['password'] === $_POST['password']) {
    session_start();
    $_SESSION['user_id'] = $user['id'];
    echo json_encode(['status' => 'ok']);
}
```

Liste toutes les failles présentes dans ce code (au moins 3), puis réécris une version corrigée.

## Exercice 2 — Stockage d'un JWT côté client

Une application SPA stocke son JWT d'authentification dans `localStorage`. Explique le risque principal, l'alternative recommandée, et une limite de cette alternative vis-à-vis du CSRF.

## Exercice 3 — Révocation d'un JWT

Un utilisateur se fait bannir par un administrateur, mais son JWT (valide encore 24h) continue de fonctionner sur l'API. Propose deux approches différentes pour permettre une révocation effective, avec leurs compromis respectifs.

## Exercice 4 — SSRF

Un formulaire "Générer un aperçu" accepte une URL fournie par l'utilisateur et fait un `fetch()` serveur vers cette URL pour en extraire le titre de la page. Explique comment un attaquant pourrait exploiter ce endpoint en SSRF, et propose deux mesures de mitigation.

## Exercice 5 — Dependency confusion

Explique en 4-5 phrases le principe d'une attaque par *dependency confusion* sur un gestionnaire de paquets (npm/Composer), et deux mesures concrètes pour s'en protéger dans une organisation qui publie des paquets internes.
