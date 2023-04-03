<script setup lang="ts">
export type Props = {
  modelValue: String
}

defineProps<Props>()

const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <ul class="filters">
    <li>
      <a
        href="#/"
        :class="{ selected: modelValue === 'all' }"
        @click="emit('update:modelValue', 'all')"
      >
        All
      </a>
    </li>

    <li>
      <a
        href="#/active"
        :class="{ selected: modelValue === 'active' }"
        @click="emit('update:modelValue', 'active')"
      >
        Active
      </a>
    </li>

    <li>
      <a
        href="#/completed"
        :class="{ selected: modelValue === 'completed' }"
        @click="emit('update:modelValue', 'completed')"
      >
        Completed
      </a>
    </li>
  </ul>
</template>

<style scoped>
@import '../assets/styles/filters.css';
</style>
