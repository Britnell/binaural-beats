<script setup lang="ts">
defineProps<{
  modelValue: number;
}>();

const emit = defineEmits(["update:modelValue"]);
</script>

<template>
  <div class="w-full">
    <label class="block text-sm font-medium text-gray-700"
      >Base Frequency: {{ modelValue }}Hz</label
    >
    <input
      type="range"
      :value="modelValue"
      @input="$event => emit('update:modelValue', (($event.target as HTMLInputElement).value))"
      min="20"
      max="500"
      class="w-full"
    />
  </div>
</template>
