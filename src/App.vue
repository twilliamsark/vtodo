<script setup>
import { ref, onMounted, watch, computed } from 'vue';
import TodoItem from './components/TodoItem.vue';

const STORAGE_KEY = 'vue-todos';

const todos = ref([]);

const newTodo = ref('');
const filter = ref('all');
const search = ref('');

const filteredTodos = computed(() => {
  let filtered = todos.value;
  if (filter.value === 'active') {
    filtered = filtered.filter((todo) => !todo.done);
  } else if (filter.value === 'completed') {
    filtered = filtered.filter((todo) => todo.done);
  }
  if (search.value.trim()) {
    const searchTerm = search.value.toLowerCase();
    filtered = filtered.filter((todo) => todo.text.toLowerCase().includes(searchTerm));
  }
  return filtered;
});

function addTodo() {
  if (newTodo.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodo.value.trim(),
      done: false,
    });
    newTodo.value = '';
  }
}

function toggleTodo(id) {
  const todo = todos.value.find((t) => t.id === id);
  if (todo) {
    todo.done = !todo.done;
  }
}

function deleteTodo(id) {
  todos.value = todos.value.filter((t) => t.id !== id);
}

onMounted(() => {
  const stored = localStorage.getItem(STORAGE_KEY);
  if (stored) {
    todos.value = JSON.parse(stored);
  }
});

watch(
  todos,
  (newTodos) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(newTodos));
  },
  { deep: true }
);
</script>

<template>
  <div id="app">
    <h1>Vue TODO App</h1>
    <div class="add-todo">
      <input
        v-model="newTodo"
        @keyup.enter="addTodo"
        placeholder="Add a new todo"
      />
      <button @click="addTodo">Add</button>
    </div>
    <div class="search">
      <input
        v-model="search"
        placeholder="Search todos..."
      />
    </div>
    <div class="filters">
      <button :class="{ active: filter === 'all' }" @click="filter = 'all'">
        All
      </button>
      <button
        :class="{ active: filter === 'active' }"
        @click="filter = 'active'"
      >
        Active
      </button>
      <button
        :class="{ active: filter === 'completed' }"
        @click="filter = 'completed'"
      >
        Completed
      </button>
    </div>
    <ul class="todo-list">
      <TodoItem
        v-for="todo in filteredTodos"
        :key="todo.id"
        :todo="todo"
        @toggle="toggleTodo"
        @delete="deleteTodo"
      />
    </ul>
  </div>
</template>

<style scoped>
#app {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.add-todo {
  display: flex;
  margin-bottom: 20px;
}

.search {
  margin-bottom: 20px;
}

.search input {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.filters button {
  padding: 8px 16px;
  background-color: #f60505;
  border: 1px solid #ccc;
  cursor: pointer;
}

.filters button.active {
  background-color: #42b883;
  color: white;
}

input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #42b883;
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #369870;
}

.todo-list {
  list-style: none;
  padding: 0;
}
</style>
