
<script setup>
import { ref, computed, onMounted } from 'vue'

const products = ref([])
const services = ref([])
const transactions = ref([])

const customer = ref('')
const itemType = ref('Produk')
const selectedIndex = ref('')
const quantity = ref(1)

const cart = ref([])

// Membaca data dari Local Storage
onMounted(() => {
  products.value = JSON.parse(
    localStorage.getItem('fotokopi_products') || '[]'
  )

  services.value = JSON.parse(
    localStorage.getItem('fotokopi_services') || '[]'
  )

  transactions.value = JSON.parse(
    localStorage.getItem('fotokopi_transactions') || '[]'
  )
})

// Daftar barang atau layanan sesuai kategori
const availableItems = computed(() => {
  return itemType.value === 'Produk'
    ? products.value
    : services.value
})

// Item yang dipilih
const selectedItem = computed(() => {
  if (selectedIndex.value === '') return null

  return availableItems.value[Number(selectedIndex.value)] || null
})

// Total seluruh pesanan
const grandTotal = computed(() => {
  return cart.value.reduce(
    (total, item) => total + item.subtotal,
    0
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

// Menambahkan item ke keranjang
function addToCart() {
  if (!selectedItem.value) {
    alert('Silakan pilih produk atau layanan!')
    return
  }

  if (!Number.isInteger(quantity.value) || quantity.value < 1) {
    alert('Jumlah pesanan minimal 1!')
    return
  }

  const item = selectedItem.value

  if (
    itemType.value === 'Produk' &&
    quantity.value > Number(item.stock)
  ) {
    alert(`Stok tersedia hanya ${item.stock}!`)
    return
  }

  const cartItem = {
    name: item.name,
    type: itemType.value,
    price: Number(item.price),
    unit: item.unit || 'Pcs',
    quantity: Number(quantity.value),
    subtotal: Number(item.price) * Number(quantity.value)
  }

  cart.value.push(cartItem)

  selectedIndex.value = ''
  quantity.value = 1
}

// Menghapus item dari keranjang
function removeItem(index) {
  cart.value.splice(index, 1)
}

// Menyimpan transaksi
function saveTransaction() {
  if (!customer.value.trim()) {
    alert('Nama pelanggan harus diisi!')
    return
  }

  if (cart.value.length === 0) {
    alert('Tambahkan minimal satu pesanan!')
    return
  }

  const transaction = {
    id: Date.now(),
    customer: customer.value.trim(),
    date: new Date().toLocaleString('id-ID'),
    items: [...cart.value],
    total: grandTotal.value
  }

  transactions.value.unshift(transaction)

  localStorage.setItem(
    'fotokopi_transactions',
    JSON.stringify(transactions.value)
  )

  alert('Transaksi berhasil disimpan!')

  customer.value = ''
  cart.value = []
  quantity.value = 1
  selectedIndex.value = ''
}

// Menghapus riwayat transaksi
function deleteTransaction(index) {
  if (confirm('Yakin ingin menghapus transaksi ini?')) {
    transactions.value.splice(index, 1)

    localStorage.setItem(
      'fotokopi_transactions',
      JSON.stringify(transactions.value)
    )
  }
}
</script>

<template>
  <div class="transaction-page">

    <div class="transaction-header">
      <div>
        <h2>Transaksi</h2>
        <p>Catat pesanan dan pembayaran pelanggan.</p>
      </div>
    </div>

    <!-- FORM TRANSAKSI -->
    <div class="transaction-form">

      <h3>Transaksi Baru</h3>

      <label>Nama Pelanggan</label>
      <input
        v-model="customer"
        type="text"
        placeholder="Masukkan nama pelanggan"
      />

      <div class="transaction-input-group">

        <div>
          <label>Jenis Pesanan</label>
          <select
            v-model="itemType"
            @change="selectedIndex = ''"
          >
            <option value="Produk">Produk</option>
            <option value="Layanan">Layanan</option>
          </select>
        </div>

        <div>
          <label>Pilih Pesanan</label>
          <select v-model="selectedIndex">
            <option value="">Pilih item</option>

            <option
              v-for="(item, index) in availableItems"
              :key="index"
              :value="index"
            >
              {{ item.name }}
            </option>
          </select>
        </div>

        <div>
          <label>Jumlah</label>
          <input
            v-model.number="quantity"
            type="number"
            min="1"
          />
        </div>

      </div>

      <div v-if="selectedItem" class="selected-item">
        <p>Harga:
          <strong>{{ formatRupiah(selectedItem.price) }}</strong>
        </p>

        <p v-if="itemType === 'Produk'">
          Stok tersedia:
          <strong>{{ selectedItem.stock }}</strong>
        </p>

        <p v-else>
          Satuan:
          <strong>{{ selectedItem.unit }}</strong>
        </p>
      </div>

      <button
        class="transaction-btn-add"
        @click="addToCart"
      >
        + Tambahkan Pesanan
      </button>

    </div>

    <!-- KERANJANG PESANAN -->
    <div class="transaction-table">

      <h3>Daftar Pesanan</h3>

      <table>
        <thead>
          <tr>
            <th>No.</th>
            <th>Nama Pesanan</th>
            <th>Jenis</th>
            <th>Harga</th>
            <th>Jumlah</th>
            <th>Subtotal</th>
            <th>Aksi</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(item, index) in cart"
            :key="index"
          >
            <td>{{ index + 1 }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.type }}</td>
            <td>{{ formatRupiah(item.price) }}</td>
            <td>{{ item.quantity }}</td>
            <td>{{ formatRupiah(item.subtotal) }}</td>
            <td>
              <button
                class="transaction-btn-delete"
                @click="removeItem(index)"
              >
                Hapus
              </button>
            </td>
          </tr>

          <tr v-if="cart.length === 0">
            <td colspan="7" class="transaction-empty">
              Belum ada pesanan.
            </td>
          </tr>
        </tbody>
      </table>

      <div class="transaction-total">
        <span>Total Pembayaran</span>
        <strong>{{ formatRupiah(grandTotal) }}</strong>
      </div>

      <button
        class="transaction-btn-save"
        @click="saveTransaction"
      >
        Simpan Transaksi
      </button>

    </div>

    <!-- RIWAYAT TRANSAKSI -->
    <div class="transaction-table">

      <h3>Riwayat Transaksi</h3>

      <table>
        <thead>
          <tr>
            <th>No.</th>
            <th>Nama Pelanggan</th>
            <th>Tanggal</th>
            <th>Jumlah Item</th>
            <th>Total</th>
            <th>Aksi</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(transaction, index) in transactions"
            :key="transaction.id"
          >
            <td>{{ index + 1 }}</td>
            <td>{{ transaction.customer }}</td>
            <td>{{ transaction.date }}</td>
            <td>{{ transaction.items.length }}</td>
            <td>{{ formatRupiah(transaction.total) }}</td>
            <td>
              <button
                class="transaction-btn-delete"
                @click="deleteTransaction(index)"
              >
                Hapus
              </button>
            </td>
          </tr>

          <tr v-if="transactions.length === 0">
            <td colspan="6" class="transaction-empty">
              Belum ada riwayat transaksi.
            </td>
          </tr>
        </tbody>
      </table>

    </div>

  </div>
</template>