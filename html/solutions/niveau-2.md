# Solutions HTML — Niveau 2

## 2.1 — Formulaire de contact complet

```html
<form action="/contact" method="post">
  <label for="name">Nom</label>
  <input type="text" id="name" name="name" required minlength="2" autocomplete="name">

  <label for="email">Email</label>
  <input type="email" id="email" name="email" required autocomplete="email">

  <label for="phone">Téléphone</label>
  <input type="tel" id="phone" name="phone" autocomplete="tel">

  <label for="subject">Sujet</label>
  <select id="subject" name="subject">
    <option value="support">Support</option>
    <option value="commercial">Commercial</option>
    <option value="autre">Autre</option>
  </select>

  <label for="message">Message</label>
  <textarea id="message" name="message" required maxlength="500"></textarea>

  <button type="submit">Envoyer</button>
</form>
```

## 2.2 — Tableau de données

```html
<table>
  <caption>Budget mensuel du foyer</caption>
  <thead>
    <tr>
      <th scope="col">Catégorie</th>
      <th scope="col">Prévu</th>
      <th scope="col">Réel</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Logement</th><td>800 €</td><td>800 €</td></tr>
    <tr><th scope="row">Courses</th><td>300 €</td><td>340 €</td></tr>
    <tr><th scope="row">Transport</th><td>100 €</td><td>85 €</td></tr>
    <tr><th scope="row">Loisirs</th><td>150 €</td><td>200 €</td></tr>
  </tbody>
</table>
```

## 2.3 — FAQ accessible sans JS

```html
<h2>FAQ</h2>

<details>
  <summary>Comment créer un compte ?</summary>
  <p>Cliquez sur "Inscription" en haut à droite...</p>
</details>

<details>
  <summary>Comment réinitialiser mon mot de passe ?</summary>
  <p>Utilisez le lien "Mot de passe oublié" sur la page de connexion.</p>
</details>

<details>
  <summary>Comment contacter le support ?</summary>
  <p>Via le formulaire de la page Contact.</p>
</details>
```

## 2.4 — `<head>` SEO/partage

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>10 astuces pour un HTML accessible — Mon Blog</title>
  <meta name="description" content="Découvrez 10 astuces concrètes pour rendre vos pages HTML plus accessibles, avec exemples et bonnes pratiques.">
  <link rel="canonical" href="https://monblog.fr/articles/html-accessible">

  <meta property="og:title" content="10 astuces pour un HTML accessible">
  <meta property="og:description" content="10 astuces concrètes, avec exemples, pour un HTML plus accessible.">
  <meta property="og:image" content="https://monblog.fr/images/article-html.jpg">

  <link rel="icon" href="/favicon.ico">
</head>
```

## 2.5 — Menu accessible avec ARIA

```html
<button aria-expanded="false" aria-controls="main-menu">Menu</button>
<ul id="main-menu" hidden>
  <li><a href="/">Accueil</a></li>
  <li><a href="/produits">Produits</a></li>
  <li><a href="/contact">Contact</a></li>
</ul>
```

Le JS (hors scope ici) devrait, au clic, basculer `hidden` sur le `<ul>` et mettre à jour `aria-expanded="true"/"false"` sur le bouton en conséquence.
