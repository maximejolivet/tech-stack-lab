# CSS — Solutions niveau 1

## 1.1 — Sélecteurs et spécificité

```css
p { color: black; }        /* (0,0,1) */
.texte { color: blue; }    /* (0,1,0) */
#intro { color: red; }     /* (1,0,0) */
```

C'est la règle `#intro` qui gagne : un id a une spécificité (1,0,0), toujours supérieure à une classe (0,1,0) ou un élément (0,0,1), indépendamment de l'ordre d'écriture dans le fichier.

## 1.2 — Box model

```css
.encart {
  box-sizing: border-box; /* clé : padding et border sont inclus dans les 300px */
  width: 300px;
  padding: 20px;
  border: 2px solid #333;
}
```

Sans `box-sizing: border-box`, le comportement par défaut (`content-box`) aurait donné une largeur totale de `300 + 2*20 + 2*2 = 344px`.

## 1.3 — Flexbox simple

```html
<nav class="navbar">
  <img class="navbar__logo" src="logo.svg" alt="Logo">
  <ul class="navbar__links">
    <li><a href="#">Accueil</a></li>
    <li><a href="#">Produits</a></li>
    <li><a href="#">Contact</a></li>
  </ul>
</nav>
```

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
}
.navbar__links {
  display: flex;
  gap: 24px;
  list-style: none;
}
```

## 1.4 — Responsive mobile-first

```css
.titre {
  font-size: 1.5rem;
}
@media (min-width: 768px) {
  .titre {
    font-size: 2.5rem;
  }
}
```

## 1.5 — Display

```css
/* .mon-lien est un <a> par défaut : display: inline.
   Les éléments inline ignorent width/height car ils s'insèrent
   dans le flux du texte et n'ont pas de "boîte" au sens du box model. */
.mon-lien {
  display: inline-block; /* ou block selon le besoin de layout */
  width: 200px;
  height: 50px;
}
```
