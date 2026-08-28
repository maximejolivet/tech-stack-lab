# Vue.js — Solutions niveau 3

## Exercice 1 — Store Pinia partagé

```js
// stores/cart.js
import { defineStore } from 'pinia'

export const useCartStore = defineStore('cart', {
  state: () => ({ items: [] }), // [{ id, name, price, qty }]
  getters: {
    total: (state) => state.items.reduce((sum, i) => sum + i.price * i.qty, 0),
    count: (state) => state.items.reduce((sum, i) => sum + i.qty, 0),
  },
  actions: {
    add(product) {
      const existing = this.items.find(i => i.id === product.id)
      existing ? existing.qty++ : this.items.push({ ...product, qty: 1 })
    },
    remove(id) {
      this.items = this.items.filter(i => i.id !== id)
    },
  },
})
```

```vue
<!-- CartIcon.vue -->
<script setup>
import { useCartStore } from '@/stores/cart'
const cart = useCartStore()
</script>
<template><span>🛒 {{ cart.count }}</span></template>
```

```vue
<!-- CartView.vue -->
<script setup>
import { useCartStore } from '@/stores/cart'
const cart = useCartStore()
</script>
<template>
  <li v-for="item in cart.items" :key="item.id">{{ item.name }} x{{ item.qty }}</li>
  <p>Total : {{ cart.total }}€</p>
</template>
```

Les deux composants appellent `useCartStore()` : Pinia retourne la **même instance** partout dans l'app, donc l'état reste synchronisé sans props ni emits entre composants non liés.

## Exercice 2 — `v-memo`

```vue
<script setup>
import { ref } from 'vue'
const items = ref(Array.from({ length: 1000 }, (_, i) => ({ id: i, value: Math.random(), selected: false })))
</script>

<template>
  <div v-for="item in items" :key="item.id" v-memo="[item.selected]">
    {{ item.value.toFixed(4) }} — {{ item.selected ? 'sélectionné' : '' }}
  </div>
</template>
```

Sans `v-memo`, chaque mise à jour de `items` (même triviale) force Vue à réévaluer le diff de chaque nœud. Avec `v-memo="[item.selected]"`, Vue **saute entièrement** le re-rendu d'un item tant que `item.selected` n'a pas changé — mesurable via Vue Devtools (onglet Performance) ou en comptant les re-renders avec `onUpdated` en dev.

## Exercice 3 — `Suspense`

```vue
<!-- UserProfile.vue -->
<script setup>
const props = defineProps({ userId: Number })
const res = await fetch(`/api/users/${props.userId}`)
if (!res.ok) throw new Error('Utilisateur introuvable')
const user = await res.json()
</script>

<template>
  <h2>{{ user.name }}</h2>
</template>
```

```vue
<!-- Parent -->
<script setup>
import { onErrorCaptured, ref } from 'vue'
import UserProfile from './UserProfile.vue'
const error = ref(null)
onErrorCaptured((err) => {
  error.value = err.message
  return false // empêche la propagation plus haut
})
</script>

<template>
  <p v-if="error">Erreur : {{ error }}</p>
  <Suspense v-else>
    <template #default><UserProfile :user-id="1" /></template>
    <template #fallback>Chargement du profil...</template>
  </Suspense>
</template>
```

Le top-level `await` dans `<script setup>` transforme le composant en composant async : il ne peut être utilisé qu'à l'intérieur d'un `<Suspense>`, qui affiche le fallback tant que la promesse n'est pas résolue.

## Exercice 4 — Formulaire multi-étapes

```js
// composables/useMultiStepForm.js
import { ref, computed } from 'vue'

const STORAGE_KEY = 'multiStepForm'

export function useMultiStepForm(steps) {
  const saved = JSON.parse(sessionStorage.getItem(STORAGE_KEY) || '{}')
  const currentStep = ref(saved.currentStep ?? 0)
  const formData = ref(saved.formData ?? {})

  const isLastStep = computed(() => currentStep.value === steps.length - 1)
  const currentStepConfig = computed(() => steps[currentStep.value])

  function persist() {
    sessionStorage.setItem(STORAGE_KEY, JSON.stringify({
      currentStep: currentStep.value,
      formData: formData.value,
    }))
  }

  function next() {
    if (!currentStepConfig.value.validate(formData.value)) return
    if (!isLastStep.value) currentStep.value++
    persist()
  }

  function back() {
    if (currentStep.value > 0) currentStep.value--
    persist()
  }

  function updateData(patch) {
    formData.value = { ...formData.value, ...patch }
    persist()
  }

  return { currentStep, formData, isLastStep, currentStepConfig, next, back, updateData }
}
```

```js
// Utilisation
const steps = [
  { key: 'personal', validate: (data) => !!data.name },
  { key: 'address', validate: (data) => !!data.city },
  { key: 'confirm', validate: () => true },
]
const { currentStep, formData, next, back, updateData } = useMultiStepForm(steps)
```

**Points clés** : l'état de navigation ET les données vivent dans le même composable pour rester synchronisés ; la validation de l'étape courante bloque `next()` avant d'avancer ; la persistance dans `sessionStorage` à chaque mutation permet de survivre à un F5 sans backend.
