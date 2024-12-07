<script setup lang="ts">
import { ref } from "vue";
import BinauralWave from "./components/BinauralWave.vue";

const a = ref();
</script>
<template>
  <div class="max-w-3xl mx-auto px-8">
    <header class="py-4">
      <h1>Binaural Beats</h1>
    </header>
    <div>
      <BinauralWave />
    </div>
  </div>
</template>

<style scoped></style>
