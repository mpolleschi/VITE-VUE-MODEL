<!-- filepath: vite-todo-vue/src/App.vue -->
<template>
  <div class="app">
    <h1>Todo App (Vue)</h1>
    <input
      v-model="text"
      @keyup.enter="addTodo"
      placeholder="Aggiungi un todo"
    />
    <select v-model="priority">
      <option disabled value="">Seleziona priorità</option>
      <option value="1">Alta</option>
      <option value="2">Media</option>
      <option value="3">Bassa</option>      
      <option value="4">Invest</option>
    </select>
    <button @click="addTodo">Aggiungi</button>
    <table class="todo-table">
      <thead>
        <tr>
          <th>Completato</th>
          <th>Testo</th>
          <th>Priorità</th>
          <th>Azioni</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(todo, idx) in todos" :key="idx">
          <td>
            <input type="checkbox" v-model="todo.status" />
          </td>
          <td :class="{ done: todo.status }">{{ todo.text }}</td>
          <td>
            {{ priorityLabel(todo.priority) }}
          </td>
          <td>
            <button @click="editTodo(idx)" title="Modifica">
              ✏️
            </button>
            <button @click="removeTodo(idx)" title="Rimuovi">
              🗑️
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { supabase } from './supabase'

const todos = ref([])
const text = ref('')
const priority = ref('')

// Carica i todo da Supabase all'avvio
onMounted(async () => {
  const { data, error } = await supabase.from('todos').select('*').order('id', { ascending: true })
  console.log('LOAD', { data, error })
  if (!error) todos.value = data
})

// Aggiungi un todo su Supabase
async function addTodo() {
  if (text.value.trim() === '' || priority.value === '') return
  const { data, error } = await supabase
    .from('todos')
    .insert([{ text: text.value, priority: Number(priority.value), status: false }])
    .select()
  console.log('ADD', { data, error })
  if (!error && data) todos.value.push(data[0])
  text.value = ''
  priority.value = ''
}

// Rimuovi un todo da Supabase
async function removeTodo(idx) {
  const todo = todos.value[idx]
  await supabase.from('todos').delete().eq('id', todo.id)
  todos.value.splice(idx, 1)
}

// Modifica un todo (carica nel form e cancella da Supabase)
async function editTodo(idx) {
  const todo = todos.value[idx]
  text.value = todo.text
  priority.value = todo.priority
  await supabase.from('todos').delete().eq('id', todo.id)
  todos.value.splice(idx, 1)
}

// Restituisce l'etichetta della priorità
function priorityLabel(val) {
  if (val == 1) return 'Alta'
  if (val == 2) return 'Media'
  if (val == 3) return 'Bassa'
  return ''
}
</script>

<style scoped>
.app {
  max-width: 600px;
  margin: 40px auto;
  padding: 2rem;
  border: 1px solid #333;
  border-radius: 8px;
  background: #222;
  color: #fafafa;
}
input,
select {
  padding: 0.5rem;
  margin-right: 0.5rem;
  background: #333;
  color: #fafafa;
  border: 1px solid #555;
}
button {
  padding: 0.4rem 0.7rem;
  margin: 0 0.2rem;
  background: #444;
  color: #fafafa;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1.1rem;
  transition: background 0.2s;
}
button:hover {
  background: #666;
}
.todo-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
  background: #222;
  table-layout: fixed;
}
.todo-table th,
.todo-table td {
  border: 1px solid #444;
  padding: 0.5rem;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.todo-table th {
  background: #333;
  font-weight: bold;
}
.todo-table th:nth-child(1),
.todo-table td:nth-child(1) {
  width: 110px;
}
.todo-table th:nth-child(2),
.todo-table td:nth-child(2) {
  width: 220px;
}
.todo-table th:nth-child(3),
.todo-table td:nth-child(3) {
  width: 110px;
}
.todo-table th:nth-child(4),
.todo-table td:nth-child(4) {
  width: 110px;
}
.todo-table tbody tr:hover {
  background: #2a2a2a;
}
.done {
  text-decoration: line-through;
  color: #8bc34a;
}
</style>