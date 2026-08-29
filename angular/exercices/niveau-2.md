# Exercices Angular — Niveau 2 (Intermédiaire)

## Exercice 1 — Service et injection

Crée un `QuoteService` (`providedIn: 'root'`) avec une méthode `getQuotes()` qui retourne un tableau de citations en dur. Injecte-le dans un composant `QuoteListComponent` qui affiche la liste.

## Exercice 2 — HttpClient et async pipe

Crée un service `PostService` avec une méthode `getPosts()` qui appelle `this.http.get<Post[]>('https://jsonplaceholder.typicode.com/posts')` et retourne l'`Observable`. Affiche les 5 premiers résultats dans un composant en utilisant le pipe `| async` directement dans le template (pas de `subscribe()` manuel).

## Exercice 3 — Reactive Form avec validation

Crée un formulaire de connexion (`FormGroup` avec `email` et `password`) utilisant `Validators.required` et `Validators.email`. Affiche un message d'erreur sous chaque champ si invalide et touché (`control.invalid && control.touched`).

## Exercice 4 — Lifecycle hooks

Crée un composant `ClockComponent` qui affiche l'heure courante, mise à jour chaque seconde via `interval(1000)` (RxJS) souscrit dans `ngOnInit`. Désabonne-toi correctement dans `ngOnDestroy` pour éviter une fuite mémoire.

## Exercice 5 — Pipe custom

Crée un pipe custom `truncate` qui coupe une chaîne à N caractères (paramètre) et ajoute "..." si elle a été tronquée. Utilise-le dans un template : `{{ longText | truncate:50 }}`.
