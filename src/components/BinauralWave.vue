<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from "vue";

const audioContext = ref<AudioContext | null>(null);
const oscillatorLeft = ref<OscillatorNode | null>(null);
const oscillatorRight = ref<OscillatorNode | null>(null);
const gainNode = ref<GainNode | null>(null);
const isPlaying = ref(false);

const baseFrequency = ref(200);
const frequencyDiff = ref(10);
const volume = ref(0.5);

const initAudio = () => {
  audioContext.value = new AudioContext();

  oscillatorLeft.value = audioContext.value.createOscillator();
  oscillatorRight.value = audioContext.value.createOscillator();

  const pannerLeft = audioContext.value.createStereoPanner();
  const pannerRight = audioContext.value.createStereoPanner();
  pannerLeft.pan.value = -1;
  pannerRight.pan.value = 1;

  gainNode.value = audioContext.value.createGain();

  oscillatorLeft.value.connect(pannerLeft);
  oscillatorRight.value.connect(pannerRight);
  pannerLeft.connect(gainNode.value);
  pannerRight.connect(gainNode.value);
  gainNode.value.connect(audioContext.value.destination);

  updateFrequencies();
  updateVolume();

  oscillatorLeft.value.start();
  oscillatorRight.value.start();
};

const updateFrequencies = () => {
  if (!oscillatorLeft.value || !oscillatorRight.value || !audioContext.value)
    return;

  const currentTime = audioContext.value.currentTime;
  oscillatorLeft.value.frequency.setValueAtTime(
    baseFrequency.value,
    currentTime
  );
  oscillatorRight.value.frequency.setValueAtTime(
    baseFrequency.value + frequencyDiff.value,
    currentTime
  );
};

const updateVolume = () => {
  if (!gainNode.value || !audioContext.value) return;

  const currentTime = audioContext.value.currentTime;
  gainNode.value.gain.setValueAtTime(volume.value, currentTime);
};

watch([baseFrequency, frequencyDiff], () => {
  updateFrequencies();
});

watch(volume, () => {
  updateVolume();
});

const togglePlay = async () => {
  if (!audioContext.value) {
    await initAudio();
    if (!audioContext.value) return;
  }

  isPlaying.value = !isPlaying.value;
  if (!isPlaying.value) {
    await audioContext.value.suspend();
  } else {
    await audioContext.value.resume();
  }
};

const onKeyPress = (ev: KeyboardEvent) => {
  if (ev.code === "Space") {
    togglePlay();
  }
};

onMounted(() => {
  window.addEventListener("keydown", onKeyPress);
});

onUnmounted(() => {
  window.removeEventListener("keydown", onKeyPress);

  if (audioContext.value) {
    audioContext.value.close();
  }
});
</script>

<template>
  <div class="binaural-wave p-4">
    <div class="controls space-y-4">
      <div class="frequency-control">
        <label class="block text-sm font-medium text-gray-700"
          >Base Frequency: {{ baseFrequency }}Hz</label
        >
        <input
          type="range"
          v-model.number="baseFrequency"
          min="20"
          max="500"
          class="w-full"
        />
      </div>
      <div class="frequency-control">
        <label class="block text-sm font-medium text-gray-700"
          >Frequency Difference: {{ frequencyDiff }}Hz</label
        >
        <input
          type="range"
          v-model.number="frequencyDiff"
          min="1"
          max="40"
          class="w-full"
        />
      </div>
      <div class="volume-control">
        <label class="block text-sm font-medium text-gray-700"
          >Volume: {{ Math.round(volume * 100) }}%</label
        >
        <input
          type="range"
          v-model.number="volume"
          min="0"
          max="1"
          step="0.01"
          class="w-full"
        />
      </div>

      <button
        @click="togglePlay"
        class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
      >
        {{ isPlaying ? "Stop" : "Start" }}
      </button>
    </div>
  </div>
</template>
