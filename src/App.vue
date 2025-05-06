<template>
  <div class="main-container">
    <h1>To-Do List</h1>
    <button @click="toggleDarkMode" class="mode-toggle">
      {{ isDarkMode ? 'Light Mode' : 'Dark Mode' }}
    </button>

    <div class="input-container">
      <input v-model="newTaskName" placeholder="Nama Tugas..." />
      <input v-model="newTaskDesc" placeholder="Deskripsi Tugas..." />
      <input type="datetime-local" v-model="newTime" />
      <select v-model="newPriority">
        <option value="High">High</option>
        <option value="Medium">Medium</option>
        <option value="Low">Low</option>
      </select>
      <button @click="addTask">Tambah</button>
    </div>

    <div class="filter-container">
      <button :class="{ active: filter === 'all' }" @click="setFilter('all')">Semua</button>
      <button :class="{ active: filter === 'unfinished' }" @click="setFilter('unfinished')">Belum Selesai</button>
      <button :class="{ active: filter === 'done' }" @click="setFilter('done')">Selesai</button>
      <button @click="togglePrioritySort">Sort Prioritas: {{ prioritySortOrder === 'desc' ? 'High → Low' : 'Low → High' }}</button>
      <input type="date" v-model="selectedDate" />
      <button @click="clearDateFilter">Reset Tanggal</button>
    </div>

    <div class="list-container">
      <div class="task-item" v-for="(task, index) in filteredTasks" :key="index">
        <div class="task-left">
          <div class="task-info">
            <input v-if="task.isEditing" v-model="task.name" class="edit-input" />
            <strong v-else :class="{ done: task.done }">{{ task.name }}</strong>
            
            <textarea v-if="task.isEditing" v-model="task.description" class="edit-input"></textarea>
            <p v-else class="desc">{{ task.description }}</p>
            <p v-if="!task.isEditing" class="priority">Priority: {{ task.priority }}</p>

            <select v-if="task.isEditing" v-model="task.priority" class="edit-input">
              <option value="High">High</option>
              <option value="Medium">Medium</option>
              <option value="Low">Low</option>
            </select>

            <input v-if="task.isEditing" type="datetime-local" v-model="task.startTime" class="edit-input" />
            <small v-else class="timestamp">Waktu Kerja: {{ new Date(task.startTime).toLocaleString() }}</small>
            <div class="time-info">
              <small v-if="task.done" class="timestamp">Selesai: {{ task.endTime }}</small>
            </div>
          </div>
        </div>

        <div class="task-actions">
          <button v-if="!task.done" class="done-btn" @click="completeTask(task)">Selesai</button>
          <button v-else class="cancel-btn" @click="cancelTask(task)">Batalkan</button>
          <button class="edit-btn" @click="toggleEditMode(task)">Edit</button>
          <button class="delete-btn" @click="removeTask(index)">Hapus</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const newTaskName = ref('')
const newTaskDesc = ref('')
const newTime = ref('')
const newPriority = ref('Medium')
const tasks = ref([])

const filter = ref('all')
const sortOrder = ref('desc')
const prioritySortOrder = ref('desc') // 'asc' or 'desc'

const addTask = () => {
  if (newTaskName.value.trim() !== '' && newTime.value !== '') {
    tasks.value.push({
      name: newTaskName.value,
      description: newTaskDesc.value,
      priority: newPriority.value,
      done: false,
      startTime: newTime.value,
      endTime: null,
    })
    resetInputFields()
  }
}

const resetInputFields = () => {
  newTaskName.value = ''
  newTaskDesc.value = ''
  newTime.value = ''
  newPriority.value = 'Medium'
}

const setFilter = (value) => {
  filter.value = value
}

const toggleSortOrder = () => {
  sortOrder.value = sortOrder.value === 'desc' ? 'asc' : 'desc'
}

const togglePrioritySort = () => {
  prioritySortOrder.value = prioritySortOrder.value === 'desc' ? 'asc' : 'desc'
}

const filteredTasks = computed(() => {
  let result = tasks.value

  if (filter.value === 'unfinished') {
    result = result.filter(task => !task.done)
  } else if (filter.value === 'done') {
    result = result.filter(task => task.done)
  }

  if (selectedDate.value) {
  result = result.filter(task => {
    const taskDateObj = new Date(task.startTime)
    const taskDate = `${taskDateObj.getFullYear()}-${(taskDateObj.getMonth()+1).toString().padStart(2, '0')}-${taskDateObj.getDate().toString().padStart(2, '0')}`
    return taskDate === selectedDate.value
  })
}

  const priorityWeight = { 'High': 3, 'Medium': 2, 'Low': 1 }
  result = result.slice().sort((a, b) => {
    if (prioritySortOrder.value === 'desc') {
      return priorityWeight[b.priority] - priorityWeight[a.priority]
    } else {
      return priorityWeight[a.priority] - priorityWeight[b.priority]
    }
  })

  return result
})

