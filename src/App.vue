<script setup lang="ts">
import { ref, computed, type Component, createApp, createCommentVNode } from 'vue'
import Quizz from './components/Quizz.vue'

interface Routes {
  [path: string]: Route
}
interface Route {
  component: Component
  props: { [k: string]: string }
}
createApp
const routes: Routes = {
  "/": { component: Quizz, props: { alphabet: "hiragana" } },
  "/hiragana": { component: Quizz, props: { alphabet: "hiragana" } },
  "/katakana": { component: Quizz, props: { alphabet: "katakana" } },
  "/hangeul": { component: Quizz, props: { alphabet: "hangeul" } },
  "/abjad": { component: Quizz, props: { alphabet: "abjad" } }
}

const currentPath = ref(window.location.hash.slice(1))

window.addEventListener('hashchange', () => {
  currentPath.value = window.location?.hash?.slice(1)
})

const currentRoute = () => {
  return routes[currentPath.value || "/"]
}
const currentView = computed<Component>(() => routes[currentPath.value || "/"].component)
const currentProperties = computed<Component>(() => {
  return routes[currentPath.value || "/"].props
})


</script>

<template>
  <div class="layout">
    <header>
      <div class="pure-menu pure-menu-horizontal">
        <a href="#" class="pure-menu-heading pure-menu-link green">
          <h3>Estez</h3>
        </a>
        <ul class="pure-menu-list">
          <li class="pure-menu-item">
            <a href="#" class="pure-menu-link">Hiragana</a>
          </li>
          <li class="pure-menu-item">
            <a href="#/katakana" class="pure-menu-link">Katakana</a>
          </li>
          <li class="pure-menu-item">
            <a href="#/hangeul" class="pure-menu-link">Hangeul</a>
          </li>
          <li class="pure-menu-item">
            <a href="#/abjad" class="pure-menu-link">Abjad</a>
          </li>
        </ul>
      </div>
    </header>

    <main>
      <div class="content">
        <keep-alive>
          <component :is="currentView" :key="currentPath" v-bind="currentProperties" />
        </keep-alive>
      </div>
    </main>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.green {
  color: hsla(160, 100%, 37%, 1);
}
</style>
