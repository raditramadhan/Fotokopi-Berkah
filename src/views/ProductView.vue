
<script setup>
import { ref, onMounted } from 'vue'

const products = ref([])
const showForm = ref(false)
const editIndex = ref(null)

const form = ref({
  name: '',
  price: '',
  stock: ''
})

onMounted(() => {
  const savedProducts = localStorage.getItem('fotokopi_products')

  if (savedProducts) {
    products.value = JSON.parse(savedProducts)
  } else {
    products.value = [
      { name: 'Kertas A4', price: 500, stock: 500 },
      { name: 'Kertas F4', price: 600, stock: 400 },
      { name: 'Map Plastik', price: 2000, stock: 50 },
      { name: 'Pulpen', price: 3000, stock: 30 },
      { name: 'Amplop', price: 1000, stock: 100 }
    ]
    saveProducts()
  }
})

function saveProducts() {
  localStorage.setItem(
    'fotokopi_products',
    JSON.stringify(products.value)
  )
}

function resetForm() {
  form.value = { name: '', price: '', stock: '' }
  editIndex.value = null
  showForm.value = false
}

function addProduct() {
  if (
    !form.value.name.trim() ||
    form.value.price === '' ||
    form.value.stock === '' ||
    Number(form.value.price) < 0 ||
    Number(form.value.stock) < 0
  ) {
    alert('Lengkapi data produk dengan benar!')
    return
  }

  const product = {
    name: form.value.name.trim(),
    price: Number(form.value.price),
    stock: Number(form.value.stock)
  }

  if (editIndex.value !== null) {
    products.value[editIndex.value] = product
  } else {
    products.value.push(product)
  }

  saveProducts()
  resetForm()
}

function editProduct(index) {
  editIndex.value = index
  form.value = { ...products.value[index] }
  showForm.value = true
}

function deleteProduct(index) {
  if (confirm('Yakin ingin menghapus produk ini?')) {
    products.value.splice(index, 1)
    saveProducts()
  }
}

function formatRupiah(value) {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    maximumFractionDigits: 0
  }).format(value)
}
</script>

<template>
  <div class="product-page">
    <div class="product-header">
      <div>
        <h2>Data Produk</h2>
        <p>Kelola daftar produk Fotokopi Berkah.</p>
      </div>

      <button class="btn-add" @click="showForm = true">
        + Tambah Produk
      </button>
    </div>

    <div v-if="showForm" class="product-form">
      <h3>{{ editIndex !== null ? 'Edit Produk' : 'Tambah Produk' }}</h3>

      <form @submit.prevent="addProduct">
        <label>Nama Produk</label>
        <input v-model="form.name" type="text"
          placeholder="Masukkan nama produk" required />

        <label>Harga (Rp)</label>
        <input v-model.number="form.price" type="number"
          min="0" placeholder="Masukkan harga" required />

        <label>Stok</label>
        <input v-model.number="form.stock" type="number"
          min="0" placeholder="Masukkan stok" required />

        <div class="form-actions">
          <button type="submit" class="btn-add">Simpan</button>
          <button type="button" class="btn-cancel" @click="resetForm">
            Batal
          </button>
        </div>
      </form>
    </div>

    <div class="product-table">
      <table>
        <thead>
          <tr>
            <th>No.</th>
            <th>Nama Produk</th>
            <th>Harga</th>
            <th>Stok</th>
            <th>Aksi</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(product, index) in products" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ product.name }}</td>
            <td>{{ formatRupiah(product.price) }}</td>
            <td>{{ product.stock }}</td>
            <td>
              <button class="btn-edit" @click="editProduct(index)">
                Edit
              </button>
              <button class="btn-delete" @click="deleteProduct(index)">
                Hapus
              </button>
            </td>
          </tr>

          <tr v-if="products.length === 0">
            <td colspan="5" class="empty-data">
              Belum ada data produk.
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>