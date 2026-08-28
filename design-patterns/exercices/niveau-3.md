# Exercices — Niveau 3 : Avancé

## 1. Appliquer la Dependency Inversion et rendre le code testable

Voici un service PHP fortement couplé :

```php
class OrderService {
    public function placeOrder(array $items): void {
        $pdo = new PDO('mysql:host=localhost;dbname=shop', 'root', '');
        $stmt = $pdo->prepare('INSERT INTO orders (items) VALUES (?)');
        $stmt->execute([json_encode($items)]);

        $mailer = new PHPMailer();
        $mailer->send(/* ... */);
    }
}
```

1. Identifie toutes les dépendances concrètes couplées en dur.
2. Introduis des interfaces (`OrderRepositoryInterface`, `MailerInterface`) et injecte-les par constructeur.
3. Écris un test qui appelle `placeOrder()` avec des implémentations en mémoire (aucune vraie base de données, aucun vrai email envoyé) pour vérifier qu'un ordre est bien enregistré.

## 2. Composition vs héritage — refactoring d'une hiérarchie rigide

Cette hiérarchie de classes JS modélise des employés :

```js
class Employee {
  work() { return "travaille"; }
  code() { throw new Error("not implemented"); }
  manage() { throw new Error("not implemented"); }
}
class Developer extends Employee {
  code() { return "code"; }
}
class Manager extends Employee {
  manage() { return "manage une équipe"; }
}
class TechLead extends Employee {
  code() { return "code"; }
  manage() { return "manage une équipe"; }
}
```

Ce design pose un problème quand un nouveau rôle combine des capacités différentes (ex: un `Freelancer` qui code ET fait de la gestion de projet ET fait de la facturation). Refactore ce système en composition (des comportements injectables : `CodingBehavior`, `ManagingBehavior`, etc.) pour que n'importe quelle combinaison de capacités soit possible sans exploser le nombre de classes.

## 3. Repository pattern + Factory combinés

Conçois (en pseudo-code ou dans le langage de ton choix) un système de notification qui :
- Définit une interface `NotificationRepositoryInterface` pour stocker l'historique des notifications envoyées (peu importe le support : SQL, fichier, mémoire)
- Utilise une `NotificationChannelFactory` qui retourne le bon canal (`EmailChannel`, `SmsChannel`, `PushChannel`) à partir d'une chaîne de configuration
- Respecte le Single Responsibility Principle : la Factory ne doit connaître que la création, le Repository que la persistance, les canaux que l'envoi

Justifie en 5 lignes pourquoi ce découpage rend le système plus facile à tester et à faire évoluer qu'une classe unique `NotificationManager` qui ferait tout.

## 4. Reconnaître la sur-ingénierie

Un développeur senior propose ce design pour un simple formulaire de contact (3 champs : nom, email, message, un seul type de traitement possible — envoyer un email) :

```
ContactFormFactory → crée → ContactFormStrategy (interface)
  → EmailContactFormStrategy implements ContactFormStrategy
ContactFormValidatorChainOfResponsibility (5 validateurs enchaînés)
ContactFormRepository (persiste en base avant envoi, jamais relu ensuite)
ContactFormObserver (notifie 3 listeners qui ne font rien pour l'instant)
```

En te basant sur YAGNI et la Rule of Three (vues dans le README), explique en quelques phrases quels éléments de ce design sont probablement de la sur-ingénierie pour ce cas précis, et ce que tu garderais à la place. Il n'y a pas de solution unique : l'objectif est de justifier ton raisonnement.
