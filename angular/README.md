# Angular

## 1. Introduction

Angular est un framework front-end complet et "opinionated", écrit en TypeScript, maintenu par Google. Contrairement à React ([`../react/`](../react/)) ou Vue ([`../vuejs/`](../vuejs/)) qui sont des bibliothèques centrées sur l'UI, Angular fournit nativement routing, injection de dépendances, formulaires, HTTP client et outillage de build — une approche "batteries included" proche dans l'esprit de Symfony ([`../symfony/`](../symfony/)) ou Spring Boot ([`../springboot/`](../springboot/)) côté back.

**À quoi sert-il ?**
- Construire des SPA (Single Page Applications) et des applications d'entreprise structurées, où la cohérence architecturale prime sur la liberté de choix.
- Fournir un cadre complet (DI, routing, forms, HTTP) sans assembler de librairies tierces disparates.

**Où se situe-t-il dans une architecture web ?** Côté client, comme React/Vue. Angular peut aussi faire du rendu serveur (Angular Universal/SSR) mais ce dossier couvre l'usage SPA standard.

**Avantages**
- Structure imposée et cohérente : DI, modules/standalone, conventions de nommage — un projet Angular ressemble à un autre projet Angular, peu importe l'équipe (proche de la discipline imposée par Symfony côté back).
- TypeScript natif et obligatoire (contrairement à React/Vue où il est optionnel) : le typage fort est garanti sur tout le projet.
- Outillage complet officiel (Angular CLI) : génération de code, build, tests, lint intégrés sans configuration à assembler.

**Limites**
- Courbe d'apprentissage plus raide que React/Vue : décorateurs, DI, RxJS et Zone.js sont des concepts supplémentaires à maîtriser.
- Plus verbeux pour de petits projets — l'"opinionated-ness" qui aide à grande échelle est un coût pour un prototype rapide.
- RxJS (programmation réactive) est quasi incontournable dès qu'on dépasse les bases, contrairement à React/Vue où l'état asynchrone reste gérable avec des Promises simples.

## 2. Prérequis

- TypeScript solide ([`../typescript/`](../typescript/)) : Angular s'appuie massivement sur les décorateurs, les interfaces et les génériques.
- Avoir déjà pratiqué React ou Vue aide à situer les différences (composants, data binding, gestion d'état) plutôt que de découvrir ces notions à zéro.
- Node.js et npm installés pour l'Angular CLI.

## 3. Rappel des bases 🟢

### 01 - Créer un projet Angular

**Explication** — Le CLI officiel (`@angular/cli`) génère la structure complète, gère le build (esbuild depuis Angular 17) et le serveur de dev.

```bash
npm install -g @angular/cli
ng new mon-app
cd mon-app && ng serve
```

**Bonne pratique** : utiliser systématiquement le CLI pour générer composants/services (`ng generate component`) plutôt que créer les fichiers à la main — garantit une structure et un nommage cohérents dans toute l'équipe.

### 02 - Components (standalone)

**Explication** — Un composant Angular combine un décorateur `@Component` (métadonnées : sélecteur, template, styles), une classe TypeScript (logique), et un template HTML. Depuis Angular 17, les composants **standalone** (sans `NgModule`) sont la valeur par défaut recommandée.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-greeting',
  standalone: true,
  template: `<h1>Bonjour, {{ name }} !</h1>`,
})
export class GreetingComponent {
  name = 'Max';
}
```

**Cas d'usage** : `selector: 'app-greeting'` permet d'utiliser ce composant ailleurs via `<app-greeting />` — équivalent conceptuel d'une balise de composant React/Vue.

### 03 - Data binding

**Explication** — Angular propose quatre formes de liaison entre la classe et le template : interpolation (`{{ }}`, affichage), property binding (`[prop]`, passer une donnée vers le DOM/un enfant), event binding (`(event)`, réagir à un événement), two-way binding (`[(ngModel)]`, combinaison des deux sur les formulaires).

```html
<h1>{{ title }}</h1>                              <!-- interpolation -->
<img [src]="imageUrl" [alt]="imageAlt" />           <!-- property binding -->
<button (click)="increment()">+1</button>             <!-- event binding -->
<input [(ngModel)]="username" />                        <!-- two-way binding -->
```

**Erreur fréquente** : oublier d'importer `FormsModule` pour utiliser `[(ngModel)]` — contrairement à l'interpolation/property/event binding disponibles nativement, le two-way binding sur formulaire nécessite cet import explicite.

### 04 - Directives de contrôle de flux (@if, @for)

**Explication** — Depuis Angular 17, la syntaxe native `@if`/`@for`/`@switch` remplace les anciennes directives structurelles `*ngIf`/`*ngFor` — plus proche syntaxiquement du JSX conditionnel de React, plus performante (pas de dépendance à `CommonModule`).

```html
@if (isLoggedIn) {
  <p>Bienvenue !</p>
} @else {
  <a routerLink="/login">Se connecter</a>
}

