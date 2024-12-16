<script setup lang="ts">
import Tensorflow from "./components/Tensorflow.vue";
</script>

<template>
  <div>
    <h1>This is faceRecognition with tensorflow</h1>
  </div>
  <Tensorflow />
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
