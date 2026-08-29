# Accessibility & Performance

## 1. Introduction

Ce dossier est une **synthèse transverse côté front** : accessibilité (a11y) et performance ne sont pas des technologies mais des exigences de qualité qui s'appliquent par-dessus tout ce qui a déjà été vu ([`../html/`](../html/), [`../css/`](../css/), [`../javascript/`](../javascript/), [`../react/`](../react/), [`../vuejs/`](../vuejs/)) — ce dossier explique comment auditer et améliorer une UI déjà construite plutôt que comment en construire une.

**À quoi sert-elle ?**
- L'accessibilité garantit qu'une interface reste utilisable par des personnes en situation de handicap (visuel, moteur, auditif, cognitif) — via clavier seul, lecteur d'écran, zoom, contrastes réduits.
- La performance garantit une expérience rapide et fluide, en particulier sur des connexions/appareils moins puissants que ceux de l'équipe de développement.

**Où se situe-t-elle ?** Transverse à toute la couche front : le HTML sémantique et l'ARIA (accessibilité structurelle), le CSS et les assets (performance visuelle), le JavaScript (interactions clavier, chargement différé).

**Enjeux** : l'accessibilité est une obligation légale dans de nombreux contextes (secteur public, RGAA en France, WCAG en référence internationale) et un facteur direct de SEO/UX ; la performance impacte directement le taux de conversion et le référencement (les Core Web Vitals sont un facteur de classement Google).
**Pièges courants** : traiter l'accessibilité comme une passe finale ("on ajoutera les `alt` à la fin") plutôt qu'un critère de conception dès le départ ; optimiser la performance sur des métriques qui ne reflètent pas l'expérience réelle perçue par l'utilisateur.

## 2. Prérequis

- HTML sémantique et CSS solides — voir [`../html/`](../html/), [`../css/`](../css/).
- JavaScript pour la gestion du focus et des interactions clavier — voir [`../javascript/`](../javascript/).
- Idéalement un framework front pratiqué (voir [`../react/`](../react/), [`../vuejs/`](../vuejs/)) pour appliquer les concepts dans un contexte réel de composants.

## 3. Rappel des bases 🟢

### 01 - HTML sémantique

**Explication** — Utiliser les éléments HTML pour ce qu'ils signifient, pas seulement pour leur apparence par défaut : un lecteur d'écran, un moteur de recherche, ou la navigation clavier native s'appuient sur cette sémantique.

```html
<!-- ❌ Un div cliquable n'est ni focusable ni annoncé comme un bouton -->
<div onclick="submit()">Valider</div>

<!-- ✅ Un vrai bouton hérite gratuitement du focus clavier et du rôle sémantique -->
<button onclick="submit()">Valider</button>
```

**Bonne pratique** : préférer systématiquement l'élément natif (`<button>`, `<nav>`, `<main>`, `<article>`) à un `<div>` stylé pour ressembler au même élément — le natif apporte gratuitement le comportement clavier et l'annonce correcte par les lecteurs d'écran.

### 02 - Texte alternatif des images

**Explication** — Chaque image porteuse d'information nécessite un attribut `alt` décrivant son contenu ou sa fonction ; une image purement décorative doit avoir un `alt=""` vide (pas d'attribut manquant) pour être explicitement ignorée par les lecteurs d'écran.

```html
<img src="graphique-ventes.png" alt="Ventes en hausse de 20% au T3 2026">
<img src="decoration.svg" alt="">
```

**Erreur fréquente** : omettre l'attribut `alt` — un lecteur d'écran annonce alors le nom du fichier image, souvent incompréhensible (`img_47281.png`).

### 03 - Contraste des couleurs

