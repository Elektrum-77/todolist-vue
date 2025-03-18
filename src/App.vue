<script setup lang="ts">
import type { Todo } from '@/Todo.ts'
import TodoView from '@/TodoView.vue'
import { computed, ref } from 'vue'
import CheckBox from '@/CheckBox.vue'
import { Icon } from '@iconify/vue'
import { useLocalStorage } from '@vueuse/core'
import { vOnClickOutside } from '@vueuse/components'

const todoLocalStorage = useLocalStorage('todos', JSON.stringify([]))
const todos = computed<Todo[]>({
  get: () => JSON.parse(todoLocalStorage.value) as Todo[],
  set: (v) => (todoLocalStorage.value = JSON.stringify(v)),
})

const deleting = ref<boolean>(false)
const deletingAll = ref<boolean>(false)

function onTodoClick(i: number): void {
  if (deleting.value) return deleteTodo(i)
  else return updateTodo(i)
}

function updateTodo(i: number) {
  const t = todos.value[i]
  t.status = !t.status
  todos.value = todos.value.with(i, t)
}

function deleteTodo(i: number): void {
  deleting.value = false
  todos.value = todos.value.toSpliced(i, 1)
}

function markAll() {
  todos.value = todos.value.map((t) => ({ ...t, status: true }))
}

function unmarkAll() {
  todos.value = todos.value.map((t) => ({ ...t, status: false }))
}

const formTitle = ref<string>('')
const formDescription = ref<string>('')

function addTodo() {
  const title = formTitle.value.trim()
  const description = formDescription.value.trim()
  const status = false

  if (title === '') return

  const todo = { title, status, description: description === '' ? undefined : description }
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
  navigator.clipboard.writeText(todoLocalStorage.value).then(() => alert('Copied to clipboard !'))
}

function deleteAll(): void {
  if (!deletingAll.value) {
    deletingAll.value = true
    return
  }
  todos.value = []
  deletingAll.value = false
}
</script>

<template>
  <div class="app">
    <div class="actions">
      <button @click="markAll" aria-label="all done action">
        <CheckBox :status="true" />
      </button>
      <button class="accent-color" @click="unmarkAll" aria-label="all undone action">
        <CheckBox :status="false" />
      </button>
      <button
        class="red"
        :class="{ 'tilt-shaking-loop': deleting }"
        @click="deleting = !deleting"
        aria-label="delete"
      >
        <Icon :icon="deleting ? 'mdi:trash-can-empty' : 'mdi-trash'" />
      </button>
      <button
        class="red clickable"
        :class="{ 'tilt-shaking-loop': deletingAll }"
        @click="deleteAll"
        v-on-click-outside="() => (deletingAll = false)"
        @keyup.esc="deletingAll = false"
        @focusout="deletingAll = false"
        aria-label="delete all"
      >
        <Icon icon="mdi:clear" />
      </button>
      <button @click="importTodos" aria-label="import from clipboard">
        <Icon icon="mdi:import" />
      </button>
      <button @click="exportTodos" aria-label="export to clipboard">
        <Icon icon="mdi:export" />
      </button>
      <div class="add">
        <button @click="addTodo()" aria-label="add new todo">
          <Icon icon="mdi:plus" />
        </button>
        <input v-model="formTitle" type="text" placeholder="Title" />
        <input v-model="formDescription" type="text" placeholder="Description" />
      </div>
    </div>
    <div class="todos">
      <button
        v-for="({ title, status, description }, i) in todos"
        :key="title"
        @click="onTodoClick(i)"
        :class="{ 'accent-color': !status, deleting, deletingAll }"
      >
        <TodoView :title :status :description />
      </button>
    </div>
  </div>
</template>

<style scoped>
.app {
  padding: 2rem;

  .todos {
    display: flex;
    flex-direction: column;

    .deletingAll,
    .deleting:focus,
    .deleting:hover {
      --color: red;
    }
  }

  .actions {
    display: flex;
    flex-direction: row;
    align-items: center;
    padding: 0.5rem;
    gap: 0.5rem;
    overflow-x: auto;
    overflow-y: clip;

    & > div,
    button {
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
