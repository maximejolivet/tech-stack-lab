# Vue.js — Solutions niveau 1

## Exercice 1 — Compteur

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)
const increment = () => count.value++
const reset = () => (count.value = 0)
</script>

<template>
  <p>Compteur : {{ count }}</p>
  <button @click="increment">+1</button>
  <button @click="reset">Reset</button>
</template>
```

## Exercice 2 — Champ de saisie contrôlé

```vue
<script setup>
import { ref, computed } from 'vue'
const text = ref('')
const length = computed(() => text.value.length)
</script>

<template>
  <input v-model="text" />
  <p>Vous avez tapé : {{ text }} ({{ length }} caractères)</p>
</template>
```

## Exercice 3 — Liste conditionnelle

```vue
<script setup>
import { ref } from 'vue'
const tasks = ref([
  { id: 1, label: 'Réviser Vue', done: false },
  { id: 2, label: 'Faire les exercices', done: true },
])
</script>

<template>
  <ul v-if="tasks.length">
    <li v-for="task in tasks" :key="task.id">
      {{ task.label }} — {{ task.done ? '✅' : '⏳' }}
    </li>
  </ul>
  <p v-else>Aucune tâche</p>
</template>
```

## Exercice 4 — Props et emit

```vue
<!-- Alert.vue -->
<script setup>
defineProps({
  message: { type: String, required: true },
  type: { type: String, default: 'info' },
})
const emit = defineEmits(['close'])
</script>

<template>
  <div :class="['alert', type]">
    {{ message }}
    <button @click="emit('close')">×</button>
  </div>
</template>
```

```vue
<!-- Parent -->
<script setup>
import { ref } from 'vue'
import Alert from './Alert.vue'
const visible = ref(true)
</script>

<template>
  <Alert v-if="visible" message="Sauvegardé avec succès" type="success" @close="visible = false" />
</template>
```

**Points clés** : `count.value` obligatoire en dehors du template (JS) mais pas dans le template (déballage auto) ; `v-model` = sucre pour `:value` + `@input` ; `:key` unique et stable (id, pas index) ; props en lecture seule, communication vers le parent uniquement via `emit`.
