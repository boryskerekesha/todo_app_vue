<script setup lang="ts">
import { onMounted, watch, ref, computed } from 'vue'
import TodoItem from './components/todo-item.vue'
import FilterBy from './components/filter-by.vue'

export interface Todo {
  id: number
  title: string
  completed: boolean
}

enum Status {
  All = 'all',
  Active = 'active',
  Completed = 'completed'
}

const todos = ref<Todo[]>([])
const status = ref<Status>(Status.All)
const title = ref('')
const newTodo = ref<HTMLInputElement | null>(null)

onMounted(() => {
  let todosStore: Todo[] = []
  const jsonData = localStorage.getItem('todos') || '[]'

  try {
    todosStore = JSON.parse(jsonData)
  } catch {
    throw new Error('not valid JSON')
  }

  todosStore.forEach((item) => todos.value.push(item))
})

watch(
  () => todos.value,
  () => {
    localStorage.setItem('todos', JSON.stringify(todos.value))
    newTodo.value?.focus()
  },
  { deep: true }
)

const activeTodos = computed(() => {
  return todos.value.filter((todo) => todo.completed === false)
})

const completedTodos = computed(() => {
  return todos.value.filter((todo) => todo.completed === true)
})

const allTodos = computed(() => [...todos.value])

const visibleTodos = () => {
  switch (status.value) {
    case Status.Active:
      return activeTodos
    case Status.Completed:
      return completedTodos
    default:
      return allTodos
  }
}

const deleteTodo = (id: number) => {
  todos.value.splice(
    todos.value.findIndex((todo: Todo) => todo.id === id),
    1
  )
}

const toggle = (index: number) => {
  todos.value[index].completed = !todos.value[index].completed
}

const handleSubmit = () => {
  if (!title.value) {
    return
  }
  todos.value.unshift({
    id: Date.now(),
    title: title.value,
    completed: false
  })

  title.value = ''
}

const handleClearCompleted = () => {
  todos.value = todos.value.filter((todo) => !todo.completed)
}

const handleToggleAll = () => {
  if (todos.value.find((todo) => !todo.completed)) {
    todos.value = todos.value.map((todo) => {
      if (!todo.completed) {
        todo.completed = true
      }

      return todo
    })

    return
  }

  todos.value = todos.value.map((todo) => {
    todo.completed = false

    return todo
  })
}
</script>

<template>
  <div class="todoapp">
    <header className="header">
      <h1>todos</h1>

      <form @submit.prevent="handleSubmit">
        <input
          type="text"
          data-cy="createTodo"
          className="new-todo"
          placeholder="What needs to be done?"
          ref="newTodo"
          v-model="title"
        />
      </form>
    </header>

    <section className="main">
      <input
        type="checkbox"
        id="toggle-all"
        class="toggle-all"
        data-cy="toggleAll"
        @click.prevent="handleToggleAll"
      />
      <label htmlFor="toggle-all"> Mark all as complete </label>

      <TransitionGroup class="todo-list" data-cy="todoList" name="list" tag="ul">
        <TodoItem
          v-for="(todo, index) of visibleTodos().value"
          :key="todo.id"
          :todo="todo"
          @toggle="toggle(index)"
          @update="Object.assign(todo, $event)"
          @delete="deleteTodo(todo.id)"
        />
      </TransitionGroup>
    </section>
    <footer class="footer" v-if="todos.length">
      <span class="todo-count" data-cy="todosCounter">
        {{ activeTodos.length }}
        items left
      </span>
      <FilterBy :model-value="status" @update:modelValue="status = $event" />
      <button
        type="button"
        class="clear-completed"
        v-if="completedTodos.length > 0"
        @click="handleClearCompleted"
      >
        Clear completed
      </button>
    </footer>
  </div>
</template>

<style>
@import './assets/styles/index.css';
@import './assets/styles/filters.css';
@import './assets/styles/todo-list.css';
@import './assets/styles/fonts.css';

.list-enter-active,
.list-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