**Explication** — Le [WCAG](https://www.w3.org/WAI/WCAG21/quickref/) exige un ratio de contraste minimal entre texte et fond : 4.5:1 pour du texte standard (niveau AA), 3:1 pour du texte large (≥18pt ou 14pt gras).

**Bonne pratique** : vérifier le contraste avec un outil dédié (Chrome DevTools, [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)) dès la maquette, pas après implémentation — c'est un point souvent corrigé tardivement et coûteux à rattraper sur toute une charte graphique.

### 04 - Attributs ARIA de base

**Explication** — ARIA (Accessible Rich Internet Applications) complète le HTML pour les cas où la sémantique native ne suffit pas (composants JS custom). Règle d'or : *"No ARIA is better than bad ARIA"* — ARIA ne doit compléter que ce que le HTML natif ne peut pas exprimer, jamais remplacer un élément natif existant.

```html
<button aria-label="Fermer la fenêtre" aria-pressed="false">✕</button>
<div role="alert">Le formulaire contient des erreurs.</div>
```

**Erreur fréquente** : ajouter `role="button"` sur un `<div>` sans aussi gérer manuellement le focus (`tabindex="0"`) et l'activation au clavier (`Enter`/`Space`) — ARIA décrit le rôle mais n'ajoute **aucun** comportement, contrairement à un `<button>` natif.

### 05 - Navigation clavier

**Explication** — Toute action réalisable à la souris doit l'être au clavier : `Tab`/`Shift+Tab` pour naviguer entre éléments focusables, `Enter`/`Espace` pour activer, `Échap` pour fermer une modale.

**Bonne pratique** : tester son interface **uniquement au clavier** (sans souris) à chaque fonctionnalité importante ajoutée — c'est le test manuel le plus simple et le plus révélateur de problèmes d'accessibilité.

### 06 - Core Web Vitals — vue d'ensemble

**Explication** — Trois métriques Google mesurant l'expérience utilisateur réelle : **LCP** (Largest Contentful Paint — temps d'affichage du plus grand élément visible, mesure la vitesse perçue de chargement), **INP** (Interaction to Next Paint — délai entre une interaction utilisateur et la mise à jour visuelle correspondante, mesure la réactivité), **CLS** (Cumulative Layout Shift — somme des décalages visuels inattendus pendant le chargement, mesure la stabilité visuelle).

**Bonne pratique** : réserver l'espace des images/publicités avant leur chargement (`width`/`height` ou `aspect-ratio` en CSS) pour éviter le CLS causé par un contenu qui "saute" une fois chargé.

## 4. Concepts intermédiaires 🟡

- **Test au lecteur d'écran** : VoiceOver (macOS/iOS, natif), NVDA (Windows, gratuit) — tester la lecture réelle d'un parcours utilisateur complet, pas seulement d'un élément isolé, révèle des problèmes qu'un audit automatisé (Lighthouse) ne détecte pas (ordre de lecture incohérent, annonces redondantes).
- **Focus trapping dans les modales** : quand une modale s'ouvre, le focus clavier doit être piégé à l'intérieur (Tab ne doit pas atteindre le contenu masqué derrière), et restauré sur l'élément déclencheur à la fermeture.
- **Niveaux de conformité WCAG** : A (minimum), AA (standard visé par la majorité des obligations légales et bonnes pratiques), AAA (niveau le plus exigeant, rarement requis intégralement car certains critères sont difficiles à atteindre sans compromis de design).
- **Formats d'image modernes** : WebP et AVIF offrent une compression nettement supérieure au JPEG/PNG à qualité équivalente ; l'élément `<picture>` permet de servir le format optimal selon le support navigateur avec un fallback.

```html
<picture>
  <source srcset="photo.avif" type="image/avif">
  <source srcset="photo.webp" type="image/webp">
  <img src="photo.jpg" alt="Description de la photo">
</picture>
```

- **Images responsives (`srcset`)** : servir une résolution d'image adaptée à la taille d'écran réelle plutôt qu'une seule image lourde pour tous les contextes.

```html
<img src="photo-800.jpg" srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
     sizes="(max-width: 600px) 400px, 800px" alt="Description">
```

- **Stratégies de chargement de police** : `font-display: swap` affiche immédiatement une police de secours puis bascule vers la police custom une fois chargée, évitant un texte invisible pendant le chargement (FOIT — Flash of Invisible Text).
- **Lazy loading** : `loading="lazy"` sur une image hors du viewport initial retarde son chargement jusqu'à son approche du scroll, réduisant le poids de la page au chargement initial.

## 5. Concepts avancés 🟠🔴

