<template>
  <div class="flex flex-col flex-grow gap-y-10">
    <div class="w-full flex justify-between gap-3">
      <div class="flex-[.5]">
        <UInput v-model="searchQuery" placeholder="Cari nama produk..." icon="heroicons:magnifying-glass-20-solid" />
      </div>
      <div class="flex justify-center gap-x-5">
        <UButton :variant="activeButtonJurusan('PPLG')" color="green" class="rounded-full"
          @click="selectJurusan('PPLG')" label="PPLG" />
        <UButton :variant="activeButtonJurusan('TJKT')" color="blue" class="rounded-full" @click="selectJurusan('TJKT')"
          label="TJKT" />
        <UButton :variant="activeButtonJurusan('TSM')" color="red" class="rounded-full" @click="selectJurusan('TSM')"
          label="TSM" />
        <UButton :variant="activeButtonJurusan('DKV')" color="orange" class="rounded-full" @click="selectJurusan('DKV')"
          label="DKV" />
        <UButton :variant="activeButtonJurusan('TOI')" color="gray" class="rounded-full" @click="selectJurusan('TOI')"
          label="TOI" />
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
    <div v-else class="min-h-full grid grid-cols-2 md:grid-cols-3 gap-x-10 gap-y-8">
      <div v-for="product in filteredProducts" :key="product.id">
        <UCard class="h-full shadow-lg" :class="borderClass(product.kelompok.kelas.nama)"
          :ui="{ body: { base: 'h-full grid grid-flow-col grid-cols-2 grid-rows-2 gap-5' } }">
          <div class="flex flex-col">
            <div class="font-bold text-lg">{{ product.nama }}</div>
            <div class="text-lg text-red-500">{{ rupiah(product.harga) }}</div>
            <div class="text-sm">Stok: {{ product.sisa }}</div>
          </div>
          <div class="flex items-end me-5">
            <UButton class="w-full justify-center" label="Add to cart" />
          </div>
          <div class="row-span-2 flex justify-center items-center">
            <NuxtImg v-if="product.fotoUrl" :src="product.fotoUrl" width="300" />
            <NuxtImg v-else src="img/img-placeholder.png" width="300" />
          </div>
          <!-- <div v-if="product.kelompok" class="text-center text-gray-500 text-sm">{{ product.kelompok.nama }} / {{
            product.kelompok.kelas.nama }}</div> -->
        </UCard>
      </div>
    </div>
  </div>
</template>

<script setup>
const supabase = useSupabaseClient()

const borderClass = (kelas) => {
  return {
    'border border-green-500 shadow-green-500': (kelas.includes('PPLG')),
    'border border-blue-500 shadow-blue-500': (kelas.includes('TJKT')),
    'border border-red-500 shadow-red-500': (kelas.includes('TSM')),
    'border border-orange-500 shadow-orange-500': (kelas.includes('DKV')),
    'border border-gray-500 shadow-gray-500': (kelas.includes('TOI'))
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
const selectJurusan = jurusan => selectedJurusan.value === jurusan ? selectedJurusan.value = '' : selectedJurusan.value = jurusan
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