# Solutions — Niveau 1

## Exercice 1
```bash
npx nuxi@latest init mon-projet
cd mon-projet && npm install && npm run dev
```
Vérifier `http://localhost:3000`.

## Exercice 2
```vue
<!-- pages/index.vue -->
<template><h1>Accueil</h1></template>
```
```vue
<!-- pages/about.vue -->
<template><h1>À propos</h1></template>
```
```vue
<!-- pages/contact.vue -->
<template><h1>Contact</h1></template>
```
Aucune config de routeur nécessaire : Nuxt génère les routes `/`, `/about`, `/contact` à partir des noms de fichiers dans `pages/`.

## Exercice 3
```vue
<!-- pages/users/[id].vue -->
<script setup lang="ts">
const route = useRoute()
</script>
<template>
  <p>Utilisateur n°{{ route.params.id }}</p>
</template>
```

## Exercice 4
```vue
<!-- layouts/default.vue -->
<template>
  <div>
    <header>Mon Header</header>
    <slot />
    <footer>Mon Footer</footer>
  </div>
</template>
```
Ce layout s'applique automatiquement à toutes les pages car aucune ne définit `definePageMeta({ layout: ... })` avec une autre valeur — `default` est le layout par défaut.

## Exercice 5
```vue
<!-- components/AppBadge.vue -->
<script setup lang="ts">
defineProps<{ text: string }>()
</script>
<template>
  <span class="badge">{{ text }}</span>
</template>
```
```vue
<!-- pages/index.vue -->
<template>
  <AppBadge text="Nouveau" />
</template>
```
`<AppBadge>` est utilisable sans `import` car tout fichier de `components/` est auto-importé par Nuxt.