- **Code splitting** : découper le bundle JavaScript par route/fonctionnalité (voir `React.lazy`/`Suspense` dans [`../react/`](../react/)) pour ne charger que le code nécessaire à l'affichage initial, réduisant le LCP et le temps avant interactivité.
- **Live regions ARIA avancées** : `aria-live="polite"` (annonce un changement de contenu sans interrompre l'utilisateur, ex. un message de confirmation) vs `aria-live="assertive"` (interrompt immédiatement, réservé aux erreurs critiques) — utiliser avec parcimonie, un usage excessif rend un lecteur d'écran inutilisable par sur-verbosité.
- **Gestion avancée du focus en SPA** : lors d'un changement de route côté client (sans rechargement de page complet), déplacer manuellement le focus vers le titre de la nouvelle page et annoncer le changement (`aria-live`) — sans quoi un utilisateur de lecteur d'écran ne sait pas que la page a changé.
- **CDN et stratégies de cache** : servir les assets statiques (images, CSS, JS) depuis un CDN proche géographiquement de l'utilisateur, avec des headers de cache longue durée (`Cache-Control: max-age=31536000, immutable`) combinés à un hash de version dans le nom de fichier pour l'invalidation.
- **Audit automatisé vs audit manuel** : Lighthouse/PageSpeed Insights détectent des problèmes structurels (alt manquants, contraste insuffisant, métriques de performance) mais ne remplacent pas un test manuel au clavier et au lecteur d'écran — un score Lighthouse de 100 n'garantit pas une expérience réellement accessible.
- **Web Vitals en conditions réelles (RUM)** : les Core Web Vitals mesurés en laboratoire (Lighthouse, réseau simulé) diffèrent des données réelles utilisateurs (Real User Monitoring, ex. Chrome UX Report) — privilégier les données terrain pour prioriser les optimisations, un site peut être rapide en lab et lent pour une part significative d'utilisateurs réels sur mobile/réseau faible.

## 6. Commandes / syntaxe à connaître

```bash
npx lighthouse https://exemple.com --view   # audit Lighthouse en ligne de commande
```

```html
<img src="photo.jpg" alt="Description" loading="lazy" width="800" height="600">
<button aria-label="Fermer" aria-pressed="false">✕</button>
<div role="alert" aria-live="assertive">Erreur de validation</div>
```

```css
img, video { max-width: 100%; height: auto; }
@font-face { font-family: 'Custom'; src: url('font.woff2'); font-display: swap; }
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Audit et correction d'accessibilité/performance d'une page produit**

À partir d'une page produit fournie (volontairement pleine de problèmes dans l'énoncé de l'exercice niveau 3) :
- Corriger la sémantique HTML (divs cliquables, images sans `alt`, structure de titres incohérente).
- Rendre le bouton "Ajouter au panier" et la modale de confirmation entièrement utilisables au clavier (focus trapping inclus).
- Vérifier et corriger les contrastes de couleur insuffisants.
- Convertir les images en `<picture>` avec formats modernes + `srcset` responsive.
- Réserver l'espace des images pour supprimer le CLS constaté.
- Bonus : lancer un audit Lighthouse avant/après et documenter l'amélioration des scores accessibilité et performance.

## Checklist

- [ ] Comprendre les fondamentaux (HTML sémantique, `alt`, contraste, navigation clavier)
- [ ] Savoir utiliser les attributs ARIA de base sans en abuser
- [ ] Maîtriser les Core Web Vitals (LCP, INP, CLS) et ce qu'ils mesurent
- [ ] Comprendre les concepts importants (focus trapping, formats d'image modernes, lazy loading)
- [ ] Savoir tester au clavier et au lecteur d'écran
- [ ] Connaître les bonnes pratiques (font-display, code splitting, cache CDN)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (live regions, gestion du focus en SPA, RUM vs lab)

## 10. Ressources

- [roadmap.sh — Frontend Performance](https://roadmap.sh/frontend-performance-best-practices) — bonnes pratiques de performance front.
- [web.dev — Core Web Vitals](https://web.dev/vitals/) — référence officielle Google sur les métriques de performance.
- [MDN — Accessibilité](https://developer.mozilla.org/fr/docs/Web/Accessibility) — référence HTML/ARIA.
- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/) — critères de conformité détaillés.
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) — outil de vérification de contraste.
