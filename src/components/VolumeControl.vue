<script setup lang="ts">
import { Ref, toRefs } from "vue";

defineProps<{
  volume: number;
}>();

const emit = defineEmits(["update:volume"]);
</script>
<template>
  <div class="volume-control flex flex-col items-center gap-2">
    <input
      type="range"
      :value="volume"
      @input="
        emit(
          'update:volume',
          ($event.target as HTMLInputElement)?.value ?? '0.3'
        )
      "
      min="0"
      max="1"
      step="0.01"
      class="h-32 appearance-none bg-gray-200 rounded-lg cursor-pointer"
      style="writing-mode: bt-lr; -webkit-appearance: slider-vertical"
    />
    <div class="text-sm font-medium text-gray-700">
      {{ Math.round(volume * 100) }}%
    </div>
  </div>
</template>

<style scoped></style>
