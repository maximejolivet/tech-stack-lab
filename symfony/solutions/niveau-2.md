# Solutions — Niveau 2

## Exercice 1

```php
namespace App\Service;

class SlugGenerator
{
    public function generate(string $title): string
    {
        $slug = iconv('UTF-8', 'ASCII//TRANSLIT', $title);
        $slug = strtolower($slug);
        $slug = preg_replace('/[^a-z0-9]+/', '-', $slug);

        return trim($slug, '-');
    }
}
```

```php
class ArticleController extends AbstractController
{
    public function __construct(private readonly SlugGenerator $slugGenerator) {}

    #[Route('/slug', name: 'slug_preview')]
    public function preview(Request $request): Response
    {
        $title = $request->query->get('title', '');

        return new Response($this->slugGenerator->generate($title));
    }
}
```

Pas besoin de déclarer le service manuellement : `SlugGenerator` est autoconfiguré/autowiré par défaut (services.yaml `autowire: true`).

## Exercice 2

```bash
php bin/console make:entity Book
# title: string, isbn: string, publishedAt: datetime_immutable
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

```php
class BookRepository extends ServiceEntityRepository
{
    public function findPublishedAfter(\DateTimeInterface $date): array
    {
        return $this->createQueryBuilder('b')
            ->andWhere('b.publishedAt > :date')
            ->setParameter('date', $date)
            ->orderBy('b.publishedAt', 'ASC')
            ->getQuery()
            ->getResult();
    }
}
```

## Exercice 3

```php
class BookType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title', TextType::class, [
                'constraints' => [new Assert\NotBlank()],
            ])
            ->add('isbn', TextType::class, [
                'constraints' => [new Assert\Regex('/^(97(8|9))?\d{9}(\d|X)$/')],
            ]);
    }
}
```

```twig
{{ form_start(form) }}
  {{ form_row(form.title) }}
  {{ form_errors(form.title) }}
  {{ form_row(form.isbn) }}
{{ form_end(form) }}
```

## Exercice 4

Boucler `$article->getAuthor()->getName()` sans jointure déclenche une requête SQL par article (Doctrine charge l'auteur en lazy loading à chaque accès) : N articles → N+1 requêtes. Correction :

```php
public function findAllWithAuthor(): array
{
    return $this->createQueryBuilder('a')
        ->leftJoin('a.author', 'author')
        ->addSelect('author')
        ->getQuery()
        ->getResult();
}
```

`addSelect('author')` force Doctrine à hydrater l'auteur dans la même requête (un seul `JOIN` SQL), éliminant les N requêtes supplémentaires.
