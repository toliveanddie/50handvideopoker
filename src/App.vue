<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <div class="poker-app">
    <HelloWorld />
  </div>
</template>

<style>
body {
  background-color: #f5f5f5;
  padding: 20px 0;
}

.poker-app {
  background-color: white;
  padding: 30px;
  border-radius: 15px;
  box-shadow: 0 10px 20px rgba(0,0,0,0.1);
  max-width: 1200px;
  margin: 0 auto;
}
</style>
