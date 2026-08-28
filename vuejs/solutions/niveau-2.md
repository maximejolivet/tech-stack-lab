# Vue.js — Solutions niveau 2

## Exercice 1 — Liste filtrable avec `computed`

```vue
<script setup>
import { ref, computed } from 'vue'

const products = ref([
  { id: 1, name: 'Clavier', price: 50, category: 'peripherique' },
  { id: 2, name: 'Écran', price: 200, category: 'ecran' },
  { id: 3, name: 'Souris', price: 25, category: 'peripherique' },
])
const selectedCategory = ref('all')

const filteredProducts = computed(() =>
  selectedCategory.value === 'all'
    ? products.value
    : products.value.filter(p => p.category === selectedCategory.value)
)
const totalPrice = computed(() =>
  filteredProducts.value.reduce((sum, p) => sum + p.price, 0)
)
</script>

<template>
  <select v-model="selectedCategory">
    <option value="all">Toutes</option>
    <option value="peripherique">Périphériques</option>
    <option value="ecran">Écrans</option>
  </select>
  <ul>
    <li v-for="p in filteredProducts" :key="p.id">{{ p.name }} — {{ p.price }}€</li>
  </ul>
  <p>Total : {{ totalPrice }}€</p>
</template>
```

## Exercice 2 — Composable `useFetch`

```js
// composables/useFetch.js
import { ref } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)
  const loading = ref(true)

  fetch(url)
    .then(res => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      return res.json()
    })
    .then(json => (data.value = json))
    .catch(err => (error.value = err))
    .finally(() => (loading.value = false))

  return { data, error, loading }
}
```

```vue
<script setup>
import { useFetch } from '@/composables/useFetch'
const { data: users, error, loading } = useFetch('https://jsonplaceholder.typicode.com/users')
</script>

<template>
  <p v-if="loading">Chargement...</p>
  <p v-else-if="error">Erreur : {{ error.message }}</p>
  <ul v-else>
    <li v-for="u in users" :key="u.id">{{ u.name }}</li>
  </ul>
</template>
```

## Exercice 3 — Formulaire avec validation

```vue
<script setup>
import { ref, computed } from 'vue'

const email = ref('')
const password = ref('')
const confirmPassword = ref('')

const emailValid = computed(() => email.value.includes('@'))
const passwordValid = computed(() => password.value.length >= 8)
const passwordsMatch = computed(() => password.value === confirmPassword.value)
const isValid = computed(() => emailValid.value && passwordValid.value && passwordsMatch.value)
</script>

<template>
  <form @submit.prevent>
    <input v-model="email" placeholder="Email" />
    <p v-if="email && !emailValid">Email invalide</p>

    <input v-model="password" type="password" placeholder="Mot de passe" />
    <p v-if="password && !passwordValid">8 caractères minimum</p>

    <input v-model="confirmPassword" type="password" placeholder="Confirmation" />
    <p v-if="confirmPassword && !passwordsMatch">Les mots de passe ne correspondent pas</p>

    <button :disabled="!isValid">S'inscrire</button>
  </form>
</template>
```

## Exercice 4 — Composants avec slots

```vue
<!-- Modal.vue -->
<script setup>
const emit = defineEmits(['close'])
</script>

<template>
  <div class="modal-overlay" @click.self="emit('close')">
    <div class="modal">
      <header><slot name="header" /><button @click="emit('close')">×</button></header>
      <main><slot /></main>
    </div>
  </div>
</template>
```

```vue
<!-- Parent -->
<script setup>
import { ref } from 'vue'
import Modal from './Modal.vue'
const showConfirm = ref(false)
const showInfo = ref(false)
</script>

<template>
  <button @click="showConfirm = true">Supprimer</button>
  <Modal v-if="showConfirm" @close="showConfirm = false">
    <template #header>Confirmation</template>
    Voulez-vous vraiment supprimer cet élément ?
  </Modal>

  <button @click="showInfo = true">Infos</button>
  <Modal v-if="showInfo" @close="showInfo = false">
    <template #header>À propos</template>
    Cette application a été créée avec Vue 3.
  </Modal>
</template>
```

**Points clés** : un `computed` par validation de champ plutôt qu'une fonction géante illisible ; un composable encapsule fetch + état de chargement/erreur, réutilisable partout ; les slots nommés permettent de personnaliser des zones précises d'un composant générique sans dupliquer sa structure.
