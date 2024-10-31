<template>
  <div class="flex h-screen">
    <div class="flex flex-col flex-1 overflow-hidden">
      <MainHeader @open-navbar="navbar = true" />
      <div class="container mx-auto px-4 py-5">
        <slot />
      </div>
      <USlideover v-model="navbar" side="left" :ui="{ background: 'bg-[#3F3F3F]' }">
        <div class="p-4 flex-1 text-white">
          <UButton color="gray" variant="ghost" size="sm" icon="i-heroicons-x-mark-20-solid"
            class="flex sm:hidden absolute end-5 top-5 z-10" square padded @click="navbar = false" />
          <UVerticalNavigation :links="links" :ui="{
            width: 'w-fit',
            active: '',
            inactive: ''
          }">
            <template #default="{ link }">
              <span class="group text-white font-bold text-xl relative flex flex-wrap items-center gap-x-2">
                <span class="flex-grow flex flex-col">
                  <span>{{ link.label }}</span>
                  <span class="flex-grow max-w-0 group-hover:max-w-full transition-all duration-500 h-0.5 bg-white"></span>
                </span>
                <UIcon name="i-heroicons-chevron-right"
                  class="transition-transform group-hover:translate-x-7 duration-500 ease-out flex-grow" />
              </span>
            </template>
          </UVerticalNavigation>
        </div>
      </USlideover>
    </div>
  </div>
</template>

<script setup>
const navbar = ref(false)

const links = [
  {
    label: 'Katalog',
    to: '/produk',
    click: () => navbar.value = false
  }
]
</script>

<style scoped></style>