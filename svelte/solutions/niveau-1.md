# Svelte — Solutions niveau 1

## Exercice 1 — Salutation réactive

```svelte
<script>
  let name = $state('Max');
</script>

<input bind:value={name} />
<p>Bonjour {name} !</p>
```

## Exercice 2 — Compteur

```svelte
<script>
  let count = $state(0);
  const increment = () => count++;
  const reset = () => (count = 0);
</script>

<p>Compteur : {count}</p>
<button onclick={increment}>+1</button>
<button onclick={reset}>Reset</button>
```

## Exercice 3 — Liste avec clé

```svelte
<script>
  let tasks = $state([
    { id: 1, label: 'Réviser Svelte', done: false },
    { id: 2, label: 'Faire les exercices', done: true },
  ]);
</script>

<ul>
  {#each tasks as task (task.id)}
    <li>{task.label} — {task.done ? '✅' : '⏳'}</li>
  {/each}
</ul>
```

## Exercice 4 — Rendu conditionnel

```svelte
<script>
  let tasks = $state([]);
</script>

{#if tasks.length}
  <ul>
    {#each tasks as task (task.id)}
      <li>{task.label}</li>
    {/each}
  </ul>
{:else}
  <p>Aucune tâche</p>
{/if}
```

## Exercice 5 — Props avec valeur par défaut

```svelte
<!-- Alert.svelte -->
<script>
  let { message, type = 'info', onClose } = $props();
</script>

<div class={['alert', type]}>
  {message}
  <button onclick={onClose}>×</button>
</div>
```

```svelte
<!-- Parent -->
<script>
  let visible = $state(true);
</script>

{#if visible}
  <Alert message="Sauvegardé avec succès" type="success" onClose={() => visible = false} />
{/if}
```

**Points clés** : `$state` remplace à la fois `ref` et `reactive` de Vue, sans `.value` à déréférencer ; les événements enfant → parent passent par une prop-fonction (`onClose`) plutôt qu'un système d'`emit` séparé ; `{#each ... (task.id)}` avec clé stable évite les bugs de réconciliation par position.
