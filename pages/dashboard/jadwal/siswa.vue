<template>
  <div class="flex flex-col flex-grow gap-y-24">
    <UBreadcrumb divider="/"
      :links="[{ label: 'Dashboard', to: '/dashboard' }, { label: 'Jadwal', to: '/dashboard/jadwal' }, { label: 'Jadwal Siswa', to: '/dashboard/jadwal/siswa' }]" />

    <div v-if="schedules" class="gap-5">
      <UCard v-for="schedule in schedules" :key="schedule.id" class="flex-1 text-center">
        <template #header>
          <div>{{ schedule.nama }}</div>
        </template>
        <div class="grid grid-cols-2 grid-rows-2 gap-5 text-justify">
          <UCard v-for="day in schedule.hari" :key="day.id">
            <template #header>
              <div class="text-center font-semibold">{{ day.nama }}</div>
            </template>
            <ol>
              <li v-for="teacher in day.guru">{{ teacher.nama }}</li>
            </ol>
            <template #footer>
              <UButton icon="heroicons:pencil-square-20-solid" color="yellow" variant="ghost" />
            </template>
          </UCard>
        </div>
      </UCard>
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  middleware: 'auth',
  layout: 'dashboard',
})

const supabase = useSupabaseClient()

const { data: schedules } = useAsyncData('schedules', async () => {
  try {
    const { data, error } = await supabase.from('hari').select(`
      id, nama,
        siswa (
          id, nama
        )
      )
    `).order('id').range(0,3)
    if (error) throw error
    return data
  } catch (error) {
    console.error(error)
    return
  }
})

</script>

<style scoped></style>