const completeTask = (task) => {
  if (!task.done) {
    const now = new Date()
    task.endTime = now.toLocaleString()
    task.done = true
  }
}

const cancelTask = (task) => {
  task.done = false
  task.endTime = null
}

const removeTask = (index) => {
  tasks.value.splice(index, 1)
}

const toggleEditMode = (task) => {
  task.isEditing = !task.isEditing
}

const selectedDate = ref('')

const clearDateFilter = () => {
  selectedDate.value = ''
}

const isDarkMode = ref(false)
const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value
  document.body.classList.toggle('dark-mode', isDarkMode.value)
}
</script>

<style>
body {
  background-color: #f9fafb;
  font-family: 'Segoe UI', sans-serif;
  margin: 0;
  height: 100vh;
  overflow: hidden;
  color: #1f2937;
}

body.dark-mode {
  background-color: #1f2937;
  color: #f3f4f6;
}

.main-container {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  max-width: 700px;
  width: 90%;
  padding: 24px;
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.08);
  text-align: center;
  overflow-y: auto;
  max-height: 90vh;
}

body.dark-mode .main-container {
  background: #374151;
  color: #f9fafb;
}


.mode-toggle {
  margin-bottom: 16px;
  padding: 10px 16px;
  border: none;
  background-color: #3b82f6;
  color: white;
  border-radius: 8px;
  cursor: pointer;
}

.mode-toggle:hover {
  background-color: #2563eb;
}

body.dark-mode input,
body.dark-mode select,
body.dark-mode textarea {
  background-color: #4b5563;
  color: #f3f4f6;
  border: 1px solid #6b7280;
}

body.dark-mode .task-item {
  background: #4b5563;
}

body.dark-mode .filter-container button {
  background: #6b7280;
  color: #f3f4f6;
}

body.dark-mode .filter-container button.active {
  background: #3b82f6;
}

body.dark-mode .edit-input {
  background-color: #4b5563;
  color: #f3f4f6;
  border: 1px solid #6b7280;
}

h1 {
  font-size: 28px;
  margin-bottom: 20px;
}

.input-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 24px;
}

.input-container input,
.input-container select,
.input-container button {
  padding: 12px 14px;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  outline: none;
  font-size: 14px;
}

.input-container input,
.input-container select {
  flex: 1 1 100%;
  background-color: #f3f4f6;
  color: #374151;
}

.input-container button {
  background-color: #3b82f6;
  color: white;
  border: none;
  cursor: pointer;
}

.input-container button:hover {
  background-color: #2563eb;
}

.filter-container {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
  margin-bottom: 24px;
}

.filter-container button,
.filter-container input[type="date"] {
  padding: 10px 14px;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  background: #f3f4f6;
  color: #374151;
  cursor: pointer;
}

.filter-container button.active {
  background: #3b82f6;
  color: white;
}

.filter-container button:hover {
  background: #93c5fd;
  color: #1e3a8a;
}

.list-container {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 14px 18px;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.04);
}

.task-info strong {
  font-size: 16px;
  color: #111827;
}

.desc {
  margin: 6px 0;
  font-size: 14px;
  color: #374151;
}

.timestamp {
  font-size: 12px;
  color: #6b7280;
  margin-top: 4px;
}

.priority {
  font-size: 13px;
  color: #111827;
}

body.dark-mode .task-info strong {
  color: #f9fafb;
}

body.dark-mode .desc {
  color: #e5e7eb;
}

body.dark-mode .timestamp {
  color: #d1d5db;
}

body.dark-mode .priority {
  color: #f9fafb;
}

.task-actions {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.task-actions button {
  width: 90px;
  padding: 8px 0;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  transition: 0.3s;
  text-align: center;
}

.done-btn {
  background-color: #10b981;
  color: white;
}

.done-btn:hover {
  background-color: #059669;
}

.cancel-btn {
  background-color: #f59e0b;
  color: white;
}

.cancel-btn:hover {
  background-color: #d97706;
}

.delete-btn {
  background-color: #ef4444;
  color: white;
}

.delete-btn:hover {
  background-color: #dc2626;
}

.edit-btn {
  background-color: #3b82f6;
  color: white;
}

.edit-btn:hover {
  background-color: #2563eb;
}

.edit-input {
  width: 100%;
  padding: 10px 14px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  background-color: #f9fafb;
  color: #374151;
  margin-bottom: 8px;
}

.done {
  text-decoration: line-through;
  color: #9ca3af;
}

</style>