@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
}
```

**Erreur fréquente** : oublier `track` dans un `@for` — comme la `key` de React (voir [`../react/`](../react/)), `track` indique à Angular comment identifier chaque élément entre deux rendus pour éviter de tout recréer inutilement.

### 05 - Services et injection de dépendances

**Explication** — Un service est une classe annotée `@Injectable` qui encapsule une logique réutilisable (appel HTTP, état partagé). Angular l'instancie et l'injecte automatiquement dans les composants qui le déclarent en constructeur — pattern d'Inversion de Contrôle proche de l'injection de services Symfony/Spring.

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  getUsers() {
    return fetch('/api/users').then(r => r.json());
  }
}

@Component({ /* ... */ })
export class UserListComponent {
  constructor(private userService: UserService) {}  // injecté automatiquement
}
```

**Bonne pratique** : `providedIn: 'root'` rend le service singleton à l'échelle de l'application — préférer cette forme à une déclaration manuelle dans les providers d'un composant, sauf besoin explicite d'une instance scopée.

### 06 - Routing

**Explication** — Le `Router` Angular associe une URL à un composant, configuré de façon déclarative — équivalent de React Router ou Vue Router.

```typescript
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users/:id', component: UserDetailComponent },
];
```

```html
<router-outlet />  <!-- zone où le composant de la route active s'affiche -->
<a routerLink="/users/42">Voir l'utilisateur 42</a>
```

**Bonne pratique** : utiliser `routerLink` plutôt que `href` pour la navigation interne — évite un rechargement complet de la page (comportement SPA préservé).

## 4. Concepts intermédiaires 🟡

- **RxJS et Observables** : Angular s'appuie massivement sur RxJS pour représenter des flux de données asynchrones (HTTP, événements, formulaires) — un `Observable` est comparable à une Promise, mais peut émettre plusieurs valeurs dans le temps et propose un riche catalogue d'opérateurs de transformation.

```typescript
import { map, filter } from 'rxjs/operators';

this.userService.getUsers$().pipe(
  filter(users => users.length > 0),
  map(users => users.map(u => u.name)),
).subscribe(names => console.log(names));
```

