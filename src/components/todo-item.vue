<script setup lang="ts">
import { ref, nextTick } from 'vue'
import type { Todo } from '../App.vue'

export type Props = {
  todo: Todo
}

const props = defineProps<Props>()
const isEditing = ref(false)
const newTodoTitle = ref('')
const title = ref<HTMLInputElement | null>(null)
const emit = defineEmits(['toggle', 'delete', 'update'])

const edit = () => {
  newTodoTitle.value = props.todo.title
  isEditing.value = true

  nextTick(() => {
    title.value?.focus()
  })
}

const rename = () => {
  if (!isEditing.value) {
    return
  }

  isEditing.value = false

  if (newTodoTitle.value === props.todo.title) {
    return
  }

  if (newTodoTitle.value === '') {
    emit('delete')

    return
  }

  emit('update', {
    ...props.todo,
    title: newTodoTitle.value
  })
}
</script>

<template>
  <li
    :class="{
      completed: todo.completed,
      editing: isEditing
    }"
  >
    <div class="view">
      <input
        type="checkbox"
        class="toggle"
        id="toggle-view"
        :checked="todo.completed"
        @change="emit('toggle')"
      />
      <label htmlFor="toggle-view" @click.prevent @dblclick="edit">
        {{ todo.title }}
      </label>
      <button type="button" class="destroy" data-cy="deleteTodo" @click="emit('delete')"></button>
    </div>
    <input
      type="text"
      v-model.trim="newTodoTitle"
      class="edit"
      ref="title"
      @blur="rename"
      @keyup.enter="rename"
      @keyup.esc="isEditing = false"
    />
  </li>
</template>
