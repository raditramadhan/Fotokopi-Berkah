
<script setup>
import AppSidebar from '../components/AppSidebar.vue'
import ProductView from './ProductView.vue'
import ServiceView from './ServiceView.vue'
import TransactionView from './TransactionView.vue'
import StockView from './StockView.vue'
import HistoryView from './HistoryView.vue'
import ReportView from './ReportView.vue'
import { ref } from 'vue'

defineProps({
  user: Object
})

const emit = defineEmits(['logout'])

const activeMenu = ref('Dashboard')

const statistics = [
  { title: 'Total Produk', value: '125', icon: '📦' },
  { title: 'Total Layanan', value: '6', icon: '🖨️' },
  { title: 'Total Transaksi', value: '48', icon: '🧾' },
  { title: 'Total Pendapatan', value: 'Rp2.450.000', icon: '💰' }
]

const transactions = [
  { id: 'TRX001', customer: 'Andi', service: 'Fotokopi', total: 'Rp15.000', status: 'Selesai' },
  { id: 'TRX002', customer: 'Budi', service: 'Print', total: 'Rp25.000', status: 'Selesai' },
  { id: 'TRX003', customer: 'Siti', service: 'Penjilidan', total: 'Rp20.000', status: 'Selesai' }
]
</script>

<template>
  <div class="app-layout">
    <AppSidebar
      :role="user.role"
      :active-menu="activeMenu"
      @change-menu="activeMenu = $event"
      @logout="emit('logout')"
    />

    <main class="main-content">
      <header class="topbar">
        <div>
          <h1>{{ activeMenu }}</h1>
          <p>Sistem Informasi Manajemen Fotokopi Berkah</p>
        </div>

        <div class="user-profile">
          <div class="avatar">
            {{ user.username.charAt(0).toUpperCase() }}
          </div>
          <div>
            <strong>{{ user.username }}</strong>
            <small>{{ user.role }}</small>
          </div>
        </div>
      </header>

      <!-- HALAMAN DASHBOARD -->
      <section v-if="activeMenu === 'Dashboard'">
        <div class="welcome">
          <h2>Selamat Datang, {{ user.role }}!</h2>
          <p>
            Berikut ringkasan informasi usaha Fotokopi Berkah.
          </p>
        </div>

        <div class="stats-grid">
          <div
            v-for="item in statistics"
            :key="item.title"
            class="stat-card"
          >
            <span class="stat-icon">{{ item.icon }}</span>
            <p>{{ item.title }}</p>
            <h2>{{ item.value }}</h2>
          </div>
        </div>

        <div class="content-card">
          <h3>Transaksi Terbaru</h3>

          <div class="table-wrapper">
            <table>
              <thead>
                <tr>
                  <th>ID Transaksi</th>
                  <th>Pelanggan</th>
                  <th>Layanan</th>
                  <th>Total</th>
                  <th>Status</th>
                </tr>
              </thead>

              <tbody>
                <tr v-for="trx in transactions" :key="trx.id">
                  <td>{{ trx.id }}</td>
                  <td>{{ trx.customer }}</td>
                  <td>{{ trx.service }}</td>
                  <td>{{ trx.total }}</td>
                  <td><span class="status">{{ trx.status }}</span></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </section>

      <!-- HALAMAN PRODUK -->
      <section v-else-if="activeMenu === 'Produk'">
        <ProductView />
      </section>

      <!-- HALAMAN LAYANAN -->
<section v-else-if="activeMenu === 'Layanan'">
  <ServiceView />
</section>

<!-- HALAMAN TRANSAKSI -->
<section v-else-if="activeMenu === 'Transaksi'">
  <TransactionView />
</section>

<!-- HALAMAN STOK -->
<section v-else-if="activeMenu === 'Stok'">
  <StockView />
</section>

<!-- HALAMAN RIWAYAT TRANSAKSI -->
<section v-else-if="activeMenu === 'Riwayat Transaksi'">
  <HistoryView />
</section>

<section v-else-if="activeMenu === 'Laporan'">
  <ReportView />
</section>

      <!-- HALAMAN LAINNYA -->
      <section v-else class="content-card">
        <h2>{{ activeMenu }}</h2>
        <p>
          Halaman {{ activeMenu }} akan dikembangkan pada tahap
          implementasi fitur berikutnya.
        </p>
      </section>
    </main>
  </div>
</template>