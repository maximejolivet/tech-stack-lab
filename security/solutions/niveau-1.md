# Solutions — Niveau 1 (Bases)

## Exercice 1

```php
$stmt = $pdo->prepare("SELECT * FROM products WHERE category = :category");
$stmt->execute(['category' => $_GET['category']]);
$result = $stmt->fetchAll();
```

## Exercice 2

Le commentaire est injecté tel quel dans le HTML : un attaquant peut poster `<script>...</script>` qui s'exécutera dans le navigateur de tout visiteur affichant ce commentaire (XSS stocké).

```php
echo "<div class='comment'>" . htmlspecialchars($_POST['comment'], ENT_QUOTES, 'UTF-8') . "</div>";
```

## Exercice 3

Le mot de passe est stocké en clair en base — en cas de fuite de données, tous les mots de passe sont directement exploitables.

```php
$hashedPassword = password_hash($_POST['password'], PASSWORD_BCRYPT);
$stmt = $pdo->prepare("INSERT INTO users (email, password) VALUES (?, ?)");
$stmt->execute([$_POST['email'], $hashedPassword]);
```

## Exercice 4

Ce formulaire est vulnérable car il ne contient aucun token unique lié à la session de l'utilisateur : un site tiers malveillant pourrait héberger un formulaire identique qui soumet automatiquement vers `/account/delete`, et le navigateur de la victime enverrait son cookie de session valide sans qu'elle ait rien décidé. Il manque un champ caché (`_token`) contenant un token CSRF, vérifié côté serveur avant d'exécuter la suppression.

## Exercice 5

C'est dangereux car la clé est visible par quiconque a accès au code source (dépôt Git, y compris son historique) et peut être utilisée pour effectuer des paiements/appels API au nom du compte. Elle devrait être stockée dans une variable d'environnement (`.env` non commité) ou un gestionnaire de secrets, jamais écrite en dur dans le code.
