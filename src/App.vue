<script setup lang="ts">
import Calendar from './components/Calendar.vue'
import Sidebar from './components/Sidebar.vue'
</script>

<template>
  <div class="flex min-h-screen bg-white">
    <Sidebar />
    <Calendar />
  </div>
</template>

<style>
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}
</style>