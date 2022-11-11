<script>
  import Task from './task/Task.svelte'

  let task = ''
  let completed = []
  let todo = []
  let view = 'all'
  let todoUpdateInProgress = false
  let currentlySelectedTask = null

  function changeView (key) {
    view = key
  }

  function addToDo (e) {
    // check if task is empty
    if (!task) return 

    // add task
    todo = [...todo, {
      id: todo?.length,
      guid: getRandomGUID(),
      task,
      completed: false,
      date: new Date()
    }]

    // save to memory
    saveTodoToMemory()

    // empty task
    task = ''
  }

  // update selected task
  function updateSelectedTask (e) {
    // check if empty
    if (!task) {
      // alert
      alert(`Task name is invalid!`)
      // pass data
      task = currentlySelectedTask.task
      return
    }

    // loop through
    todo.map((item, index) => {
      // check if id matches
      if (item.guid == currentlySelectedTask.guid) {
        todo[index].task = task
      }
    })

    // pass data
    todo = todo

    // save to memory
    saveTodoToMemory()

    // empty task
    task = ''

    // change status
    todoUpdateInProgress = false
  }

  function initiateEditTaskInfo (e) {
    let control = e.detail.target
    // change status
    todoUpdateInProgress = true
    // pass data
    currentlySelectedTask = e.detail.task
    // pass name
    task = currentlySelectedTask.task
  }

  // generate a random GUID key
  function getRandomGUID () {
    let array = new Uint32Array(8);
    window.crypto.getRandomValues(array);
    let str = '';
    for (let i = 0; i < array.length; i++) {
      str += (i < 2 || i > 5 ? '' : '-') + array[i].toString(16).slice(-4);
    }
    return str;
  }

  function getCompleted () {
    // get completed
    completed = todo.filter(item => item.completed)
  }

  function markAsCompleted (e) {    
    let control = e.detail.target

    // loop through
    todo.map((item, index) => { 
      // if this is current todo
      if (item.guid == control.value) {
        todo[index].completed = control.checked
      }
    })

    todo = todo

    // save to memory
    saveTodoToMemory()

    // compute completed again
    getCompleted()
  }

  function removeTodoFromList (e) {
    let control = e.detail.target

    // loop through
    todo.map((item, index) => {
      // check if id matches
      if (item.guid == control.getAttribute('data-control-id')) {
        // remove
        todo.splice(index, 1)
      }
    })

    // pass data
    todo = todo

    // save to memory
    saveTodoToMemory()
  }

  function loadTodoListFromMemory () {
    todo = JSON.parse(localStorage.getItem('sv_todo_list') || '[]')
  }

  function toggleAllTasksStatus () {
    let firstTaskStatus = todo[0].completed

    todo.map(item => item.completed = !firstTaskStatus)

    todo = todo

    // compute completed again
    getCompleted()

    // save to memory
    saveTodoToMemory()
  }

  function removeAllCompletedTasks () {
    let activeTasks = todo.filter(item => !item.completed)

    todo = activeTasks

    // compute completed again
    getCompleted()

    // save to memory
    saveTodoToMemory()
  }

  function saveTodoToMemory () {
    // save to memory
    localStorage.setItem('sv_todo_list', JSON.stringify(todo))
  }

  // load todo from memory
  loadTodoListFromMemory()

  // get completed
  getCompleted()
</script>

