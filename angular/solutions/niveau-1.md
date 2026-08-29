# Solutions — Niveau 1 (Bases)

## Exercice 1

```typescript
@Component({
  selector: 'app-greeting',
  standalone: true,
  template: `<h1>Bonjour, {{ name }} !</h1>`,
})
export class GreetingComponent {
  name = 'Max';
}
```

## Exercice 2

```typescript
@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <button (click)="count = count - 1">-1</button>
    <span> {{ count }} </span>
    <button (click)="count = count + 1">+1</button>
    <button (click)="count = 0">Reset</button>
  `,
})
export class CounterComponent {
  count = 0;
}
```

## Exercice 3

```typescript
@Component({
  selector: 'app-fruit-list',
  standalone: true,
  template: `
    <ul>
      @for (item of items; track item.id) {
        <li>{{ item.name }}</li>
      }
    </ul>
  `,
})
export class FruitListComponent {
  items = [
    { id: 1, name: 'Pomme' },
    { id: 2, name: 'Banane' },
    { id: 3, name: 'Cerise' },
  ];
}
```

## Exercice 4

```typescript
@Component({
  selector: 'app-status-badge',
  standalone: true,
  template: `
    @if (isOnline) {
      <span>🟢 En ligne</span>
    } @else {
      <span>🔴 Hors ligne</span>
    }
  `,
})
export class StatusBadgeComponent {
  @Input() isOnline = false;
}
```

## Exercice 5

```typescript
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-username-form',
  standalone: true,
  imports: [FormsModule],
  template: `
    <input [(ngModel)]="username" />
    <p>Bonjour, {{ username }}</p>
  `,
})
export class UsernameFormComponent {
  username = '';
}
```
