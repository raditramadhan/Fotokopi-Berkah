
<script setup>
defineProps({
  role: String,
  activeMenu: String
})

const emit = defineEmits(['change-menu', 'logout'])

const menus = [
  'Dashboard',
  'Produk',
  'Layanan',
  'Transaksi',
  'Stok',
  'Riwayat Transaksi',
  'Laporan'
]
</script>

<template>
  <aside class="sidebar">
    <div class="brand">
      <div class="brand-logo">FB</div>
      <div>
        <h2>Fotokopi Berkah</h2>
        <small>Management System</small>
      </div>
    </div>

    <p class="menu-label">MENU UTAMA</p>

    <nav>
      <button
        v-for="menu in menus"
        :key="menu"
        v-show="
          role === 'Pemilik' ||
          (role === 'Kasir' &&
            menu !== 'Laporan') ||
          (role === 'Pelanggan' &&
            ['Dashboard', 'Produk', 'Layanan'].includes(menu))
        "
        :class="{ active: activeMenu === menu }"
        @click="emit('change-menu', menu)"
      >
        {{ menu }}
      </button>
    </nav>

    <div class="sidebar-bottom">
      <span>Login sebagai: {{ role }}</span>
      <button class="logout-button" @click="emit('logout')">
        Keluar
      </button>
    </div>
  </aside>
</template>