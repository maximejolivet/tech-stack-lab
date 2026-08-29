# Exercices Accessibility & Performance — Niveau 3 (Avancé)

## Exercice 1 — Audit d'une page produit

Voici un extrait volontairement problématique d'une page produit :

```html
<div class="title">Casque audio sans fil</div>
<img src="casque.jpg">
<div class="price" style="color:#bbb">49,99 €</div>
<div class="add-to-cart" onclick="addToCart()">Ajouter au panier</div>
```

Liste tous les problèmes d'accessibilité présents (au moins 4), puis réécris ce bloc corrigé.

## Exercice 2 — Focus trapping dans une modale

La modale de confirmation d'ajout au panier s'ouvre au clic sur "Ajouter au panier" mais le focus clavier continue de circuler dans la page derrière elle (Tab atteint des liens masqués). Décris, en pseudo-code ou en JavaScript, la logique de focus trapping à mettre en place à l'ouverture et à la fermeture de la modale.

## Exercice 3 — Diagnostiquer un CLS

Un site a un score CLS élevé signalé par Lighthouse. Liste trois causes possibles de décalage de mise en page (layout shift) sur une page produit typique, et la correction associée à chacune.

## Exercice 4 — Code splitting par route

Une application React charge actuellement tout son code (y compris les pages Admin, rarement visitées) dans un seul bundle JavaScript initial. Explique comment réduire le poids du chargement initial pour un visiteur qui ne consulte que la page d'accueil, avec le code correspondant (voir aussi [`../react/`](../react/)).

## Exercice 5 — RUM vs mesure en laboratoire

Un site obtient un score Lighthouse de 95/100 en LCP, mais les données Real User Monitoring montrent qu'une part significative des utilisateurs mobiles subit un LCP supérieur à 4 secondes. Explique en 3-4 phrases pourquoi ces deux mesures peuvent diverger autant, et laquelle prioriser pour décider des optimisations à mener.
