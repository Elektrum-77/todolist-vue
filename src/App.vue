<script setup lang="ts">
import type { Todo } from '@/Todo.ts'
import TodoView from '@/TodoView.vue'
import { computed, ref } from 'vue'
import CheckBox from '@/CheckBox.vue'
import { Icon } from '@iconify/vue'
import { useLocalStorage } from '@vueuse/core'

const defaultTodos: Todo[] = []

const todoLocalStorage = useLocalStorage('todos', JSON.stringify(defaultTodos))
const todos = computed<Todo[]>({
  get: () => JSON.parse(todoLocalStorage.value) as Todo[],
  set: (v) => (todoLocalStorage.value = JSON.stringify(v)),
})

const deleting = ref<boolean>(false)

function onTodoClick(i: number): void {
  if (deleting.value) {
    deleting.value = false
    todos.value = todos.value.toSpliced(i, 1)
    return
  }

  const t = todos.value[i]
  t.status = !t.status
  todos.value = todos.value.with(i, t)
}

const formTitle = ref<string>('')
const formDescription = ref<string>('')

function addTodo() {
  const title = formTitle.value
  const description = formDescription.value.trim() === '' ? undefined : formDescription.value
  const status = false

  if (title.trim() === '') return

  const todo = { title, status, description }
  todos.value = [...todos.value, todo]
  formTitle.value = ''
  formDescription.value = ''
}

function isTodo(t: unknown): t is Todo {
  return t !== null && typeof t === 'object' && 'status' in t && 'title' in t
}

function importTodos(): void {
  navigator.clipboard
    .readText()
    .then((text) => JSON.parse(text))
    .then((imported) => {
      if (Array.isArray(imported)) return imported
      throw 'Not an array'
    })
    .then((imported) => imported.filter((t) => isTodo(t)))
    .then(
      (imported) =>
        (todos.value = [
          ...todos.value,
          ...imported.map(({ title, status, description }) => ({ title, status, description })),
        ]),
    )
    .catch((e) => alert('Clipboard is not valid : ' + e))
}

function exportTodos(): void {
  navigator.clipboard.writeText(todoLocalStorage.value)
}
</script>

<template>
  <div class="app">
    <div class="actions">
      <CheckBox
        class="accent-color clickable"
        :status="false"
        @click="todos = todos.map((t) => ({ ...t, status: false }))"
        tabindex="0"
        aria-label="all false action"
      />
      <Icon
        :icon="deleting ? 'mdi:trash-can-empty' : 'mdi-trash'"
        class="red clickable"
        :class="{ 'tilt-shaking-loop': deleting }"
        @click="deleting = !deleting"
        tabindex="0"
        aria-label="delete"
      />
      <Icon
        icon="mdi:clear"
        class="red clickable"
        @click="todos = defaultTodos"
        tabindex="0"
        aria-label="delete all"
      />
      <Icon
        icon="mdi:import"
        class="clickable"
        @click="importTodos"
        tabindex="0"
        aria-label="import from clipboard"
      />
      <Icon
        icon="mdi:export"
        class="clickable"
        @click="exportTodos"
        tabindex="0"
        aria-label="export to clipboard"
      />
      <div class="add">
        <Icon
          class="clickable"
          icon="mdi:plus"
          @click="addTodo()"
          tabindex="0"
          aria-label="add new todo"
        />
        <input v-model="formTitle" type="text" placeholder="Title" />
        <input v-model="formDescription" type="text" placeholder="Description" />
      </div>
    </div>
    <div class="todos" :class="{ red: deleting }">
      <TodoView
        v-for="({ title, status, description }, i) in todos"
        :key="title"
        :title
        :status
        :description
        @click="onTodoClick(i)"
        :class="{ 'accent-color': !status && !deleting }"
        tabindex="0"
      />
    </div>
  </div>
</template>

<style scoped>
.app {
  padding: 2rem;

  .todos {
    display: flex;
    flex-direction: column;
  }

  .actions {
    display: flex;
    flex-direction: row;
    align-items: center;
    padding: 0.5rem;
    gap: 0.5rem;
    overflow-x: auto;
    overflow-y: clip;

    & > div {
      display: flex;
      flex-direction: row;
      align-items: center;
      gap: 0.5rem;
    }

    svg {
      min-width: 2rem; /* does not work on small screen without this */
      font-size: 2rem;
    }
  }
}
</style>
