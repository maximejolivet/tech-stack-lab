# Solutions — Niveau 1

## Exercice 1

```php
// routes/web.php
use App\Http\Controllers\GreetingController;

Route::get('/hello/{name}', [GreetingController::class, 'show']);
```

```php
class GreetingController extends Controller
{
    public function show(string $name)
    {
        return "Bonjour, {$name} !";
    }
}
```

## Exercice 2

```blade
{{-- resources/views/greeting.blade.php --}}
@if (count($names) === 0)
    <p>Aucun prénom</p>
@else
    <ul>
        @foreach ($names as $name)
            <li>{{ $name }}</li>
        @endforeach
    </ul>
@endif
```

## Exercice 3

```php
// database/migrations/xxxx_create_books_table.php
Schema::create('books', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->integer('pages');
    $table->boolean('read')->default(false);
    $table->timestamps();
});
```

```php
class Book extends Model
{
    protected $fillable = ['title', 'pages', 'read'];
}
```

```php
// php artisan tinker
Book::create(['title' => 'Clean Code', 'pages' => 464]);
```

## Exercice 4

`env('APP_NAME')` lit directement le fichier `.env` à chaque appel. En production, on exécute généralement `php artisan config:cache`, qui fige la config dans un fichier compilé unique et fait que `env()` retourne `null` en dehors des fichiers `config/*.php` (Laravel ne relit plus `.env` après ce cache). L'alternative correcte est de définir la valeur dans `config/app.php` (`'name' => env('APP_NAME', 'Laravel')`) et de la lire partout ailleurs via `config('app.name')`, qui reste fiable que la config soit cachée ou non.
