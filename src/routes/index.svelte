<script>
  import { initializeApp } from 'firebase/app'
  import {
    getFirestore,
    query,
    collection,
    onSnapshot,
    orderBy,
    doc,
    addDoc,
    updateDoc,
    deleteDoc,
  } from 'firebase/firestore'
  import { firebaseConfig } from '$lib/firebaseConfig.js'
  import Todos from '$lib/Todos.svelte'

  const firebaseApp = initializeApp(firebaseConfig)
  const db = getFirestore()
  const colRef = collection(db, 'todos')
  const q = query(colRef, orderBy('createdAt', 'desc'))

  let todos = []
  let task = ''

  const unsubscribe = onSnapshot(q, querySnapshot => {
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

  const updateTodo = todo => {
    task = todo.task
    deleteTodo(todo)
  }

  const handleKeydown = e => e.key === 'Enter' && addTodo()
</script>

<svelte:window on:keydown={handleKeydown} />

<main>
  <header>
    <h1>Realtime Todo List <small>({todos.length})</small></h1>
  </header>

  <section>
    <input type="text" bind:value={task} placeholder="Add task..." />
    <button class="add-new" on:click={addTodo} disabled={!task}>Add New</button>
    <Todos {todos} {markComplete} {deleteTodo} {updateTodo} />
  </section>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    padding-bottom: 4rem;
    background: hsl(210, 70%, 56%);
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
      Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    font-size: 1.5rem;
  }
  header {
    text-align: center;
    color: hsl(210, 100%, 16%);
  }
  section {
    background: white;
    padding: 4rem;
    border-radius: 0.25em;
    box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2);
  }
  input,
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
  button:hover:not([disabled]) {
    transform: scale(0.99);
  }
  button[disabled] {
    cursor: not-allowed;
  }
  button.add-new {
    margin-left: 0.5em;
    margin-right: 0;
    padding: 1rem 2rem;
    background: dodgerblue;
    color: white;
    transition: background 100ms ease-in-out;
  }
  button.add-new:hover:not([disabled]) {
    background: hsl(210, 100%, 46%);
  }
</style>
