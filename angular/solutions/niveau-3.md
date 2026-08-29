# Solutions — Niveau 3 (Avancé)

## Exercice 1

```typescript
interface CartItem { id: number; price: number; quantity: number; }

@Component({
  selector: 'app-cart',
  standalone: true,
  template: `<p>Total : {{ total() }} €</p>`,
})
export class CartComponent {
  items = signal<CartItem[]>([]);
  total = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );

  addItem(item: CartItem) {
    this.items.update(current => [...current, item]);  // nouvelle référence, immutable
  }
}
```

## Exercice 2

```typescript
@Component({
  selector: 'app-user-card',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ user.name }}</p>`,
})
export class UserCardComponent {
  @Input() user!: User;
}
```

Avec `OnPush`, Angular ne redéclenche la détection de changement pour ce composant que si la **référence** de l'`@Input()` change entre deux rendus (comparaison `===`), pas si son contenu est muté :

```typescript
// ❌ Mutation directe : la référence `user` reste identique, OnPush ne détecte rien, le template ne se met pas à jour
this.user.name = 'x';

// ✅ Nouvelle référence : OnPush détecte le changement de référence et re-render le composant
this.user = { ...this.user, name: 'x' };
```

## Exercice 3

```typescript
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'admin', loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent) },
];
```

Dans l'onglet Network, le chunk correspondant à `admin.component` (visible dans la liste des fichiers JS chargés, nommé selon la config du bundler, ex. `chunk-ADMIN.js`) n'apparaît qu'après avoir navigué vers `/admin` — au chargement initial de l'app, seul le bundle principal (incluant `HomeComponent`) est téléchargé.

## Exercice 4

```typescript
@Component({
  selector: 'app-card',
  standalone: true,
  template: `
    <div class="card">
      <div class="card-body"><ng-content /></div>
      <div class="card-footer"><ng-content select="[card-footer]" /></div>
    </div>
  `,
})
export class CardComponent {}
```

```html
<!-- Composition 1 : contenu simple, pas de footer -->
<app-card>
  <p>Contenu principal.</p>
</app-card>

<!-- Composition 2 : contenu + footer projeté séparément -->
<app-card>
  <p>Contenu principal.</p>
  <div card-footer>
    <button>Action</button>
  </div>
</app-card>
```
