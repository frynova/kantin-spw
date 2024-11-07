<template>
  <div class="flex flex-col min-h-full justify-center items-center mt-48">
    <div class="w-fit relative border rounded-2xl">
      <UButton class="absolute top-3 left-3" icon="i-line-md-arrow-close-left" variant="ghost" @click="navigateTo('/dashboard/siswa')"></UButton>
      <UForm class="p-10 space-y-4 flex flex-col" :validate="validate" :state="state" @submit="addSiswa">
        <UFormGroup label="Nama Siswa" name="nama">
          <UInput v-model="state.nama" placeholder="Nama lengkap" />
        </UFormGroup>
        <UFormGroup label="Kelas" name="kelas">
          <USelect :options="classes" option-attribute="nama" value-attribute="id"
            v-model="state.kelas" />
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
  nama: '',
  kelas: null
})

const validate = (state) => {
  const errors = []
  if (!state.nama) errors.push({ path: 'nama', message: 'Required' })
  if (!state.kelas) errors.push({ path: 'kelas', message: 'Required' })
  return errors
}

const { execute: addSiswa, status, error } = await useAsyncData('addSiswa', async () => {
  const { error } = await supabase.from('siswa').insert([{
    nama: state.nama,
    kelas: Number(state.kelas),
  }])
  if (error) throw new Error('Gagal menambahkan data')
  navigateTo('/dashboard/siswa')
}, {
  immediate: false
})

const { data: classes } = await useAsyncData('classes', async () => {
  const { data, error } = await supabase.from('kelas').select()
  if (error) throw error
  return data
})
</script>

<style scoped></style>