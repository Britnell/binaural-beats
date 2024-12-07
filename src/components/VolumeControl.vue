<script setup lang="ts">
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
      class=" "
      style="writing-mode: vertical-lr; direction: rtl"
    />
    <div class="text-sm font-medium text-gray-700">
      {{ Math.round(volume * 100) }}%
    </div>
  </div>
</template>

<style scoped></style>
