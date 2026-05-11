<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface User {
  _id: string
  name: string
  email: string
  __v?: number
}

const users = ref<User[]>([])
const loading = ref(true)
const error = ref<string | null>(null)
const apiUrl = 'https://mongo-api-beta.vercel.app/api/users'
const showForm = ref(false)
const editingUser = ref<User | null>(null)
const formData = ref({ name: '', email: '' })

const fetchUsers = async () => {
  try {
    loading.value = true
    error.value = null
    const response = await fetch(apiUrl)
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    users.value = await response.json()
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Failed to fetch users'
  } finally {
    loading.value = false
  }
}

const addUser = async () => {
  try {
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData.value)
    })
    if (!response.ok) throw new Error('Failed to add user')
    await fetchUsers()
    resetForm()
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Failed to add user'
  }
}

const updateUser = async () => {
  if (!editingUser.value) return
  try {
    const response = await fetch(`${apiUrl}/${editingUser.value._id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData.value)
    })
    if (!response.ok) throw new Error('Failed to update user')
    await fetchUsers()
    resetForm()
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Failed to update user'
  }
}

const deleteUser = async (id: string) => {
  if (!confirm('Are you sure you want to delete this user?')) return
  try {
    const response = await fetch(`${apiUrl}/${id}`, {
      method: 'DELETE'
    })
    if (!response.ok) throw new Error('Failed to delete user')
    await fetchUsers()
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Failed to delete user'
  }
}

const openAddForm = () => {
  editingUser.value = null
  formData.value = { name: '', email: '' }
  showForm.value = true
}

const openEditForm = (user: User) => {
  editingUser.value = user
  formData.value = { name: user.name, email: user.email }
  showForm.value = true
}

const resetForm = () => {
  showForm.value = false
  editingUser.value = null
  formData.value = { name: '', email: '' }
  error.value = null
}

const submitForm = () => {
  if (editingUser.value) {
    updateUser()
  } else {
    addUser()
  }
}

onMounted(() => {
  fetchUsers()
})
</script>

<template>
  <div class="container">
    <h1>User List</h1>
    
    <button @click="openAddForm" class="btn-add">+ Add User</button>
    
    <div v-if="showForm" class="form-overlay">
      <div class="form-modal">
        <h2>{{ editingUser ? 'Edit User' : 'Add User' }}</h2>
        <input v-model="formData.name" type="text" placeholder="Name" required />
        <input v-model="formData.email" type="email" placeholder="Email" required />
        <div class="form-actions">
          <button @click="submitForm" class="btn-save">Save</button>
          <button @click="resetForm" class="btn-cancel">Cancel</button>
        </div>
      </div>
    </div>
    
    <div v-if="loading" class="loading">Loading...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <table v-else>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user._id">
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td class="actions">
            <button @click="openEditForm(user)" class="btn-edit">Edit</button>
            <button @click="deleteUser(user._id)" class="btn-delete">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  color: #42b983;
  text-align: center;
}

.btn-add {
  background-color: #42b983;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  margin-bottom: 20px;
}

.btn-add:hover {
  background-color: #38a16f;
}

.form-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.form-modal {
  background: white;
  padding: 30px;
  border-radius: 8px;
  width: 100%;
  max-width: 400px;
}

.form-modal h2 {
  margin-top: 0;
  color: #42b983;
}

.form-modal input {
  width: 100%;
  padding: 10px;
  margin-bottom: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

.form-actions {
  display: flex;
  gap: 10px;
}

.btn-save {
  background-color: #42b983;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  flex: 1;
}

.btn-save:hover {
  background-color: #38a16f;
}

.btn-cancel {
  background-color: #999;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  flex: 1;
}

.btn-cancel:hover {
  background-color: #777;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

th, td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

th {
  background-color: #42b983;
  color: white;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}

tr:hover {
  background-color: #f1f1f1;
}

.actions {
  display: flex;
  gap: 10px;
}

.btn-edit {
  background-color: #3498db;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
}

.btn-edit:hover {
  background-color: #2980b9;
}

.btn-delete {
  background-color: #e74c3c;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
}

.btn-delete:hover {
  background-color: #c0392b;
}

.loading {
  text-align: center;
  font-size: 18px;
  color: #666;
}

.error {
  text-align: center;
  color: #ff4444;
  padding: 20px;
  border: 1px solid #ff4444;
  border-radius: 4px;
}
</style>