<main>    

  <div class="card">
    
    <h3 class="mb-10 text-dark font-weight-bold text-uppercase">
      {todoUpdateInProgress ? 'Update Task' : 'What needs to be done?'}
    </h3>

    <form on:submit|preventDefault={null}>
      <div class="mb-5">
        <input type="text" class="task-input" id="task-input" bind:value={task} placeholder={todoUpdateInProgress ? 'Task name' : 'Type your task here...'} />
      </div>
      
      <div class="mb-10">
        {#if !todoUpdateInProgress}
          <button type="submit" class="task-add-button font-weight-bold" on:click={addToDo}>Add To List</button>
        {:else}
          <button type="submit" class="task-add-button font-weight-bold" on:click={updateSelectedTask}>Update</button>
        {/if}
      </div>
    </form>
    
    {#if todo?.length && !todoUpdateInProgress}
      <div class="task-categories-buttons mb-10">
        <button class={view == 'all' ? 'active' : ''} on:click={e => changeView('all')}>
          All ({todo?.length})
        </button>
        <button class={view == 'active' ? 'active' : ''} on:click={e => changeView('active')}>
          Ative ({todo?.length - completed?.length})
        </button>
        <button class={view == 'completed' ? 'active' : ''} on:click={e => changeView('completed')}>
          Completed ({completed?.length})
        </button>
      </div>
    {/if}
    
    {#if todo?.length && !todoUpdateInProgress}
      <div class="task-summary font-weight-bold mb-10">{completed?.length} out of {todo?.length} completed.</div>
    {/if}

    {#if !todoUpdateInProgress}
      <div class="task-container mb-5">
        {#if 
          (view == 'all' && todo?.length) || 
          (view == 'active' && (todo?.length - completed?.length) > 0) ||
          (view == 'completed' && completed?.length)
        }
          <div class="text-uppercase text-left font-weight-bold mb-5">TO-DO LIST</div>
        {/if}

        {#each todo as task (task.guid)}
          {#if view == 'all'}
            <Task {task} on:complete={markAsCompleted} on:remove={removeTodoFromList} on:edit={initiateEditTaskInfo} />
          {:else if view == 'active' && !task.completed}
            <Task {task} on:complete={markAsCompleted} on:remove={removeTodoFromList} on:edit={initiateEditTaskInfo} />
          {:else if view == 'completed' && task.completed}
            <Task {task} on:complete={markAsCompleted} on:remove={removeTodoFromList} on:edit={initiateEditTaskInfo} />
          {/if}
        {/each}
      
      </div>
    {/if}

    {#if 
      ((view == 'all' && todo?.length) || 
      (view == 'active' && (todo?.length - completed?.length) > 0) ||
      (view == 'completed' && completed?.length)) &&
      !todoUpdateInProgress
    }
      <div class="task-categories-buttons mb-5">
        {#if todo?.length > 1}
          <button on:click={toggleAllTasksStatus}>
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mr-2"><polyline points="17 1 21 5 17 9"></polyline><path d="M3 11V9a4 4 0 0 1 4-4h14"></path><polyline points="7 23 3 19 7 15"></polyline><path d="M21 13v2a4 4 0 0 1-4 4H3"></path></svg>
            Toggle All
          </button>
        {/if}
        
        {#if completed?.length}
          <button on:click={removeAllCompletedTasks}>
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mr-2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path><polyline points="22 4 12 14.01 9 11.01"></polyline></svg>
            Remove Completed
          </button>
        {/if}
      </div>
    {/if}

  </div>

</main>

<style>
  button {
    font-family: inherit;
    font-size: 16px;
    display: flex;
    place-items: center;
    place-content: center;
    margin: auto;
  }
  button:focus-visible,
  button:focus {
    outline: none;
  }
  .card {
    background-color: #fff;
    width: 600px;
    border: 1px solid #eee;
    border-radius: 10px;
    -webkit-border-radius: 10px;
    font-size: 18px;
  }
  input.task-input {
    border: 2px solid #ff8c0066;
    background-color: transparent;
    border-radius: 10px;
    -webkit-border-radius: 10px;
    width: 100%;
    padding: 10px;
    font-family: inherit;
    font-size: 16px;
    color: #444;
    word-spacing: unset;
    box-sizing: border-box;
  }
  input.task-input:focus-visible,
  input.task-input:focus {
    border: 2px solid #ff8c00 !important;
    outline: none;
  }
  button.task-add-button {
    padding: 10px;
    background-color: #ff8c00;
    border: none;
    width: 100%;
    color: #fff;
  }
  button.task-add-button:hover {
    background-color: #ff8c0066;
    color: #333;
    outline: none;
    border: none;
  }
  div.task-categories-buttons {
    display: flex;
    padding: 10px;
    place-content: center;
  }
  div.task-categories-buttons > button {
    margin: 5px;
    padding: 10px;
    min-width: 120px;
    background-color: #f9f9f9;
    border: 1px solid #eee;
    color: #444;
  }
  div.task-categories-buttons > button:hover,
  div.task-categories-buttons > button:focus,
  div.task-categories-buttons > button:focus-visible,
  div.task-categories-buttons > button.active {
    color: #ff8c00bb;
    cursor: pointer;
    background-color: #fff;
    border-color: #ff8c00bb;
  }
  div.task-summary {
    color: #444;
    background-color: #f9f9f9;
    padding: 10px;
  }
  svg {
    width: 18px;
    height: 18px;
    margin: 3px;
  }
  svg:hover {
    stroke: #ff8c00;
  }
</style>
