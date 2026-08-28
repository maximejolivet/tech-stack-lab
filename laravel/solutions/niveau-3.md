# Solutions — Niveau 3

## Exercice 1

Avant (fat controller) : validation, création, envoi d'email et log tout mélangés dans `store()`.

Après :

```php
class CreateBook
{
    public function __construct(private BookRepository $books) {}

    public function handle(array $data): Book
    {
        $book = $this->books->create($data);

        SendAuthorNotification::dispatch($book); // job, voir exercice 2

        Log::info('Book created', ['book_id' => $book->id]);

        return $book;
    }
}

class BookController extends Controller
{
    public function store(StoreBookRequest $request, CreateBook $action): RedirectResponse
    {
        $action->handle($request->validated());
        return redirect()->route('books.index');
    }
}
```

Justification : le contrôleur ne fait plus que orchestrer (valider via le Form Request, déléguer à l'action, répondre) — la logique métier (création + effets de bord) est testable indépendamment du HTTP, réutilisable ailleurs (une commande Artisan, un autre contrôleur), et le contrôleur reste lisible même si la logique métier grossit.

## Exercice 2

```php
class SendAuthorNotification implements ShouldQueue
{
    use Queueable;

    public function __construct(private Book $book) {}

    public function handle(): void
    {
        Mail::to($this->book->author->email)->send(new NewBookMail($this->book));
    }
}
```

Dispatché (`SendAuthorNotification::dispatch($book)`), l'envoi d'email sort du cycle requête/réponse : l'utilisateur reçoit sa réponse HTTP immédiatement (redirection) sans attendre que l'email parte réellement, ce qui rend l'action perçue comme quasi-instantanée. Pour que le job soit réellement traité, il faut qu'un worker tourne en continu (`php artisan queue:work`), généralement supervisé par un process manager (Supervisor) ou Horizon en production — sans worker actif, le job reste en attente indéfiniment dans la table/le driver de queue.

## Exercice 3

```php
class BookResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'author_name' => $this->author->name,
        ];
    }
}
```

Pour une v2 qui doit exposer `pages` en plus sans casser la v1 : créer une route/namespace versionné (`/api/v2/books`) avec une `BookResourceV2` distincte (ou la même Resource avec un champ conditionnel via `$request->route()->getPrefix()` — moins propre). L'approche la plus sûre et la plus lisible reste des classes Resource séparées par version, chacune stable dans le temps, plutôt que de faire varier dynamiquement la sortie d'une seule Resource selon le contexte.

## Exercice 4

Démarche : activer `Model::preventLazyLoading()` en dev pour faire remonter les accès lazy immédiatement, ou inspecter les requêtes générées via Laravel Debugbar / Telescope / `DB::listen()` sur la page en question.

Causes les plus probables :
1. **N+1 sur la relation `books`** : `Author::all()` puis accès à `$author->books` dans la vue déclenche une requête par auteur → corriger avec `Author::with('books')->get()`.
2. **Colonnes manquantes en index** : si le tri/filtre se fait sur une colonne non indexée (ex. `author.name`), la requête peut scanner toute la table même à faible volume — vérifier avec `EXPLAIN`.
3. **Trop de données chargées** : sélectionner toutes les colonnes (`SELECT *`) alors que seules 2-3 sont affichées — utiliser `select()` pour ne charger que le nécessaire, surtout si des colonnes volumineuses (texte long, JSON) sont présentes.
