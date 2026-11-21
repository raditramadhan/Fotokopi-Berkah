
<script setup>
import { ref, computed, onMounted } from 'vue'

const transactions = ref([])
const search = ref('')
const selectedTransaction = ref(null)
const showDetail = ref(false)

// Membaca riwayat transaksi
onMounted(() => {
  transactions.value = JSON.parse(
    localStorage.getItem('fotokopi_transactions') || '[]'
  )
})

// Menghitung total pendapatan
const totalRevenue = computed(() => {
  return transactions.value.reduce(
    (total, transaction) =>
      total + Number(transaction.total || 0),
    0
  )
})

// Mencari transaksi berdasarkan nama pelanggan
const filteredTransactions = computed(() => {
  return transactions.value.filter(transaction =>
    transaction.customer
      .toLowerCase()
      .includes(search.value.toLowerCase())
  )
})

// Format Rupiah
function formatRupiah(value) {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    maximumFractionDigits: 0
  }).format(value)
}

// Melihat detail transaksi
function viewDetail(transaction) {
  selectedTransaction.value = transaction
  showDetail.value = true
}

// Menutup detail transaksi
function closeDetail() {
  showDetail.value = false
  selectedTransaction.value = null
}

// Menghapus transaksi
function deleteTransaction(id) {
  if (confirm('Yakin ingin menghapus riwayat transaksi ini?')) {
    transactions.value = transactions.value.filter(
      transaction => transaction.id !== id
    )

    localStorage.setItem(
      'fotokopi_transactions',
      JSON.stringify(transactions.value)
    )

    if (selectedTransaction.value?.id === id) {
      closeDetail()
    }

    alert('Riwayat transaksi berhasil dihapus!')
  }
}
</script>

<template>
  <div class="history-page">

    <!-- HEADER -->
    <div class="history-header">
      <div>
        <h2>Riwayat Transaksi</h2>
        <p>Lihat dan periksa transaksi pelanggan.</p>
      </div>
    </div>

    <!-- STATISTIK -->
    <div class="history-statistics">

      <div class="history-stat-card">
        <p>Total Transaksi</p>
        <h2>{{ transactions.length }}</h2>
      </div>

      <div class="history-stat-card">
        <p>Total Pendapatan</p>
        <h2 class="history-revenue">
          {{ formatRupiah(totalRevenue) }}
        </h2>
      </div>

    </div>

    <!-- DAFTAR TRANSAKSI -->
    <div class="history-table">

      <div class="history-table-header">
        <h3>Daftar Transaksi</h3>
      </div>

      <input
        v-model="search"
        type="text"
        class="history-search"
        placeholder="Cari nama pelanggan..."
      />

      <table>
        <thead>
          <tr>
            <th>No.</th>
            <th>Nama Pelanggan</th>
            <th>Tanggal</th>
            <th>Jumlah Item</th>
            <th>Total Pembayaran</th>
            <th>Aksi</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(transaction, index) in filteredTransactions"
            :key="transaction.id"
          >
            <td>{{ index + 1 }}</td>

            <td>{{ transaction.customer }}</td>

            <td>{{ transaction.date }}</td>

            <td>{{ transaction.items.length }}</td>

            <td>
              {{ formatRupiah(transaction.total) }}
            </td>

            <td>
              <button
                class="history-btn-detail"
                @click="viewDetail(transaction)"
              >
                Detail
              </button>

              <button
                class="history-btn-delete"
                @click="deleteTransaction(transaction.id)"
              >
                Hapus
              </button>
            </td>
          </tr>

          <tr v-if="filteredTransactions.length === 0">
            <td colspan="6" class="history-empty">
              {{
                search
                  ? 'Transaksi tidak ditemukan.'
                  : 'Belum ada riwayat transaksi.'
              }}
            </td>
          </tr>
        </tbody>
      </table>

    </div>

    <!-- DETAIL TRANSAKSI -->
    <div
      v-if="showDetail && selectedTransaction"
      class="history-modal"
    >
      <div class="history-modal-content">

        <div class="history-modal-header">
          <h3>Detail Transaksi</h3>

          <button
            class="history-close"
            @click="closeDetail"
          >
            ×
          </button>
        </div>

        <div class="history-detail-info">
          <p>
            Nama Pelanggan:
            <strong>
              {{ selectedTransaction.customer }}
            </strong>
          </p>

          <p>
            Tanggal:
            <strong>
              {{ selectedTransaction.date }}
            </strong>
          </p>

          <p>
            ID Transaksi:
            <strong>
              {{ selectedTransaction.id }}
            </strong>
          </p>
        </div>

        <h4>Daftar Pesanan</h4>

        <div class="history-detail-table">
          <table>
            <thead>
              <tr>
                <th>Pesanan</th>
                <th>Jumlah</th>
                <th>Subtotal</th>
              </tr>
            </thead>

            <tbody>
              <tr
                v-for="(item, index) in selectedTransaction.items"
                :key="index"
              >
                <td>{{ item.name }}</td>
                <td>{{ item.quantity }}</td>
                <td>
                  {{ formatRupiah(item.subtotal) }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="history-detail-total">
          <span>Total Pembayaran</span>

          <strong>
            {{ formatRupiah(selectedTransaction.total) }}
          </strong>
        </div>

        <button
          class="history-btn-close"
          @click="closeDetail"
        >
          Tutup
        </button>

      </div>
    </div>

  </div>
</template>