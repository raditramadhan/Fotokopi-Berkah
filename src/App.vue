
<script setup>
import { ref } from 'vue'
import LoginView from './views/LoginView.vue'
import DashboardView from './views/DashboardView.vue'

const user = ref(
  JSON.parse(localStorage.getItem('fotokopiUser')) || null
)

function handleLogin(account) {
  user.value = account
  localStorage.setItem(
    'fotokopiUser',
    JSON.stringify(account)
  )
}

function handleLogout() {
  user.value = null
  localStorage.removeItem('fotokopiUser')
}
</script>

<template>
  <LoginView
    v-if="!user"
    @login="handleLogin"
  />

  <DashboardView
    v-else
    :user="user"
    @logout="handleLogout"
  />
</template>