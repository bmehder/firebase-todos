<script>
  import { scale } from 'svelte/transition'
  import { getFirestore, doc, updateDoc, deleteDoc } from 'firebase/firestore'

  const db = getFirestore()

  export let todos = []
  export let task = ''

  const markComplete = async todo =>
    await updateDoc(doc(db, 'todos', todo.id), {
      isComplete: !todo.isComplete,
    })

  const updateTodo = todo => {
    task = todo.task
    deleteTodo(todo)
  }

  const deleteTodo = async todo => await deleteDoc(doc(db, 'todos', todo.id))
</script>

<ul>
  {#each todos as todo, index (todo.id)}
    <li transition:scale>
      <span class:complete={todo.isComplete} on:click={() => updateTodo(todo)}>
        {@html todo.task}
      </span>
      <span>
        <button on:click={() => markComplete(todo)}
          ><img
            src="https://img.icons8.com/emoji/32/000000/check-mark-button-emoji.png"
            alt="check"
          /></button
        >
        <button on:click={() => deleteTodo(todo)}
          ><img
            src="https://img.icons8.com/pastel-glyph/32/000000/trash.png"
            alt="trash"
          /></button
        >
      </span>
    </li>
  {:else}
    <p>No todos</p>
  {/each}
</ul>

<style>
  ul {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    margin: 1rem 0;
    padding: 0;
  }
  li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1em;
    border: 1px solid #ccc;
  }
  button {
    padding: 1rem;
    font-size: inherit;
  }
  button {
    margin-right: 0.25em;
    padding: 0;
    background: transparent;
    outline: none;
    border: none;
    cursor: pointer;
  }
  .complete {
    text-decoration: line-through;
  }
</style>
