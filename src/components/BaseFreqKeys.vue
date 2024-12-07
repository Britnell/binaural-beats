<script setup lang="ts">
import { computed, ref } from "vue";

const props = defineProps<{
  modelValue: number;
}>();

const emit = defineEmits(["update:modelValue"]);

const octave = ref(4);

const keys = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"];
const A4_FREQUENCY = 440;

function getNoteFrequency(key: string, oct: number) {
  const noteIndex = keys.indexOf(key);
  const midiNote = noteIndex + 12 * (oct + 1);
  return A4_FREQUENCY * Math.pow(2, (midiNote - 69) / 12);
}

const setNote = (key: string, oct: number) => {
  const f = getNoteFrequency(key, oct);
  emit("update:modelValue", f);
};

const currentNote = computed(() => {
  const relNote = 12 * Math.log2(props.modelValue / A4_FREQUENCY) + 69;
  const offBy = Math.abs(relNote - Math.floor(relNote));
  if (offBy > 0.01) return "";

  const noteIndex = Math.round(relNote);
  const octaveIndex = Math.floor(noteIndex / 12) - 1;
  return `${keys[noteIndex % 12]}${octaveIndex}`;
});
</script>

<template>
  <div class="">
    <div class="space-y-1">
      <div
        v-for="oct in [octave + 1, octave, octave - 1]"
        :key="oct"
        class="flex"
      >
        <button
          v-for="key in keys"
          :key="'#' + key + oct"
          @click="setNote(key, oct)"
          class="w-10 border border-black border-r-0 last:border-r"
          :class="{
            'bg-black text-white': key.includes('#'),
            ' bg-blue-600': `${key}${oct}` === currentNote,
          }"
        >
          {{ key }}{{ oct }}
        </button>
      </div>
    </div>
    <div class="flex gap-2 mt-2">
      <button @click="octave--" class="px-2 py- bg-gray-200">
        Octave Down
      </button>
      <button @click="octave++" class="px-2 py- bg-gray-200">Octave Up</button>
    </div>
  </div>
</template>
