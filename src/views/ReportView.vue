
<script setup>
import { ref, computed, onMounted } from 'vue'
function printReport() {
  window.print()
}

const transactions = ref([])
const selectedPeriod = ref('all')

onMounted(() => {
  transactions.value = JSON.parse(
    localStorage.getItem('fotokopi_transactions') || '[]'
  )
})

// Mengambil tanggal transaksi.
// ID transaksi dibuat menggunakan Date.now() pada fitur Transaksi.
function getTransactionDate(transaction) {
  const idDate = Number(transaction.id)

  if (Number.isFinite(idDate) && idDate > 0) {
    return new Date(idDate)
  }

  return new Date(transaction.date)
}

// Memeriksa transaksi sesuai periode yang dipilih.
const filteredTransactions = computed(() => {
  const now = new Date()

  return transactions.value.filter(transaction => {
    const date = getTransactionDate(transaction)

    if (Number.isNaN(date.getTime())) return false

    if (selectedPeriod.value === 'today') {
      return date.toDateString() === now.toDateString()
    }

    if (selectedPeriod.value === 'month') {
      return (
        date.getMonth() === now.getMonth() &&
        date.getFullYear() === now.getFullYear()
      )
    }

    return true
  })
})

// Menghitung total pendapatan.
const totalRevenue = computed(() => {
  return filteredTransactions.value.reduce(
    (total, transaction) =>
      total + Number(transaction.total || 0),
    0
  )
})

// Menghitung jumlah transaksi.
const totalTransactions = computed(() => {
  return filteredTransactions.value.length
})

// Menghitung total seluruh item yang terjual.
const totalItemsSold = computed(() => {
  return filteredTransactions.value.reduce(
    (total, transaction) => {
      return total + (transaction.items || []).reduce(
        (itemTotal, item) =>
          itemTotal + Number(item.quantity || 0),
        0
      )
    },
    0
  )
})

// Membuat rekap penjualan berdasarkan nama dan jenis item.
const salesRecap = computed(() => {
  const recap = {}

  filteredTransactions.value.forEach(transaction => {
    ;(transaction.items || []).forEach(item => {
      const key = `${item.type || 'Item'}-${item.name}`

      if (!recap[key]) {
        recap[key] = {
          name: item.name,
          type: item.type || 'Item',
          quantity: 0,
          revenue: 0
        }
      }

      recap[key].quantity += Number(item.quantity || 0)
      recap[key].revenue += Number(item.subtotal || 0)
    })
  })

  return Object.values(recap).sort(
    (a, b) => b.quantity - a.quantity
  )
})

function formatRupiah(value) {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    maximumFractionDigits: 0
  }).format(Number(value) || 0)
}

function formatDate(transaction) {
  const date = getTransactionDate(transaction)

  if (Number.isNaN(date.getTime())) {
    return transaction.date || '-'
  }

  return date.toLocaleDateString('id-ID', {
    day: '2-digit',
    month: 'long',
    year: 'numeric'
  })
}
</script>

<template>
  <div class="report-page">
<div class="report-header">
  <div>
    <h2>Laporan Penjualan</h2>
    <p>Lihat laporan pendapatan dan rekap penjualan.</p>
  </div>

  <div class="report-actions">
    <select v-model="selectedPeriod" class="report-filter">
      <option value="all">Semua Waktu</option>
      <option value="today">Hari Ini</option>
      <option value="month">Bulan Ini</option>
    </select>

    <button class="report-print-btn" @click="printReport">
      Cetak / Ekspor PDF
    </button>
  </div>
</div>

    <div class="report-statistics">
      <div class="report-stat-card">
        <p>Total Pendapatan</p>
        <h2>{{ formatRupiah(totalRevenue) }}</h2>
      </div>

      <div class="report-stat-card">
        <p>Jumlah Transaksi</p>
        <h2>{{ totalTransactions }}</h2>
      </div>

      <div class="report-stat-card">
        <p>Total Item Terjual</p>
        <h2>{{ totalItemsSold }}</h2>
      </div>

      <div class="report-stat-card">
        <p>Jenis Item Terjual</p>
        <h2>{{ salesRecap.length }}</h2>
      </div>
    </div>

    <div class="report-section">
      <h3>Rekap Penjualan</h3>

      <div class="report-table-wrapper">
        <table>
          <thead>
            <tr>
              <th>No.</th>
              <th>Nama Item</th>
              <th>Jenis</th>
              <th>Jumlah Terjual</th>
              <th>Total Pendapatan</th>
            </tr>
          </thead>

          <tbody>
            <tr
              v-for="(item, index) in salesRecap"
              :key="`${item.type}-${item.name}`"
            >
              <td>{{ index + 1 }}</td>
              <td>{{ item.name }}</td>
              <td>{{ item.type }}</td>
              <td>{{ item.quantity }}</td>
              <td>{{ formatRupiah(item.revenue) }}</td>
            </tr>

            <tr v-if="salesRecap.length === 0">
              <td colspan="5" class="report-empty">
                Belum ada data penjualan pada periode ini.
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div class="report-section">
      <h3>Daftar Transaksi</h3>

      <div class="report-table-wrapper">
        <table>
          <thead>
            <tr>
              <th>No.</th>
              <th>Nama Pelanggan</th>
              <th>Tanggal</th>
              <th>Jumlah Item</th>
              <th>Total Pembayaran</th>
            </tr>
          </thead>

          <tbody>
            <tr
              v-for="(transaction, index) in filteredTransactions"
              :key="transaction.id"
            >
              <td>{{ index + 1 }}</td>
              <td>{{ transaction.customer }}</td>
              <td>{{ formatDate(transaction) }}</td>
              <td>{{ (transaction.items || []).length }}</td>
              <td>{{ formatRupiah(transaction.total) }}</td>
            </tr>

            <tr v-if="filteredTransactions.length === 0">
              <td colspan="5" class="report-empty">
                Belum ada transaksi pada periode ini.
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>