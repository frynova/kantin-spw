<template>
  <div class="flex flex-col flex-grow gap-y-5">
    <div class="w-full flex justify-between gap-3">
      <div class="flex-1">
        <UInput v-model="searchQuery" placeholder="Cari nama produk..." icon="heroicons:magnifying-glass-20-solid" />
      </div>
      <div class="flex-1 flex justify-center gap-x-5">
        <UButton :variant="activeButtonJurusan('PPLG')" color="green" class="rounded-full" @click="selectJurusan('PPLG')" label="PPLG" />
        <UButton :variant="activeButtonJurusan('TJKT')" color="blue" class="rounded-full" @click="selectJurusan('TJKT')" label="TJKT" />
        <UButton :variant="activeButtonJurusan('TSM')" color="red" class="rounded-full" @click="selectJurusan('TSM')" label="TSM" />
        <UButton :variant="activeButtonJurusan('DKV')" color="orange" class="rounded-full" @click="selectJurusan('DKV')" label="DKV" />
        <UButton :variant="activeButtonJurusan('TOI')" color="gray" class="rounded-full" @click="selectJurusan('TOI')" label="TOI" />
      </div>
    </div>
    <div v-if="status == 'pending' || status == 'error'">
      <div v-if="status == 'pending'">
        Loading...
      </div>

      <div v-if="status == 'error'">
        {{ error.message }}
      </div>
    </div>
    <div v-else class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-x-10 gap-y-5">
      <div v-for="product in filteredProducts" :key="product.id" class="flex">
        <UCard class="flex flex-col" :class="borderClass(product.kelompok.kelas.nama)" :ui="{ header: { base: 'flex-grow flex items-center' } }">
          <template #header>
            <NuxtImg v-if="product.fotoUrl" :src="product.fotoUrl" width="300" />
            <NuxtImg v-else src="img/img-placeholder.png" width="300" />
          </template>
          <div class="text-center font-bold text-lg">{{ product.nama }}</div>
          <div v-if="product.kelompok" class="text-center text-gray-500 text-sm">{{ product.kelompok.nama }} / {{
            product.kelompok.kelas.nama }}</div>
          <div class="text-sm">Stok: {{ product.sisa }}</div>
          <div class="text-sm">Harga: {{ rupiah(product.harga) }}</div>
        </UCard>
      </div>
    </div>
  </div>
</template>

<script setup>
const supabase = useSupabaseClient()

const borderClass = (kelas) => {
  return {
    'border border-green-500': (kelas.includes('PPLG')),
    'border border-blue-500': (kelas.includes('TJKT')),
    'border border-red-500': (kelas.includes('TSM')),
    'border border-orange-500': (kelas.includes('DKV')),
    'border border-gray-500': (kelas.includes('TOI'))
  }
}

const { data: products, status, error } = useLazyAsyncData('products', async () => {
  try {
    let { data, error } = await supabase.from('produk').select(`
      *,
      kelompok (
        id, nama,
        kelas (
          nama
        )
      )
    `).order('foto')
    if (error) throw error
    if (data) {
      data = data.map(data => {
        const { data: url } = supabase.storage.from('produk').getPublicUrl(data.foto)
        return {
          ...data,
          fotoUrl: data.foto ? url.publicUrl : null
        }
      })
    }
    return data
  } catch (error) {
    console.error(error)
    return
  }
})

const availableProducts = computed(() => products.value.filter(product => product.is_available))

const searchQuery = ref('')
const selectedJurusan = ref('')
const selectJurusan = jurusan => selectedJurusan.value = jurusan
const activeButtonJurusan = jurusan => selectedJurusan.value === jurusan ? 'solid' : 'outline'

const filteredProducts = computed(() => {
  const filtered = availableProducts.value.filter((product) => {
    const filterName = product.nama.toLowerCase().includes(searchQuery.value.toLowerCase())
    const filterMajor = selectedJurusan.value ? product.kelompok.kelas.nama.includes(selectedJurusan.value) : true
    
    return filterName && filterMajor
  })

  return filtered
})
</script>

<style scoped></style>