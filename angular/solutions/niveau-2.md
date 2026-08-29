# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```typescript
@Injectable({ providedIn: 'root' })
export class QuoteService {
  getQuotes() {
    return [
      'Le simple est la sophistication suprême.',
      'Fais simple, mais pas plus simple.',
      'La perfection est atteinte quand il n\'y a plus rien à retirer.',
    ];
  }
}

@Component({
  selector: 'app-quote-list',
  standalone: true,
  template: `
    <ul>
      @for (quote of quotes; track quote) {
        <li>{{ quote }}</li>
      }
    </ul>
  `,
})
export class QuoteListComponent {
  quotes: string[];
  constructor(private quoteService: QuoteService) {
    this.quotes = this.quoteService.getQuotes();
  }
}
```

## Exercice 2

```typescript
interface Post { id: number; title: string; body: string; }

@Injectable({ providedIn: 'root' })
export class PostService {
  constructor(private http: HttpClient) {}
  getPosts() {
    return this.http.get<Post[]>('https://jsonplaceholder.typicode.com/posts');
  }
}

@Component({
  selector: 'app-post-list',
  standalone: true,
  imports: [AsyncPipe],
  template: `
    <ul>
      @for (post of (posts$ | async)?.slice(0, 5); track post.id) {
        <li>{{ post.title }}</li>
      }
    </ul>
  `,
})
export class PostListComponent {
  posts$;
  constructor(private postService: PostService) {
    this.posts$ = this.postService.getPosts();
  }
}
```

## Exercice 3

```typescript
@Component({
  selector: 'app-login-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form">
      <input formControlName="email" placeholder="Email" />
      @if (form.controls.email.invalid && form.controls.email.touched) {
        <p>Email invalide</p>
      }
      <input formControlName="password" type="password" placeholder="Mot de passe" />
      @if (form.controls.password.invalid && form.controls.password.touched) {
        <p>Mot de passe requis (8 caractères minimum)</p>
      }
    </form>
  `,
})
export class LoginFormComponent {
  form = new FormGroup({
    email: new FormControl('', [Validators.required, Validators.email]),
    password: new FormControl('', [Validators.required, Validators.minLength(8)]),
  });
}
```

## Exercice 4

```typescript
@Component({
  selector: 'app-clock',
  standalone: true,
  template: `<p>{{ currentTime }}</p>`,
})
export class ClockComponent implements OnInit, OnDestroy {
  currentTime = new Date().toLocaleTimeString();
  private subscription?: Subscription;

  ngOnInit() {
    this.subscription = interval(1000).subscribe(() => {
      this.currentTime = new Date().toLocaleTimeString();
    });
  }

  ngOnDestroy() {
    this.subscription?.unsubscribe();
  }
}
```

## Exercice 5

```typescript
@Pipe({ name: 'truncate', standalone: true })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number): string {
    return value.length > limit ? value.slice(0, limit) + '...' : value;
  }
}
```

```html
<p>{{ longText | truncate:50 }}</p>
```
