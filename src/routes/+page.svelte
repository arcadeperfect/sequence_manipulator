<script lang="ts">
  // A list of items
  let items: string[] = [];
  
  // Input field value
  let newItem: string = "";
  
  // Add an item to the list
  function addItem(): void {
    if (newItem) {
      items = [...items, newItem];
      newItem = "";
    }
  }
  
  // Remove an item by its position in the list
  function removeItem(index: number): void {
    items = items.filter((item: string, i: number) => i !== index);
  }
</script>

<main>
  <h1>My List</h1>
  
  <div class="input-row">
    <input 
      bind:value={newItem} 
      placeholder="Add something..."
      onkeydown={(e: KeyboardEvent) => e.key === "Enter" && addItem()}
    />
    <button onclick={addItem}>Add</button>
  </div>

  {#if items.length === 0}
    <p class="empty">No items yet. Add one above!</p>
  {:else}
    <ul>
      {#each items as item, index}
        <li>
          <span>{item}</span>
          <button onclick={() => removeItem(index)}>×</button>
        </li>
      {/each}
    </ul>
  {/if}
</main>

<style>
  main {
    max-width: 400px;
    margin: 2rem auto;
    font-family: system-ui, sans-serif;
  }

  .input-row {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  input {
    flex: 1;
    padding: 0.5rem;
    font-size: 1rem;
  }

  button {
    padding: 0.5rem 1rem;
    cursor: pointer;
  }

  ul {
    list-style: none;
    padding: 0;
  }

  li {
    display: flex;
    justify-content: space-between;
    padding: 0.5rem;
    background: #f0f0f0;
    margin-bottom: 0.25rem;
    border-radius: 4px;
  }

  .empty {
    color: #666;
    font-style: italic;
  }
</style>