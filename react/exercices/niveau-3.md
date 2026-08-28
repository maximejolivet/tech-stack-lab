# Exercices React — Niveau 3 (Avancé)

## Exercice 1 — Mémoïsation ciblée

Tu as un composant `ProductList` qui affiche 10 000 produits filtrables. Le filtrage (`items.filter(...)`) recalculait à chaque frappe dans un champ de recherche non lié aux produits (ex: un champ "commentaire" sur la même page), provoquant un lag perceptible. Identifie où placer `useMemo` (et pourquoi), et explique pourquoi ajouter `useMemo` partout dans l'app ne serait PAS une bonne pratique.

## Exercice 2 — Suspense + lazy loading

Découpe une app à 3 pages (`Home`, `Settings`, `Admin`) en chargement à la demande avec `React.lazy` et `Suspense`, avec un fallback de chargement commun. Explique en une phrase le gain attendu sur le bundle initial.

## Exercice 3 — Compound component

Implémente un composant `Tabs` en pattern "compound component" : `<Tabs><Tabs.List><Tabs.Tab id="a">Onglet A</Tabs.Tab></Tabs.List><Tabs.Panel id="a">Contenu A</Tabs.Panel></Tabs>`. L'état de l'onglet actif doit être géré en interne via Context, sans prop à passer manuellement entre `Tabs.Tab` et `Tabs.Panel`.

## Exercice 4 — Test de comportement

Écris un test avec React Testing Library pour le composant `Counter` de l'exercice 2 du niveau 1 : vérifie qu'un clic sur "+1" fait passer l'affichage de "0" à "1", **sans** accéder à l'état interne du composant (teste uniquement ce que voit l'utilisateur).

## Exercice 5 — Diagnostic de re-render inutile

Un composant enfant `<ExpensiveChart data={data} />` se re-render à chaque frappe dans un champ de recherche du parent, alors que `data` n'a pas changé. Explique pourquoi (indice : la référence de la prop passée), et propose une correction avec `React.memo` + `useCallback`/`useMemo` selon le cas.
