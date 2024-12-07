<script setup lang="ts">
import { ref } from "vue";
import BinauralWave from "./components/BinauralWave.vue";

const a = ref();
</script>
<template>
  <div>
    <header>
      <h1>App</h1>
    </header>
    <BinauralWave />
  </div>
</template>

<style scoped></style>
