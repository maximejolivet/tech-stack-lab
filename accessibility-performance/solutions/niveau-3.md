# Solutions — Niveau 3 (Avancé)

## Exercice 1

Problèmes : (1) `.title` en `<div>` au lieu d'un vrai titre (`<h1>`/`<h2>`), non identifié comme titre par un lecteur d'écran ; (2) l'image n'a pas d'attribut `alt` ; (3) le prix en `#bbb` (gris clair) sur fond blanc a probablement un contraste insuffisant ; (4) le bouton "Ajouter au panier" est un `<div>` cliquable, ni focusable ni annoncé comme actionnable.

```html
<h1>Casque audio sans fil</h1>
<img src="casque.jpg" alt="Casque audio sans fil noir, vue de face">
<p class="price">49,99 €</p>
<button class="add-to-cart" onclick="addToCart()">Ajouter au panier</button>
```

(le prix passe à une couleur suffisamment contrastée, ex. `#333` sur fond blanc)

## Exercice 2

```
À l'ouverture de la modale :
  - Mémoriser l'élément actuellement focus (le bouton déclencheur)
  - Déplacer le focus sur le premier élément focusable de la modale
  - Intercepter Tab/Shift+Tab pour boucler uniquement parmi les éléments focusables de la modale
    (si focus sur le dernier élément + Tab → revenir au premier ; si focus sur le premier + Shift+Tab → aller au dernier)
  - Intercepter la touche Échap pour fermer la modale

À la fermeture de la modale :
  - Restaurer le focus sur l'élément mémorisé (le bouton déclencheur)
```

## Exercice 3

1. **Images sans dimensions déclarées** — corrigé en ajoutant `width`/`height` ou `aspect-ratio` en CSS pour réserver l'espace avant chargement.
2. **Police web qui remplace une police de secours de taille différente une fois chargée** — corrigé en dimensionnant la police de secours pour occuper un espace proche de la police finale (`size-adjust` ou choix d'une police de fallback à métriques proches).
3. **Contenu injecté dynamiquement au-dessus du contenu existant** (ex. une bannière de cookies ou publicité chargée après coup sans espace réservé) — corrigé en réservant l'espace dès le rendu initial (placeholder de la bonne hauteur) plutôt que d'insérer le contenu après coup.

## Exercice 4

Utiliser le lazy loading de composant (`React.lazy` + `Suspense`) pour charger le code de la section Admin uniquement quand sa route est visitée, au lieu de l'inclure dans le bundle initial.

```tsx
const AdminPage = React.lazy(() => import('./AdminPage'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/admin" element={<AdminPage />} />
      </Routes>
    </Suspense>
  );
}
```

Un visiteur qui ne consulte que `/` ne télécharge jamais le code de `AdminPage`, réduisant le bundle JavaScript initial et donc le temps avant interactivité.

## Exercice 5

Lighthouse mesure en laboratoire, dans des conditions réseau/appareil simulées et fixes (souvent proches d'un environnement favorable), tandis que le RUM mesure les conditions réelles très hétérogènes des visiteurs (appareils bas de gamme, réseau mobile instable, connexions saturées). Un site peut être objectivement rapide dans les conditions standardisées du lab tout en étant lent pour une part réelle d'utilisateurs sur des conditions défavorables non représentées par le test synthétique. Il faut prioriser les données RUM pour décider des optimisations, car elles reflètent l'expérience réellement vécue par les utilisateurs, en particulier ceux les plus pénalisés — le score Lighthouse reste utile comme indicateur de régression continue en CI, mais pas comme mesure définitive de l'expérience réelle.
