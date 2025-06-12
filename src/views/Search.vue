<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

import InputIcon from 'primevue/inputicon'
import InputText from 'primevue/inputtext'
import ProgressSpinner from 'primevue/progressspinner'

const route = useRoute()
const router = useRouter()

const word = ref('')
const definition = ref(null)
const error = ref(null)
const audioSrc = ref(null)
const syllabified = ref('')
const isLoading = ref(false)
let audioElement = null

onMounted(() => {
  if (route.query.search) {
    word.value = route.query.search
    fetchDefinition()
  }
})

const fetchDefinition = async () => {
  if (!word.value.trim()) return

  router.replace({
    query: { ...route.query, search: word.value }
  })

  isLoading.value = true
  definition.value = null
  error.value = null
  audioSrc.value = null
  syllabified.value = ''

  try {
    const res = await fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${word.value}`)
    if (!res.ok) throw new Error('Word not found')
    const data = await res.json()
    definition.value = data[0]

    const phonetics = definition.value.phonetics || []
    const audio = phonetics.find(p => p.audio)
    if (audio?.audio) {
      audioSrc.value = audio.audio
      audioElement = new Audio(audio.audio)
    }

    syllabified.value = await fetchSyllables(definition.value.word)
  } catch (err) {
    error.value = err.message
  } finally {
    isLoading.value = false
  }
}

const playAudio = () => {
  if (audioElement) {
    audioElement.play()
  }
}

const fetchSyllables = async (word) => {
  try {
    const res = await fetch(`https://api.datamuse.com/words?sp=${word}&md=s`)
    const data = await res.json()
    if (data.length && data[0].numSyllables) {
      const syllables = data[0].numSyllables
      return insertMiddleDots(data[0].word, syllables)
    }
  } catch (err) {
    console.error('Failed to fetch syllables:', err)
  }
  return word
}

const insertMiddleDots = (word, syllables) => {
  if (syllables <= 1 || word.length <= syllables) return word
  const avgLen = Math.floor(word.length / syllables)
  const parts = []
  let i = 0
  for (let s = 1; s < syllables; s++) {
    parts.push(word.slice(i, i + avgLen))
    i += avgLen
  }
  parts.push(word.slice(i))
  return parts.join('·')
}

function goHome() {
  router.push({ name: 'Home' })
}
</script>

<template>
  <div class="flex flex-col justify-center items-center py-[8px]">
    <div class="w-[20rem] md:w-[30rem]">
      <div class="flex justify-between items-center gap-4">
        <i class="pi pi-chevron-left" style="font-size: 1rem" @click="goHome"></i>

        <div class="relative my-8 w-full">
          <InputText
            v-model="word"
            @keyup.enter="fetchDefinition"
            placeholder="Search"
            class="w-full pr-12 py-2 px-4 outline-[#819A91]"
          />
          <InputIcon
            class="pi pi-search absolute right-3 top-1/2 -translate-y-1/2 text-white bg-[#819A91] rounded-full p-2 cursor-pointer"
            @click="fetchDefinition"
          />
        </div>
      </div>

      <div v-if="isLoading" class="card flex justify-center">
        <i class="pi pi-spin pi-spinner-dotted text-[#A7C1A8]" style="font-size: 2rem"></i>
      </div>
      
      <div v-if="error && word.trim()" class="w-full bg-[#D1D8BE] text-gray-600 rounded-xl text-center flex flex-col py-8 gap-4">
        <i class="pi pi-times" style="font-size: 2.5rem"></i>
        {{ error }}
      </div>

      <div v-if="!word" class="w-full bg-[#D1D8BE] text-gray-600 rounded-xl text-center flex flex-col py-8 gap-4">
        <i class="pi pi-search" style="font-size: 2.5rem"></i>
        <p>Search for a word</p>
      </div>

      <div v-else>
        <div v-if="definition && word.trim()">
        <div class="flex justify-between items-center">
          <h2 class="font-bold text-3xl md:text-5xl w-[90%] break-words">{{ syllabified || definition.word }}</h2>
          <button v-if="audioSrc" class="pi pi-volume-up" style="font-size: 2rem" @click="playAudio" aria-label="Play pronunciation">
          </button>
        </div>

        <div v-if="definition.phonetic" class="mt-4 italic">
          {{ definition.phonetic }}
        </div>

        <div v-if="definition.origin" class="italic">
          <strong>Origin:</strong> {{ definition.origin }}
        </div>

        <div v-if="definition.meanings.length" class="mt-[2rem]">
          <div v-for="(meaning, index) in definition.meanings" :key="index" class="my-4 bg-[#D1D8BE] p-4 rounded-xl">
            <h4 class="font-semibold text-[#555] text-xl md:text-2xl">{{ meaning.partOfSpeech }}</h4>
            <div v-for="(def, i) in meaning.definitions" :key="i" class="mt-4">
              <p><strong>{{ i + 1 }}.</strong> {{ def.definition }}</p>
              <p v-if="def.example" class="ml-4 mt-4 text-[#4a5759]"><em>{{ def.example }}</em></p>
            </div>
          </div>
        </div>
      </div>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>