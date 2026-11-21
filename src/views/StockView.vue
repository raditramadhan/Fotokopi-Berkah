
<script setup>
import { ref, computed, onMounted } from 'vue'

const products = ref([])
const search = ref('')

const showForm = ref(false)
const selectedIndex = ref('')
const adjustmentType = ref('Tambah')
const adjustmentAmount = ref(1)

// Membaca data produk
onMounted(() => {
  products.value = JSON.parse(
    localStorage.getItem('fotokopi_products') || '[]'
  )
})

// Menyimpan data produk
function saveProducts() {
  localStorage.setItem(
    'fotokopi_products',
    JSON.stringify(products.value)
  )
}

// Filter pencarian produk
const filteredProducts = computed(() => {
  return products.value.filter(product =>
    product.name.toLowerCase().includes(search.value.toLowerCase())
  )
})

// Menghitung statistik stok
const totalProducts = computed(() => products.value.length)

const lowStock = computed(() =>
  products.value.filter(
    product => Number(product.stock) > 0 &&
    Number(product.stock) <= 5
  ).length
)

const emptyStock = computed(() =>
  products.value.filter(
    product => Number(product.stock) <= 0
  ).length
)

// Status stok
function stockStatus(stock) {
  if (Number(stock) <= 0) return 'Habis'
  if (Number(stock) <= 5) return 'Menipis'
  return 'Tersedia'
}

// Format angka
function formatNumber(value) {
  return new Intl.NumberFormat('id-ID').format(value)
}

// Membuka form pengaturan stok
function openForm(index) {
  selectedIndex.value = String(index)
  adjustmentType.value = 'Tambah'
  adjustmentAmount.value = 1
  showForm.value = true
}

// Menutup form
function closeForm() {
  showForm.value = false
  selectedIndex.value = ''
  adjustmentAmount.value = 1
}

// Menyimpan perubahan stok
function updateStock() {
  if (selectedIndex.value === '') {
    alert('Silakan pilih produk!')
    return
  }

  if (
    !Number.isInteger(adjustmentAmount.value) ||
    adjustmentAmount.value < 1
  ) {
    alert('Jumlah stok harus minimal 1!')
    return
  }

  const index = Number(selectedIndex.value)
  const product = products.value[index]

  if (!product) {
    alert('Produk tidak ditemukan!')
    return
  }

  const currentStock = Number(product.stock) || 0

  if (adjustmentType.value === 'Kurangi') {
    if (adjustmentAmount.value > currentStock) {
      alert('Stok tidak mencukupi!')
      return
    }

    product.stock = currentStock - adjustmentAmount.value
  } else {
    product.stock = currentStock + adjustmentAmount.value
  }

  saveProducts()

  alert('Stok berhasil diperbarui!')

  closeForm()
}
</script>

<template>
  <div class="stock-page">

    <!-- HEADER -->
    <div class="stock-header">
      <div>
        <h2>Manajemen Stok</h2>
        <p>Kelola persediaan produk Fotokopi Berkah.</p>
      </div>
    </div>

    <!-- STATISTIK STOK -->
    <div class="stock-statistics">

      <div class="stock-stat-card">
        <p>Total Produk</p>
        <h2>{{ totalProducts }}</h2>
      </div>

      <div class="stock-stat-card">
        <p>Stok Menipis</p>
        <h2 class="stock-warning">{{ lowStock }}</h2>
      </div>

      <div class="stock-stat-card">
        <p>Stok Habis</p>
        <h2 class="stock-danger">{{ emptyStock }}</h2>
      </div>

    </div>

    <!-- FORM UPDATE STOK -->
    <div v-if="showForm" class="stock-form">

      <h3>Perbarui Stok Produk</h3>

      <label>Produk</label>
      <select v-model="selectedIndex">
        <option value="">Pilih produk</option>

        <option
          v-for="(product, index) in products"
          :key="index"
          :value="String(index)"
        >
          {{ product.name }}
        </option>
      </select>

      <label>Jenis Perubahan</label>
      <select v-model="adjustmentType">
        <option value="Tambah">Tambah Stok</option>
        <option value="Kurangi">Kurangi Stok</option>
      </select>

      <label>Jumlah</label>
      <input
        v-model.number="adjustmentAmount"
        type="number"
        min="1"
      />

      <div class="stock-form-actions">

        <button
          class="stock-btn-save"
          @click="updateStock"
        >
          Simpan
        </button>

        <button
          class="stock-btn-cancel"
          @click="closeForm"
        >
          Batal
        </button>

      </div>

    </div>

    <!-- DAFTAR STOK -->
    <div class="stock-table">

      <div class="stock-table-header">
        <h3>Daftar Stok Produk</h3>

        <button
          class="stock-btn-add"
          @click="showForm = true"
          :disabled="products.length === 0"
        >
          + Perbarui Stok
        </button>
      </div>

      <input
        v-model="search"
        type="text"
        placeholder="Cari nama produk..."
        class="stock-search"
      />

      <table>
        <thead>
          <tr>
            <th>No.</th>
            <th>Nama Produk</th>
            <th>Harga</th>
            <th>Jumlah Stok</th>
            <th>Status</th>
            <th>Aksi</th>
          </tr>
        </thead>

        <tbody>

          <tr
            v-for="(product, index) in filteredProducts"
            :key="index"
          >
            <td>{{ index + 1 }}</td>
            <td>{{ product.name }}</td>
            <td>
              Rp{{ formatNumber(product.price) }}
            </td>
            <td>
              {{ formatNumber(Number(product.stock) || 0) }}
            </td>
            <td>
              <span
                class="stock-badge"
                :class="{
                  'stock-badge-success': stockStatus(product.stock) === 'Tersedia',
                  'stock-badge-warning': stockStatus(product.stock) === 'Menipis',
                  'stock-badge-danger': stockStatus(product.stock) === 'Habis'
                }"
              >
                {{ stockStatus(product.stock) }}
              </span>
            </td>
            <td>
              <button
                class="stock-btn-edit"
                @click="openForm(products.indexOf(product))"
              >
                Atur Stok
              </button>
            </td>
          </tr>

          <tr v-if="filteredProducts.length === 0">
            <td colspan="6" class="stock-empty">
              Tidak ada produk ditemukan.
            </td>
          </tr>

        </tbody>
      </table>

    </div>

  </div>
</template>