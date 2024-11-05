<template>
  <UPopover mode="hover" :popper="{ placement: 'bottom-end' }">
    <UChip color="red" size="xl" :text="orders.length" :show="orders.length > 0">
      <UButton icon="i-heroicons-shopping-cart" variant="ghost" color="black" :padded="false" to="/cart" />
    </UChip>

    <template #panel>
      <UCard :ui="{ base: 'text-black', body: { base: 'flex flex-col gap-3' } }">
        <template #header>
          <div class="font-bold text-xl">Keranjang ({{ orders.length }})</div>
        </template>
        <div v-for="product in orderedProducts" class="flex justify-between items-center gap-5">
          <NuxtImg v-if="product.fotoUrl" :src="product.fotoUrl" width="100" height="100" />
          <NuxtImg v-else src="img/img-placeholder.png" width="100" height="100" />
          <div>{{ product.nama }}</div>
          <div class="font-semibold">{{ product.jumlah }} x {{ rupiah(product.harga) }}</div>
        </div>
      </UCard>
    </template>
  </UPopover>
</template>

<script setup>
const supabase = useSupabaseClient()

const orders = useCookie('orders', { default: () => [] })

const { data: products } = useAsyncData('products', async () => {
  try {
    let { data, error } = await supabase.from('produk').select('id, nama, harga, foto')
    if (error) throw error
    if (data) {
      data = data.map(data => {
        const { data: url } = supabase.storage.from('produk').getPublicUrl(data.foto)
  
        return {
          ...data,
          fotoUrl: url.publicUrl ?? null
        }
      })
    }
    return data
  } catch (error) {
    console.error(error)
    return
  }
})

const orderedProducts = computed(() => {
  return orders.value.map(order => {
    const product = products.value.find(product => product.id === order.produk)
    if (!product) return null

    return {
      ...product,
      jumlah: order.jumlah
    }
  }).filter(Boolean)
})
</script>

<style scoped></style>