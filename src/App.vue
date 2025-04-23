<script setup lang="ts">
import Quizz from './components/Quizz.vue'
</script>

<template>
  <div class="layout">
    <header>
      <div class="pure-menu pure-menu-horizontal">
        <a href="#" class="pure-menu-heading pure-menu-link green">
          <h3>🥸 Estez</h3>
        </a>
        <ul class="pure-menu-list">
          <li class="pure-menu-item">
            <a href="#" class="pure-menu-link">Hiragana</a>
          </li>
        </ul>
      </div>
    </header>

    <main>
      <div class="content">
        <Quizz alphabet="hiragana" />
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
