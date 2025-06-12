<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import WordOfTheDay from '../components/WordOfTheDay.vue'

import InputIcon from 'primevue/inputicon'
import InputText from 'primevue/inputtext'

const word = ref('')
const router = useRouter()

function goToSearch() {
  router.push({ name: 'Search', query: { search: word.value } })
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center">
    <div class="w-[18rem] md:w-[25rem]">
      <img src="../assets/joshionary-logo.png" alt="joshionary-logo">

      <div class="relative mt-4 mb-8">
        <input
          v-model="word"
          @keyup.enter="goToSearch"
          placeholder="Search"
          class="w-full pr-4 py-3 px-4 outline-[#819A91] hover:outline-[#819A91] bg-white rounded-xl text-black focus:outline-2 focus:outline-[#819A91]"
        />
        <InputIcon
          class="pi pi-search absolute right-3 top-1/2 -translate-y-1/2 text-white bg-[#819A91] rounded-full p-2 cursor-pointer"
          @click="goToSearch"
        />
      </div>

      <WordOfTheDay />
    </div>
  </div>
</template>
