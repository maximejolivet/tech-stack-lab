# Exercices Angular — Niveau 3 (Avancé)

## Exercice 1 — Signals et computed

Crée un composant `CartComponent` avec un `signal<CartItem[]>([])` pour les articles du panier, et un `computed()` qui calcule le total (`price * quantity` sommé). Ajoute une méthode `addItem` qui utilise `update()` pour ajouter un article de façon immutable.

## Exercice 2 — OnPush et immutabilité

Prends un composant `UserCardComponent` qui reçoit un `@Input() user: User` et passe sa stratégie à `ChangeDetectionStrategy.OnPush`. Démontre (en commentaire) pourquoi muter directement une propriété de `user` depuis le parent (`user.name = 'x'`) ne déclenche PAS de re-render avec OnPush, alors que réassigner une nouvelle référence (`this.user = { ...user, name: 'x' }`) le déclenche.

## Exercice 3 — Lazy loading de route

Configure une route `/admin` qui charge un `AdminComponent` via `loadComponent` uniquement quand elle est visitée. Vérifie dans l'onglet Network du navigateur que le chunk JS correspondant n'est chargé qu'à la navigation vers `/admin`, pas au chargement initial de l'app.

## Exercice 4 — Content projection

Crée un composant `CardComponent` avec un `<ng-content>` pour le contenu principal, et un second slot nommé (`<ng-content select="[card-footer]">`) pour un pied de carte optionnel. Utilise-le avec deux compositions différentes pour illustrer la flexibilité du pattern.
