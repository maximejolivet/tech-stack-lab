# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

1. **Red** : écrire `expect(isValidEmail("test@example.com")).toBe(true)` — le test échoue car `isValidEmail` n'existe pas encore.
2. **Green** : écrire l'implémentation minimale qui fait passer ce test (ex. retourner `true` inconditionnellement, ou une regex simple).
3. **Red** : ajouter un second cas, `expect(isValidEmail("pas-un-email")).toBe(false)` — échoue si l'implémentation était triviale.
4. **Green** : ajuster l'implémentation (vraie regex ou validation) pour satisfaire les deux cas.
5. **Refactor** : nettoyer l'implémentation (extraire la regex dans une constante nommée, par exemple) en gardant les deux tests au vert.
6. Répéter le cycle pour d'autres cas limites (email vide, sans domaine, avec sous-domaine...).

## Exercice 2

(a) Un test unitaire de `createOrder` isolerait le service de paiement (stub/mock retournant un résultat fixe) pour vérifier uniquement la logique de calcul du total et l'appel correct au service (bons arguments), rapidement et sans dépendance réseau.

(b) Un test d'intégration avec un vrai service de paiement sandbox vérifierait en plus des choses qu'aucun mock ne peut garantir : le format réel attendu par l'API du service de paiement, son comportement sur des cas d'erreur réels (carte refusée), et que l'intégration technique (authentification, sérialisation) fonctionne vraiment — un mock mal calibré pourrait masquer une intégration cassée.

## Exercice 3

À moyen terme, cette habitude vide le test de snapshot de sa valeur : l'équipe valide mécaniquement chaque nouveau snapshot sans vérifier s'il reflète un changement voulu ou une vraie régression visuelle, ce qui revient à désactiver le test tout en gardant l'illusion qu'il protège contre les régressions.

## Exercice 4

Le test appelle `divide(10, 2)` mais ne vérifie **aucun résultat** — il exécute la ligne (donc la "couvre") sans affirmer qu'elle produit la bonne valeur ; une implémentation buguée (`return a * b` par exemple) ferait passer ce test tout aussi bien.

```
test("divides two numbers", () => {
  expect(divide(10, 2)).toBe(5);
});
```

## Exercice 5

Remplacer le `sleep(1000)` fixe par une attente conditionnelle qui poll jusqu'à ce que la condition réelle soit remplie (ex. `await waitFor(() => expect(result).toBeDefined())`), avec un timeout maximal en filet de sécurité plutôt qu'un délai fixe supposé toujours suffisant — le test attend exactement le temps nécessaire, ni plus (lenteur inutile) ni moins (flakiness sur environnement lent).
