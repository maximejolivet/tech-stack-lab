# Exercices Testing — Niveau 1 (Bases)

## Exercice 1 — La pyramide de tests

Explique en 2-3 phrases pourquoi une suite de tests composée à 80% de tests E2E et 20% de tests unitaires (pyramide inversée) pose problème, même si elle "couvre" bien l'application.

## Exercice 2 — Arrange-Act-Assert

Réécris ce test (qui mélange tout) en respectant la structure Arrange-Act-Assert, avec des commentaires marquant chaque section :

```
test("stack works", () => {
  const s = new Stack();
  expect(s.isEmpty()).toBe(true);
  s.push(1); s.push(2);
  expect(s.pop()).toBe(2);
});
```

## Exercice 3 — Nommer un test

Voici trois noms de tests : `test_getTotal`, `test1`, `calculates total with multiple items`. Explique lequel est le meilleur nom et pourquoi, en te basant sur le critère de lisibilité.

## Exercice 4 — Identifier un test doubles

Un test remplace un service d'envoi d'email par un objet qui enregistre chaque appel reçu (destinataire, sujet) sans envoyer de vrai email, et le test vérifie ensuite qu'il a bien été appelé une fois avec le bon destinataire. S'agit-il d'un stub, d'un mock, d'un spy ou d'un fake ? Justifie.

## Exercice 5 — Isolation des tests

Deux tests dans le même fichier échouent uniquement quand ils s'exécutent dans un certain ordre, mais passent individuellement. Explique la cause la plus probable et comment la corriger.
