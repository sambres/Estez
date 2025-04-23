<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Letter {
  symbol: string
  translation: string[]
  prononciation: string
}

const props = defineProps(['alphabet'])
const alphabetName = props.alphabet

const letters = ref<Letter[]>([])
const loaded = ref(false)
const text = ref('')
const done = []
const current = ref<Letter | null>(null)
const result = ref<boolean | null>(null)


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
}

function parseAlphabet(data: string): Letter[] {
  let letters = []
  for (let l of data.split('\n')) {
    const match = l.split(';')
    letters.push({
      symbol: match[0],
      translation: match[1].split('|'),
      prononciation: match[2]
    })
  }
  return letters
}

function fetchAlphabet() {
  const url = `/data/${alphabetName}.csv`
  return fetch(url).
    then(res => res.text()).
    then((data) => {
      letters.value = shuffle(parseAlphabet(data))
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
  current.value = letters.value[done.length]
}

function nextQuestion() {
  done.push(current.value)
  current.value = letters.value[done.length]
  text.value = ""
}

onMounted(() => {
  fetchAlphabet().then(startQuizz)
})

</script>

<template>
  <!-- <h3 class="alphabet-title">{{ alphabetName }}</h3> -->
  <div v-if="!loaded">

  </div>
  <form class="pure-form">
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
        <p>{{ done.length }}/ {{ letters.length }}</p>
      </div>
    </fieldset>

  </form>
  <ul v-if="loaded">
    <li v-for="letter in letters">
      {{ letter.symbol }}: {{ letter.translation }}
    </li>
  </ul>
</template>



<style scoped>
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

button[type="submit"] {
  border-radius: 4px
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
</style>
