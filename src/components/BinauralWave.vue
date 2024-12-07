<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from "vue";
import VolumeControl from "./VolumeControl.vue";

const audioContext = ref<AudioContext | null>(null);
const oscillatorLeft = ref<OscillatorNode | null>(null);
const oscillatorRight = ref<OscillatorNode | null>(null);
const gainNode = ref<GainNode | null>(null);
const isPlaying = ref(false);

const baseFrequency = ref(260);
const frequencyDiff = ref(6);
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
  gainNode.value.gain.setValueAtTime(0, audioContext.value.currentTime);

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
  gainNode.value.gain.linearRampToValueAtTime(volume.value, currentTime + 0.5);
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
  <div class="controls grid grid-cols-[1fr_80px] gap-4">
    <div class="freq">
      <div class=" ">
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
      <div class=" ">
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
    </div>

    <VolumeControl v-model:volume.number="volume" />
  </div>
  <div class="flex justify-center">
    <button
      @click="togglePlay"
      class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
    >
      {{ isPlaying ? "Stop" : "Start" }}
    </button>
  </div>
</template>
