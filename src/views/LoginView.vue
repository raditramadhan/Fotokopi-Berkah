
<script setup>
import { ref } from 'vue'

const emit = defineEmits(['login'])

const username = ref('')
const password = ref('')
const error = ref('')

const accounts = [
  { username: 'pemilik', password: '123456', role: 'Pemilik' },
  { username: 'kasir', password: '123456', role: 'Kasir' },
  { username: 'pelanggan', password: '123456', role: 'Pelanggan' }
]

function handleLogin() {
  const account = accounts.find(
    item =>
      item.username === username.value &&
      item.password === password.value
  )

  if (account) {
    error.value = ''
    emit('login', account)
  } else {
    error.value = 'Username atau password salah!'
  }
}
</script>

<template>
  <div class="login-page">
    <div class="login-card">
      <div class="login-logo">FB</div>

      <h1>Fotokopi Berkah</h1>
      <p class="login-subtitle">
        Sistem Informasi Manajemen
      </p>

      <form @submit.prevent="handleLogin">
        <label>Username</label>
        <input
          v-model="username"
          type="text"
          placeholder="Masukkan username"
          required
        />

        <label>Password</label>
        <input
          v-model="password"
          type="password"
          placeholder="Masukkan password"
          required
        />

        <p v-if="error" class="error-message">
          {{ error }}
        </p>

        <button type="submit" class="login-button">
          Masuk ke Sistem
        </button>
      </form>

      <div class="login-info">
        <strong>Akun Demo</strong>
        <p>Pemilik: pemilik / 123456</p>
        <p>Kasir: kasir / 123456</p>
        <p>Pelanggan: pelanggan / 123456</p>
      </div>
    </div>
  </div>
</template>