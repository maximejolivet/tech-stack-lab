# Exercices — Niveau 2 : Intermédiaire

## 1. Implémenter un Strategy

Une plateforme e-commerce calcule une remise selon le type de client : `standard` (0%), `premium` (10%), `vip` (20%). Le code actuel :

```js
function applyDiscount(customerType, amount) {
  if (customerType === "standard") return amount;
  if (customerType === "premium") return amount * 0.9;
  if (customerType === "vip") return amount * 0.8;
}
```

Le métier annonce qu'un quatrième type (`partner`, -15%) arrive le mois prochain, et que d'autres suivront. Refactore ce code avec le pattern Strategy pour que l'ajout d'un type de remise ne modifie pas le code existant (respect de l'OCP).

## 2. Implémenter un Observer / event bus

En PHP ou en JS, implémente un petit système d'événements permettant :
- de s'abonner à un événement nommé (`on(event, callback)`)
- d'émettre un événement avec des données (`emit(event, payload)`)
- que plusieurs abonnés indépendants réagissent au même événement (ex : `order.created` déclenche à la fois un email de confirmation ET une mise à jour de stock, dans deux fonctions séparées qui ne se connaissent pas)

## 3. Appliquer l'Interface Segregation Principle

Cette interface TypeScript est utilisée par trois types de comptes (`AdminAccount`, `CustomerAccount`, `GuestAccount`) mais tous n'ont pas besoin de toutes les méthodes :

```ts
interface Account {
  login(): void;
  logout(): void;
  manageUsers(): void;   // seul Admin en a besoin
  viewOrders(): void;    // Admin et Customer, pas Guest
}
```

Redécoupe cette interface en plusieurs interfaces plus petites, puis indique laquelle (ou lesquelles) chaque type de compte doit implémenter.

## 4. Décorer sans modifier

Tu as une classe `Coffee` avec une méthode `cost(): number` qui retourne `2`. Sans modifier `Coffee`, utilise le pattern Decorator pour permettre de composer dynamiquement des extras (`+ lait` : +0.5, `+ sucre` : +0.2) et empiler plusieurs décorateurs sur un même café.
