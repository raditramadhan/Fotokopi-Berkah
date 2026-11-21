
<script setup>
import { ref, onMounted } from 'vue'

const services = ref([])
const showForm = ref(false)
const editIndex = ref(null)

const form = ref({
  name: '',
  price: '',
  unit: 'Lembar'
})

onMounted(() => {
  const savedServices = localStorage.getItem('fotokopi_services')

  if (savedServices) {
    services.value = JSON.parse(savedServices)
  } else {
    services.value = [
      { name: 'Fotokopi Hitam Putih', price: 300, unit: 'Lembar' },
      { name: 'Print Hitam Putih', price: 500, unit: 'Lembar' },
      { name: 'Print Warna', price: 1000, unit: 'Lembar' },
      { name: 'Scan Dokumen', price: 2000, unit: 'File' },
      { name: 'Penjilidan', price: 5000, unit: 'Buku' }
    ]

    saveServices()
  }
})

function saveServices() {
  localStorage.setItem(
    'fotokopi_services',
    JSON.stringify(services.value)
  )
}

function resetForm() {
  form.value = {
    name: '',
    price: '',
    unit: 'Lembar'
  }

  editIndex.value = null
  showForm.value = false
}

function addService() {
  if (
    !form.value.name.trim() ||
    form.value.price === '' ||
    Number(form.value.price) < 0 ||
    !form.value.unit
  ) {
    alert('Lengkapi data layanan dengan benar!')
    return
  }

  const service = {
    name: form.value.name.trim(),
    price: Number(form.value.price),
    unit: form.value.unit
  }

  if (editIndex.value !== null) {
    services.value[editIndex.value] = service
  } else {
    services.value.push(service)
  }

  saveServices()
  resetForm()
}

function editService(index) {
  editIndex.value = index
  form.value = { ...services.value[index] }
  showForm.value = true
}

function deleteService(index) {
  if (confirm('Yakin ingin menghapus layanan ini?')) {
    services.value.splice(index, 1)
    saveServices()
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
  <div class="service-page">
    <div class="service-header">
      <div>
        <h2>Data Layanan</h2>
        <p>Kelola daftar layanan Fotokopi Berkah.</p>
      </div>

      <button class="service-btn-add" @click="showForm = true">
        + Tambah Layanan
      </button>
    </div>

    <div v-if="showForm" class="service-form">
      <h3>
        {{ editIndex !== null ? 'Edit Layanan' : 'Tambah Layanan' }}
      </h3>

      <form @submit.prevent="addService">
        <label>Nama Layanan</label>
        <input
          v-model="form.name"
          type="text"
          placeholder="Masukkan nama layanan"
          required
        />

        <label>Harga (Rp)</label>
        <input
          v-model.number="form.price"
          type="number"
          min="0"
          placeholder="Masukkan harga"
          required
        />

        <label>Satuan</label>
        <select v-model="form.unit" required>
          <option value="Lembar">Lembar</option>
          <option value="File">File</option>
          <option value="Buku">Buku</option>
          <option value="Dokumen">Dokumen</option>
          <option value="Paket">Paket</option>
        </select>

        <div class="service-form-actions">
          <button type="submit" class="service-btn-add">
            Simpan
          </button>

          <button
            type="button"
            class="service-btn-cancel"
            @click="resetForm"
          >
            Batal
          </button>
        </div>
      </form>
    </div>

    <div class="service-table">
      <table>
        <thead>
          <tr>
            <th>No.</th>
            <th>Nama Layanan</th>
            <th>Harga</th>
            <th>Satuan</th>
            <th>Aksi</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(service, index) in services"
            :key="index"
          >
            <td>{{ index + 1 }}</td>
            <td>{{ service.name }}</td>
            <td>{{ formatRupiah(service.price) }}</td>
            <td>{{ service.unit }}</td>
            <td>
              <button
                class="service-btn-edit"
                @click="editService(index)"
              >
                Edit
              </button>

              <button
                class="service-btn-delete"
                @click="deleteService(index)"
              >
                Hapus
              </button>
            </td>
          </tr>

          <tr v-if="services.length === 0">
            <td colspan="5" class="service-empty">
              Belum ada data layanan.
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>