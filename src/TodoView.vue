<script setup lang="ts">
import type { Todo } from '@/Todo.ts'
import CheckBox from '@/CheckBox.vue'

defineProps<Todo>()
</script>

<template>
  <div class="todo-view clickable">
    <CheckBox :status />
    <span class="title" v-text="title" />
    <span v-if="description !== undefined"> - {{ description }}</span>
  </div>
</template>

<style scoped>
.todo-view {
  display: flex;
  flex-direction: row;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
  width: fit-content;

  svg {
    font-size: 2rem;
  }
}

.title {
  font-size: 1.5rem;
}
</style>
