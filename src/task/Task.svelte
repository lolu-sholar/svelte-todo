<script>
    import { createEventDispatcher } from 'svelte'

    // props
    export let task

    // create dispatcher
    let dispatch = createEventDispatcher()

    // mark as completed
    function markAsCompleted (e) {
        dispatch('complete', e)
    }

    // remove from todo list
    function removeFromTodoList (e) {
        dispatch('remove', e)
    }

    function initiateEditTodo (e, task) {
        dispatch('edit', {
            target: e.target,
            task
        })
    }
</script>

<div class="task">
    <div class="text-left">
      <label for="task-{task.guid}" class="pointer">
        <input type="checkbox" class="mr-5" on:change={markAsCompleted} checked={task.completed} value={task.guid} id="task-{task.guid}" />
        <div>
          {#if task.completed}
            <s>{task.task}</s>
          {:else}
            {task.task}
          {/if}

          {#if false}
            <div>
              <small class="fs-sm">{task.date}</small>
            </div>
          {/if}
        </div>
      </label>
    </div>
    <div class="task-controls">
      <svg on:click={e => initiateEditTodo(e, task)} data-control-id={task.guid} on:keydown={null} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="pointer"><path d="M17 3a2.828 2.828 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5L17 3z"></path></svg>
      <svg on:click={removeFromTodoList} data-control-id={task.guid} on:keydown={null} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="pointer"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
    </div>
</div>

<style>
div.task {
    display: flex;
    flex-direction: row;
    color: #333;
    border-bottom: 1px solid #eee;
    padding-top: 15px;
    padding-bottom: 10px;
  }
  div.task label {
    font-size: 20px;
    color: #444444bb;
    display: flex;
  }
  div.task input[type="checkbox"] {
    background-color: red;
    color-scheme: none;
  }
  div.task > div:first-child {
    flex-grow: 1;
  }
  div.task > div:last-child {
    flex-shrink: 1;
  }
  svg {
    width: 18px;
    height: 18px;
    margin: 3px;
  }
  svg:hover {
    stroke: #ff8c00;
  }
  .fs-sm {
    font-size: 11px;
  }
</style>