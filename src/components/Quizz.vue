<script setup lang="ts">
import { ref, onMounted } from 'vue'

const props = defineProps(['alphabet'])
const alphabetName = props.alphabet

const letters = ref([])
const loaded = ref(false)
const text = ref('')
const done = []
const current = ref(null)


function submit() {
  console.log(text.value, current.value.translation)
  if (text.value == current.value.translation) {
    nextQuestion()
  }
}

interface Letter {
  symbol: string
  translation: string
  prononciation: string
}

function parseAlphabet(data: string): Letter[] {
  let letters = []
  for (let l of data.split('\n')) {
    const match = l.split(';')
    letters.push({
      symbol: match[0],
      translation: match[1],
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
function shuffle(array) {
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
  <h3>{{ alphabetName }}</h3>
  <div v-if="!loaded">

  </div>
  <form class="pure-form">
    <fieldset>
      <legend>{{ current?.symbol }}</legend>
      <input v-model="text" type="text" />
      <button @click="submit" type="submit" class="pure-button pure-button-primary">Submit</button>
    </fieldset>
  </form>
  <ul v-if="loaded">
    <li v-for="letter in letters">
      {{ letter.symbol }}: {{ letter.translation }}
    </li>
  </ul>
</template>
