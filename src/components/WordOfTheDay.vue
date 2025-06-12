<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const wordOfTheDay = ref('')
const router = useRouter()

function goToSearch() {
  if (!wordOfTheDay.value) return
  router.push({ name: 'Search', query: { search: wordOfTheDay.value } })
}

async function fetchRandomWord() {
  const res = await fetch('https://random-word-api.herokuapp.com/word?number=1')
  const data = await res.json()
  return data[0]
}

async function fetchWordOfTheDay() {
  const today = new Date().toISOString().split('T')[0]
  const cached = JSON.parse(localStorage.getItem('wordOfTheDay') || '{}')

  if (cached.date === today && cached.word) {
    wordOfTheDay.value = cached.word
    return
  }

  let validWord = ''
  for (let attempts = 0; attempts < 10; attempts++) {
    try {
      const randomWord = await fetchRandomWord()
      const defRes = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${randomWord}`)

      if (defRes.ok) {
        validWord = randomWord
        break
      }
    } catch (error) {
      console.error('Error validating word:', error)
    }
  }

  if (validWord) {
    wordOfTheDay.value = validWord
    localStorage.setItem('wordOfTheDay', JSON.stringify({ date: today, word: validWord }))
  } else {
    console.warn('No valid word found after multiple attempts.')
  }
}

onMounted(() => {
  fetchWordOfTheDay()
})
</script>

<template>
  <div v-if="wordOfTheDay" class="mt-6 text-lg w-full bg-[#D1D8BE] rounded-xl p-[20px]" @click="goToSearch">
    <p
      class="text-[#819A91] text-left text-3xl md:text-5xl w-[90%] break-words font-bold cursor-pointer hover:underline"
      @click="goToSearch"
    >
      {{ wordOfTheDay }}.
    </p>
    <div class="w-full border-2 border-t border-[#819A91] my-4"></div>

    <p class="font-semibold text-gray-600">word of the day</p>
  </div>
</template>
