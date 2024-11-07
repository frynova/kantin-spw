<template>
  <div class="flex flex-col min-h-full items-center mt-48">
    <div class="w-fit relative border rounded-2xl">
      <UButton class="absolute top-3 left-3" icon="i-line-md-arrow-close-left" variant="ghost" @click="navigateTo('/dashboard/guru')"></UButton>
      <UForm class="p-10 space-y-4 flex flex-col" :validate="validate" :state="state" @submit="addGuru">
        <UFormGroup label="Nama Guru" name="nama">
          <UInput v-model="state.nama" placeholder="Nama lengkap" />
        </UFormGroup>
        <UButton :loading="status == 'pending'" class="justify-center" type="submit">
          Tambah
        </UButton>
        <div class="text-red-500" v-if="status == 'error'">{{ error.message }}</div>
      </UForm>
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  layout: 'dashboard',
  middleware: 'auth'
})

const supabase = useSupabaseClient()

const state = reactive({
  nama: ''
})

const validate = (state) => {
  const errors = []
  if (!state.nama) errors.push({ path: 'nama', message: 'Required' })
  return errors
}

const { execute: addGuru, status, error } = await useAsyncData('addGuru', async () => {
  const { error } = await supabase.from('guru').insert([{
    nama: state.nama
  }])
  if (error) throw new Error('Gagal menambahkan data')
  navigateTo('/dashboard/guru')
}, {
  immediate: false
})
</script>

<style scoped></style>