- **HttpClient** : client HTTP intégré, retourne des `Observable` plutôt que des Promises — nécessite un `subscribe()` explicite (ou l'`async` pipe côté template) pour déclencher la requête.

```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  constructor(private http: HttpClient) {}
  getUsers() {
    return this.http.get<User[]>('/api/users');  // Observable, rien ne part tant que non "subscribed"
  }
}
```

- **Async pipe** : `| async` dans le template s'abonne automatiquement à un `Observable` et se désabonne à la destruction du composant — évite les fuites mémoire d'un `subscribe()` manuel oublié.

```html
<ul>
  @for (user of users$ | async; track user.id) {
    <li>{{ user.name }}</li>
  }
</ul>
```

- **Reactive Forms** : approche typée et pilotée par le code (`FormGroup`, `FormControl`, validateurs) — alternative aux Template-Driven Forms (`[(ngModel)]`), recommandée pour des formulaires complexes avec validation.

```typescript
form = new FormGroup({
  email: new FormControl('', [Validators.required, Validators.email]),
  password: new FormControl('', Validators.minLength(8)),
});
```

- **Lifecycle hooks** : méthodes d'interface (`ngOnInit`, `ngOnDestroy`, `ngOnChanges`) appelées à des moments précis du cycle de vie d'un composant — `ngOnInit` pour l'initialisation (proche du `useEffect(() => {}, [])` de React), `ngOnDestroy` pour le nettoyage (désabonnements manuels, timers).

```typescript
export class TimerComponent implements OnInit, OnDestroy {
  private subscription?: Subscription;
  ngOnInit() { this.subscription = interval(1000).subscribe(...); }
  ngOnDestroy() { this.subscription?.unsubscribe(); }
}
```

- **Pipes** : transforment une valeur directement dans le template (`{{ value | pipe }}`) — pipes natifs (`date`, `currency`, `uppercase`) ou custom via `@Pipe`.

```html
{{ price | currency:'EUR' }}
{{ publishedAt | date:'dd/MM/yyyy' }}
```

## 5. Concepts avancés 🟠🔴

- **Signals** (Angular 16+) : nouveau modèle de réactivité à granularité fine, alternative à Zone.js pour la détection de changement — une valeur `signal()` notifie précisément les consommateurs qui en dépendent, sans recalculer tout l'arbre de composants. Tend à devenir le standard, en complément voire en remplacement progressif de RxJS pour l'état local.

```typescript
export class CounterComponent {
  count = signal(0);
  doubled = computed(() => this.count() * 2);  // dérivé, recalculé seulement si count change
  increment() { this.count.update(c => c + 1); }
}
```

```html
<p>{{ count() }} — doublé : {{ doubled() }}</p>
```

- **Change detection et Zone.js** : par défaut, Angular utilise Zone.js pour intercepter automatiquement les événements asynchrones (clic, `setTimeout`, requête HTTP) et déclencher une vérification de tout l'arbre de composants. La stratégie `OnPush` restreint cette vérification aux composants dont les `@Input()` ont changé par référence — optimisation comparable à `React.memo`.

```typescript
@Component({ changeDetection: ChangeDetectionStrategy.OnPush, /* ... */ })
export class UserCardComponent {
  @Input() user!: User;
}
```

- **Lazy loading des routes** : charger le code d'une fonctionnalité seulement quand sa route est visitée, via `loadComponent`/`loadChildren` — réduit le bundle initial, équivalent de `React.lazy` + `Suspense`.

```typescript
{ path: 'admin', loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent) }
```

- **Content projection (`<ng-content>`)** : mécanisme de composition proche de `children` en React ou des slots en Vue — un composant parent peut injecter du contenu dans un composant enfant sans que ce dernier en connaisse le détail.
- **Migration NgModules → standalone** : les projets Angular historiques organisent leurs composants en `NgModule` (déclarations, imports, providers groupés) ; l'architecture standalone (par défaut depuis Angular 17) simplifie ce modèle en rendant chaque composant autonome — connaître les deux reste utile face à un codebase existant.

## 6. Commandes / syntaxe à connaître

```bash
ng new mon-app                       # créer un nouveau projet
ng serve                              # serveur de dev
ng generate component mon-composant    # générer un composant
ng generate service mon-service          # générer un service
ng build                                  # build de production
ng test                                    # lancer les tests unitaires
```

```typescript
@Component({ selector: 'app-x', standalone: true, template: `...` })
@Injectable({ providedIn: 'root' })
signal(initialValue)
computed(() => derive())
this.http.get<T>(url).pipe(map(...), filter(...))
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Gestionnaire de tâches avec service et routing**

Construire une petite application Angular qui doit :
- Modéliser une tâche (`interface Task { id: number; title: string; done: boolean }`).
- Créer un `TaskService` (`providedIn: 'root'`) qui expose les tâches via un `signal` ou un `Observable`.
- Un composant `TaskListComponent` qui affiche la liste (`@for` avec `track`) et permet de cocher/décocher une tâche.
- Un composant `TaskFormComponent` avec un Reactive Form pour ajouter une tâche (validation : titre non vide).
- Deux routes (`/tasks` et `/tasks/:id`) avec `router-outlet` et navigation via `routerLink`.

Objectif : mobiliser components, services/DI, signals ou RxJS, reactive forms et routing dans un exercice concret réalisable en quelques heures.

## Checklist

- [ ] Comprendre les fondamentaux (components, data binding, @if/@for)
- [ ] Savoir créer un projet Angular avec le CLI
- [ ] Maîtriser la syntaxe principale (services, DI, routing)
- [ ] Comprendre les concepts importants (RxJS, HttpClient, async pipe, reactive forms)
- [ ] Savoir debugger avec Angular DevTools
- [ ] Connaître les bonnes pratiques (OnPush, désabonnement, standalone components)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (signals, change detection, lazy loading)

## 10. Ressources

- [Documentation officielle Angular](https://angular.dev) — référence à jour (signals, standalone, nouvelle syntaxe de contrôle de flux).
- [RxJS — documentation officielle](https://rxjs.dev/) pour approfondir les opérateurs.
- [Angular DevTools](https://angular.dev/tools/devtools) — extension navigateur pour inspecter composants et profiler la détection de changement.
- [roadmap.sh — Angular](https://roadmap.sh/angular) — vue d'ensemble du parcours d'apprentissage.
