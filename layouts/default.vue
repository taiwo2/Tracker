<template>
  <div>
  <div class="bg-gray-100 shadow-md py-2 px-4 flex items-center justify-between">
    <!-- Left section -->
    <div class="flex items-center space-x-4">
      <h1 class="text-xl font-semibold text-gray-800">Example</h1>
      <div class="flex items-center space-x-2">
        <StarIcon class="w-5 h-5 text-gray-500 hover:text-gray-800" />
        <LockClosedIcon class="w-5 h-5 text-gray-500 hover:text-gray-800" />
        <span class="text-gray-600 text-sm">Private</span>
  
        <div class="flex items-center space-x-4">
          <button @click="toggleBoardModal" class="flex items-center bg-gray-300 text-gray-800 px-3 py-1 rounded-lg font-semibold">
            <MenuAlt1Icon class="flex w-5 h-5 mr-3"/>
            <nuxt-link to="/">
              <h3 class="text-xl font-bold">Board</h3>
            </nuxt-link>
          </button>
          <ChevronDownIcon class="w-5 h-5 text-gray-500 hover:text-gray-800" />
        </div>
      </div>
    </div>
  </div>
    <!-- content -->
    <NuxtPage  />
  </div>
  </template>

<script setup>
import { ref } from 'vue';
import { StarIcon, LockClosedIcon, ChevronDownIcon, MenuAlt1Icon } from '@heroicons/vue/outline';



</script>

<style scoped>
.top-bar {
  padding: 1rem;
  background-color: #f5f5f5;
}
</style>
