<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from "vue";
import VolumeControl from "./VolumeControl.vue";
import BaseFrequencyControl from "./BaseFrequencyControl.vue";
import TabContainer from "./TabContainer.vue";
import BaseFreqKeys from "./BaseFreqKeys.vue";

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
  gainNode.value.gain.linearRampToValueAtTime(volume.value, currentTime + 0.2);
};

watch([baseFrequency, frequencyDiff], () => {
  updateFrequencies();
});

watch(volume, () => {
  updateVolume();
});

const fadeGainPromise = (gain: number, sec: number) => {
  return new Promise((resolve) => {
    if (!audioContext.value) return;
    gainNode.value?.gain.linearRampToValueAtTime(
      gain,
      audioContext.value.currentTime + sec
    );
    setTimeout(() => {
      resolve(true);
    }, sec * 1000);
  });
};

const onOffFade = 0.8;
const togglePlay = async () => {
  if (!audioContext.value) {
    await initAudio();
    if (!audioContext.value) return;
  }

  const stop = isPlaying.value;
  if (stop) {
    fadeGainPromise(0, onOffFade).then(() => audioContext.value?.suspend());
  } else {
    gainNode.value?.gain.setValueAtTime(0, audioContext.value.currentTime);
    audioContext.value
      .resume()
      .then(() => fadeGainPromise(volume.value, onOffFade));
  }
  isPlaying.value = !isPlaying.value;
};

const onKeyPress = (ev: KeyboardEvent) => {
  if (ev.code === "Space") {
    // togglePlay();
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
  <div class="controls sm:grid grid-cols-[auto_80px] gap-2">
    <div class="row-span-2 space-y-4">
      <div>
        <h2 class="font-bold text-lg mb-1">Binaural Frequency</h2>
        <div class="border border-gray-300 p-2 flex flex-col items-center">
          <label class="block text-gray-700"
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
      <div>
        <h2 class="font-bold text-lg mb-1">Base Frequency</h2>
        <TabContainer :tabs="['Notes', 'Hz']">
          <template #Hz>
            <div class="grow grid place-items-center">
              <BaseFrequencyControl v-model.number="baseFrequency" />
            </div>
          </template>
          <template #Notes>
            <BaseFreqKeys v-model.number="baseFrequency" />
          </template>
        </TabContainer>
      </div>
    </div>
    <VolumeControl v-model:volume.number="volume" />
    <div class="place-self-center">
      <button
        @click="togglePlay"
        class="size-14 sm:size-16 rounded-full bg-blue-500 text-white sm:text-lg hover:bg-blue-600 transition-colors"
      >
        {{ isPlaying ? "Stop" : "Start" }}
      </button>
    </div>
  </div>
</template>
