<template>
  <div class="flex h-screen">
    <div class="flex-1 flex flex-col overflow-hidden">
      <DashboardHeader />
      <div class="flex h-full">
        <Sidebar />
        <div class="container px-4 py-10 flex justify-center overflow-x-hidden overflow-y-auto">
          <slot />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>

</script>

<style scoped></style>