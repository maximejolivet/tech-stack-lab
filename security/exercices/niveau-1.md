# Exercices Security — Niveau 1 (Bases)

## Exercice 1 — Corriger une injection SQL

Voici du code PHP vulnérable :

```php
$result = $pdo->query("SELECT * FROM products WHERE category = '" . $_GET['category'] . "'");
```

Réécris-le en utilisant une requête préparée.

## Exercice 2 — Corriger une faille XSS

Voici un code qui affiche un commentaire posté par un utilisateur :

```php
echo "<div class='comment'>" . $_POST['comment'] . "</div>";
```

Explique le risque, puis corrige-le.

## Exercice 3 — Hachage de mot de passe

Voici le code d'inscription d'un utilisateur :

```php
$stmt = $pdo->prepare("INSERT INTO users (email, password) VALUES (?, ?)");
$stmt->execute([$_POST['email'], $_POST['password']]);
```

Identifie le problème et corrige-le avec `password_hash`.

## Exercice 4 — Reconnaître un token CSRF

Explique en 2-3 phrases pourquoi le formulaire suivant est vulnérable au CSRF, et ce qu'il manque pour le protéger :

```html
<form method="POST" action="/account/delete">
    <button type="submit">Supprimer mon compte</button>
</form>
```

## Exercice 5 — Secrets en dur

Voici un extrait de code :

```php
$apiKey = "sk_live_51Hf8x..."; // clé Stripe
```

Explique en une phrase pourquoi c'est dangereux, et comment ce secret devrait être géré à la place.
