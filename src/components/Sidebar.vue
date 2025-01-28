<script setup lang="ts">
const tasks = [
  { id: 1, title: 'Call Nicolas', icon: '📞' },
  { id: 2, title: 'Come back to designer', icon: '📧' },
  { id: 3, title: 'Schedule marketing brainstorm', icon: '📅' },
  { id: 4, title: 'Complete iOS feedback', icon: '📱' },
  { id: 5, title: 'Close Julien project', icon: '📂' },
  { id: 6, title: 'Create template file', icon: '📄' },
  { id: 7, title: 'Re: Employee feedback', icon: '📧' }
]
</script>

<template>
  <div class="w-64 border-r border-gray-200 p-4">
    <div class="flex flex-col space-y-4">
      <div v-for="task in tasks" :key="task.id" 
           class="flex items-center space-x-2 p-2 hover:bg-gray-100 rounded cursor-pointer">
        <input type="checkbox" class="rounded">
        <span>{{ task.title }}</span>
        <span class="ml-auto">{{ task.icon }}</span>
      </div>
    </div>
  </div>
</template>