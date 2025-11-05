<script setup lang="ts">
import { ref, onMounted } from "vue"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogFooter
} from "@/components/ui/dialog"

definePageMeta({
  layout: "dashboard",
})

// ✅ API
const API_URL = "http://localhost:5049/api/Users"

// ✅ Types
interface User {
  userId: number | null
  username: string
  email: string
  passwordHash: string
  displayName: string
}

// ✅ State
const users = ref<User[]>([])
const loading = ref(false)
const isDialogOpen = ref(false)
const isEdit = ref(false)

const form = ref<User>({
  userId: null,
  username: "",
  email: "",
  passwordHash: "",
  displayName: ""
})

// ✅ Load Users
async function loadUsers() {
  loading.value = true
  try {
    const res = await fetch(API_URL)
    if (!res.ok) throw new Error("Failed to load users")
    users.value = await res.json()
  } catch (e) {
    console.error(e)
    alert("Error loading users")
  } finally {
    loading.value = false
  }
}

// ✅ Save User (Add or Update)
async function saveUser() {
  if (!form.value.username || !form.value.email) {
    alert("Username and Email required!")
    return
  }

  const method = isEdit.value ? "PUT" : "POST"
  const url = isEdit.value ? `${API_URL}/${form.value.userId}` : API_URL

  try {
    await fetch(url, {
      method,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(form.value)
    })

    isDialogOpen.value = false
    await loadUsers()
    resetForm()
  } catch (e) {
    alert("Error saving user")
  }
}

// ✅ Delete User
async function deleteUser(id: number | null) {
  if (!id) return
  if (!confirm("Delete this user?")) return

  await fetch(`${API_URL}/${id}`, { method: "DELETE" })
  await loadUsers()
}

// ✅ Edit User
function editUser(u: User) {
  form.value = { ...u }
  isEdit.value = true
  isDialogOpen.value = true
}

// ✅ Add New User
function openAddUser() {
  resetForm()
  isDialogOpen.value = true
}

// ✅ Reset Form
function resetForm() {
  isEdit.value = false
  form.value = {
    userId: null,
    username: "",
    email: "",
    passwordHash: "",
    displayName: ""
  }
}

// ✅ Auto load on page enter
onMounted(loadUsers)
</script>

<template>
  <div class="p-6 space-y-6">

    <!-- Header -->
    <div class="flex justify-between items-center">
      <h1 class="text-2xl font-bold">Users</h1>
      <Button @click="openAddUser">Add User</Button>
    </div>

    <!-- Users Table -->
    <div class="border rounded-lg p-3">
      <table class="w-full text-left text-sm">
        <thead>
          <tr class="border-b bg-gray-50">
            <th class="p-2">ID</th>
            <th class="p-2">Username</th>
            <th class="p-2">Email</th>
            <th class="p-2">Display Name</th>
            <th class="p-2">Actions</th>
          </tr>
        </thead>

        <tbody>
          <tr v-if="loading">
            <td colspan="5" class="p-4 text-center">Loading...</td>
          </tr>

          <tr v-for="(u, i) in users" :key="u.userId ?? i" class="border-b">
            <td class="p-2">{{ u.userId }}</td>
            <td class="p-2">{{ u.username }}</td>
            <td class="p-2">{{ u.email }}</td>
            <td class="p-2">{{ u.displayName }}</td>
            <td class="p-2 space-x-2">
              <Button size="sm" variant="outline" @click="editUser(u)">Edit</Button>
              <Button size="sm" variant="destructive" @click="deleteUser(u.userId)">Delete</Button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Dialog -->
    <Dialog v-model:open="isDialogOpen">
      <DialogContent>
        <DialogHeader>
          <DialogTitle>{{ isEdit ? "Edit User" : "Add User" }}</DialogTitle>
        </DialogHeader>

        <div class="space-y-3">
          <Input v-model="form.username" placeholder="Username" />
          <Input v-model="form.email" placeholder="Email" />
          <Input v-model="form.passwordHash" type="password" placeholder="Password" />
          <Input v-model="form.displayName" placeholder="Display Name" />
        </div>

        <DialogFooter class="mt-4">
          <Button @click="saveUser">Save</Button>
          <Button variant="outline" @click="isDialogOpen = false">Cancel</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  </div>
</template>
