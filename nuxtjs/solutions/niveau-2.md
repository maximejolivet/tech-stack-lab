# Solutions — Niveau 2

## Exercice 1
```ts
// server/api/produits.ts
export default defineEventHandler(() => {
  return [
    { id: 1, nom: 'Clavier', prix: 49 },
    { id: 2, nom: 'Souris', prix: 19 },
    { id: 3, nom: 'Écran', prix: 199 },
  ]
})
```
```vue
<!-- pages/produits.vue -->
<script setup lang="ts">
const { data: produits } = await useFetch('/api/produits')
</script>
<template>
  <ul>
    <li v-for="p in produits" :key="p.id">{{ p.nom }} — {{ p.prix }}€</li>
  </ul>
</template>
```

## Exercice 2
```ts
// composables/useCompteur.ts
export function useCompteur() {
  const compteur = useState('compteur', () => 0)
  function incrementer() { compteur.value++ }
  return { compteur, incrementer }
}
```
Utilisé dans deux composants distincts, `compteur` reste synchronisé car `useState` partage la même clé (`'compteur'`) pour toute l'app — contrairement à une simple `ref()` locale qui serait isolée par composant.

## Exercice 3
```ts
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  const isLoggedIn = useState('isLoggedIn', () => false)
  if (!isLoggedIn.value) {
    return navigateTo('/login')
  }
})
```
```vue
<!-- pages/dashboard.vue -->
<script setup lang="ts">
definePageMeta({ middleware: 'auth' })
</script>
```

## Exercice 4
```ts
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    '/admin/**': { ssr: false },
  },
})
```
En inspectant le HTML source (`curl` ou "view-source") : `/admin/...` renvoie une coquille HTML quasi vide (rendu 100% client), alors que les autres routes renvoient le HTML déjà rempli par le serveur.

## Exercice 5
```ts
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    public: {
      apiBase: '',
    },
  },
})
```
```bash
# .env
NUXT_PUBLIC_API_BASE=https://api.example.com
```
```vue
<script setup lang="ts">
const config = useRuntimeConfig()
const { data } = await useFetch(`${config.public.apiBase}/produits`)
</script>
```
