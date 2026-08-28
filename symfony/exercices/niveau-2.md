# Exercices — Niveau 2 (Intermédiaire)

## Exercice 1 — Service injecté

Crée un service `SlugGenerator` (méthode `generate(string $title): string`, utilisant `iconv`/`preg_replace` pour produire un slug). Injecte-le dans un contrôleur par constructeur et utilise-le pour générer le slug d'un titre reçu en query string.

## Exercice 2 — Entité et migration

Crée une entité `Book` (`title`, `isbn`, `publishedAt`), génère la migration correspondante, applique-la, puis crée un `BookRepository` avec une méthode custom `findPublishedAfter(\DateTimeInterface $date): array`.

## Exercice 3 — Formulaire avec validation

Crée un `BookType` (FormType) pour l'entité `Book`, avec `#[Assert\NotBlank]` sur `title` et une contrainte de format ISBN sur `isbn`. Affiche les erreurs de validation dans le template en cas de soumission invalide.

## Exercice 4 — N+1 à corriger

On te donne (mentalement ou en code) une liste d'articles avec leurs auteurs, affichée en bouclant `$article->getAuthor()->getName()` dans Twig sans jointure. Explique pourquoi c'est un problème de performance et réécris la méthode du repository pour le corriger avec `leftJoin` + `addSelect`.
