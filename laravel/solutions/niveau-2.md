# Solutions — Niveau 2

## Exercice 1

```php
class Author extends Model
{
    public function books(): HasMany
    {
        return $this->hasMany(Book::class);
    }
}

class Book extends Model
{
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }
}
```

```php
Route::get('/authors', function () {
    $authors = Author::withCount('books')->get(); // 1 seule requête, avec un COUNT groupé en SQL
    return view('authors.index', ['authors' => $authors]);
});
```

`withCount('books')` génère une seule requête SQL avec une sous-requête `COUNT`, contrairement à `Author::all()` puis `$author->books->count()` dans une boucle qui déclencherait une requête par auteur (N+1). C'est le choix optimal ici puisqu'on n'a besoin que du nombre, pas des livres eux-mêmes.

## Exercice 2

```php
class StoreBookRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'pages' => 'required|integer|min:1',
            'author_id' => 'required|exists:authors,id',
        ];
    }
}

public function store(StoreBookRequest $request): RedirectResponse
{
    Book::create($request->validated());
    return redirect()->route('books.index');
}
```

## Exercice 3

```php
class EnsureBookLimit
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user()->books()->count() >= 10) {
            return back()->withErrors(['limit' => 'Limite de 10 livres atteinte.']);
        }
        return $next($request);
    }
}
```

```php
// routes/web.php
Route::post('/books', [BookController::class, 'store'])
    ->middleware(['auth', EnsureBookLimit::class]);
```

## Exercice 4

```php
public function test_guest_cannot_create_book(): void
{
    $response = $this->post('/books', ['title' => 'Test', 'pages' => 100]);
    $response->assertRedirect('/login'); // ou assertStatus(401) selon la config d'auth
}

public function test_authenticated_user_can_create_book(): void
{
    $user = User::factory()->create();
    $author = Author::factory()->create();

    $response = $this->actingAs($user)->post('/books', [
        'title' => 'Clean Architecture',
        'pages' => 432,
        'author_id' => $author->id,
    ]);

    $response->assertRedirect('/books');
    $this->assertDatabaseHas('books', ['title' => 'Clean Architecture']);
}
```
