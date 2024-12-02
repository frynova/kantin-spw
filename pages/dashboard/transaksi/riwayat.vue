<template>
  <div class="flex flex-col flex-grow gap-y-5">
    <UBreadcrumb divider="/" :links="[
      { label: 'Dashboard', to: '/dashboard' },
      { label: 'Transaksi', to: '/dashboard/transaksi' },
      { label: 'Riwayat Transaksi', to: '/dashboard/transaksi/riwayat' }
    ]" />

    <div class="flex">
      <UInput type="date" v-model="date" @change="refresh" />
    </div>
    <div v-if="status === 'error'" class="text-red-500">{{ error.message }}</div>
    <div v-else class="flex flex-col gap-y-5">
      <UTable :rows="transactions" :columns="columns" :loading="status === 'pending'" class="w-full border rounded-lg">
        <template #no-data="{ index }">
          {{ index + 1 }}
        </template>
        <template #waktu-data="{ row: { created_at: time } }">
          {{ new Date(time).toLocaleString('id-ID') }}
        </template>
      </UTable>
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  layout: 'dashboard',
  middleware: 'auth'
})


const supabase = useSupabaseClient()
const user = useSupabaseUser()

const date = ref(new Date().toLocaleDateString('en-CA'))

const { data: userData } = await useAsyncData('userData', async () => {
  const { data, error } = await supabase.from('users').select('nama, role').eq('id', user.value.id).maybeSingle()
  if (error) throw error
  return data
})

const { data: transactions, status, error, refresh } = useLazyAsyncData('transactions', async () => {
  let query = supabase.from('transaksi').select(`
    id, created_at, jumlah,
    produk!inner (
      id, nama,
      kelompok!inner (
        id, nama,
        kelas!inner (
          id, nama
        )
      )
    )
  `).like('produk.kelompok.kelas.nama', `%${userData.value.nama.split(' ')[1]}%`).order('created_at')
  if (date.value) {
    const start = new Date(new Date(date.value).setHours(0, 0, 0, 0)).toISOString()
    const end = new Date(new Date(date.value).setHours(23, 59, 59, 999)).toISOString()
    console.log(start)
    query = query.gte('created_at', start).lte('created_at', end)
  }
  const { data, error } = await query
  if (error) throw error
  console.log(data)
  return data
})

const columns = [
  {
    key: 'no',
    label: 'No'
  },
  {
    key: 'waktu',
    label: 'Tanggal, Waktu'
  },
  {
    key: 'produk.kelompok.nama',
    label: 'Kelompok'
  },
  {
    key: 'produk.kelompok.kelas.nama',
    label: 'Kelas'
  },
  {
    key: 'produk.nama',
    label: 'Nama Barang'
  },
  {
    key: 'jumlah',
    label: 'Jumlah'
  }
]
</script>

<style scoped></style>