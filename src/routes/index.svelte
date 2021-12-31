<script>
  import { initializeApp } from 'firebase/app'
  import {
    getFirestore,
    collection,
    onSnapshot,
    doc,
    addDoc,
    updateDoc,
    deleteDoc,
  } from 'firebase/firestore'
  import { firebaseConfig } from '$lib/firebaseConfig.js'

  const firebaseApp = initializeApp(firebaseConfig)
  const db = getFirestore()
  const colRef = collection(db, 'todos')

  let todos = []
  let task = ''

  const unsubscribe = onSnapshot(colRef, querySnapshot => {
    let fbTodos = []

    querySnapshot.forEach(doc => {
      let todo = { ...doc.data(), id: doc.id }
      fbTodos = [todo, ...fbTodos]
    })

    todos = fbTodos
  })

  const addTodo = async () => {
    if (!task) return

    const docRef = await addDoc(collection(db, 'todos'), {
      task,
      isComplete: false,
      createdAt: new Date(),
    })

    task = ''
  }

  const markComplete = async todo =>
    await updateDoc(doc(db, 'todos', todo.id), {
      isComplete: !todo.isComplete,
    })

  const deleteTodo = async todo => await deleteDoc(doc(db, 'todos', todo.id))

  const handleKeydown = e => e.key === 'Enter' && addTodo()
</script>

<svelte:window on:keydown={handleKeydown} />

<input type="text" bind:value={task} placeholder="Add task..." />
<button on:click={addTodo} disabled={!task}>Add New</button>
<ol>
  {#each todos as todo}
    <li>
      <span class:complete={todo.isComplete}>
        {todo.task}
      </span>
      <span>
        <button on:click={() => markComplete(todo)}>√</button>
        <button on:click={() => deleteTodo(todo)}>X</button>
      </span>
    </li>
  {:else}
    <p>No todos</p>
  {/each}
</ol>

<style>
  button[disabled] {
    cursor: not-allowed;
  }
  .complete {
    text-decoration: line-through;
  }
</style>
