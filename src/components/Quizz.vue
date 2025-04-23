<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Letter {
  symbol: string
  translation: string[]
  prononciation: string
  tags: string[]
}

const props = defineProps(['alphabet'])
const alphabetName = props.alphabet

let alphabet: Letter[] = []
const letters = ref<Letter[]>([])
const loaded = ref(false)
const text = ref('')
let done: Letter[] = []
const current = ref<Letter | null>(null)
const result = ref<boolean | null>(null)
const options = ref<Map<string, boolean>>(new Map())
const crowns = ref(0)

let timer: number
function fadeResult(res: boolean) {
  result.value = res
  clearTimeout(timer)
  timer = setTimeout(() => {
    result.value = null
  }, 1000)
}

function submit() {
  if (current.value && current.value.translation.includes(text.value)) {
    nextQuestion()
    fadeResult(true)
  } else {
    fadeResult(false)
  }
  if (done.length == letters.value.length) {
    crowns.value++
    resetAlphabet()
  }
}

function setOption(option: string) {
  const val = options.value.get(option)
  console.log(option, val)
  if (val == undefined) {
    return
  }
  options.value.set(option, !val)
  resetAlphabet()
}

function resetAlphabet() {
  letters.value = shuffle(alphabet.filter(l => l.tags.every(t => options.value.get(t))))
  startQuizz()
}

function parseAlphabet(data: string): Letter[] {
  let letters = []
  const map = new Map<string, boolean>()
  for (let l of data.split('\n')) {
    const match = l.split(';')
    const tags = match[4].split("|")
    letters.push({
      symbol: match[0],
      translation: match[1].split('|'),
      prononciation: match[2],
      tags: tags
    })
    for (const tag of tags) {
      if (!map.has(tag)) {
        map.set(tag, true)
      }
    }
  }
  options.value = map
  return letters
}

function fetchAlphabet() {
  const url = `data/${alphabetName}.csv`
  return fetch(url).
    then(res => res.text()).
    then((data) => {
      alphabet = parseAlphabet(data)
      letters.value = shuffle(alphabet)
      loaded.value = true
    })
}

// https://stackoverflow.com/questions/2450954/how-to-randomize-shuffle-a-javascript-array
function shuffle<T>(array: T[]) {
  let currentIndex = array.length;
  while (currentIndex != 0) {
    let randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex--;
    [array[currentIndex], array[randomIndex]] = [
      array[randomIndex], array[currentIndex]];
  }
  return array
}

function startQuizz() {
  done = []
  current.value = letters.value[done.length]
}

function nextQuestion() {
  done.push(current.value as Letter)
  current.value = letters.value[done.length]
  text.value = ""
}

onMounted(() => {
  fetchAlphabet().then(startQuizz)
})

</script>

<template>
  <!-- <h3 class="alphabet-title">{{ alphabetName }}</h3> -->
  <div class="alphabet-loader" v-if="!loaded">
    <h1>
      🧐
    </h1>
  </div>
  <div v-if="loaded">

    <div class="alphabet-options">
      <template v-for="option in options">
        <button class="github-button" :class="{ 'primary': option[1] }" @click="setOption(option[0])">
          <span v-if="option[1]">✔️</span>
          {{ option[0] }}
          Kana</button>
      </template>
    </div>
    <form class="pure-form" @submit.prevent>
      <fieldset>
        <legend>{{ current?.symbol }}</legend>
        <input v-model="text" type="text" />
        <button @click="submit" type="submit" class="pure-button pure-button-primary">></button>
        <Transition>
          <span class="result-icon success" v-if="result == true">✅</span>
        </Transition>
        <Transition>
          <span class="result-icon wrong" v-if="result == false">❌</span>
        </Transition>
        <div class="quizz-result">
          <p>{{ done.length }}/ {{ letters.length }}

            <template v-for="i in crowns">
              <span class="crown">👑</span>
            </template>
          </p>
        </div>
      </fieldset>
    </form>
    <ul>
      <li v-for="letter in letters.slice(done.length)">
        {{ letter.symbol }}: {{ letter.translation.join(", ") }} ({{ letter.tags.join(", ") }})
      </li>
    </ul>
  </div>

</template>



<style scoped>
.alphabet-loader {
  font-size: 4vh;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  transform: -webkit-translate(-50%, -50%);
  transform: -moz-translate(-50%, -50%);
  transform: -ms-translate(-50%, -50%);
}

.alphabet-loader h1 {
  animation: spin 4s linear infinite
}

.alphabet-title {
  text-transform: capitalize;
}

form {
  margin-top: 8vh;
}

fieldset {
  text-align: center;
}

legend {
  font-size: 250%;
  border: none;
}

.quizz-result .crown {
  font-size: 16pt;
}

.alphabet-options button {
  font-size: 10pt;
  margin-right: 4px;
  text-transform: capitalize
}

.result-icon {
  display: inline-block;
  font-size: 18pt;
  vertical-align: middle;
}

.result-icon.wrong.v-enter-active {
  animation: horizontal-shaking 0.75s
}

.result-icon.success.v-enter-active {
  animation: bounce-up 1.5s
}


.result-icon.v-leave-active {
  transition: opacity 0.5s ease;
}

.result-icon.v-leave-to {
  opacity: 0;
}

@keyframes horizontal-shaking {
  0% {
    transform: translateX(0)
  }

  25% {
    transform: translateX(4px)
  }

  50% {
    transform: translateX(-4px)
  }

  75% {
    transform: translateX(4px)
  }

  100% {
    transform: translateX(0)
  }
}

@keyframes bounce-up {
  0% {
    transform: translateY(0%);
  }

  10% {
    transform: translateY(-8px);
  }

  20% {
    transform: translateY(0%);
  }

  25% {
    transform: translateY(-7%);
  }

  27% {
    transform: translateY(0%);
  }

  29% {
    transform: translateY(-3%);
  }

  30% {
    transform: translateY(0);
  }
}

@keyframes spin {
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
</style>
