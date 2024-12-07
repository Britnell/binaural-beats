<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch, computed } from "vue";
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

const bands = {
  delta: 4,
  theta: 8,
  alpha: 13,
  beta: 30,
  gamma: 50,
};

const bandKeys = Object.keys(bands) as (keyof typeof bands)[];

const currentBand = computed(() => {
  for (const key of bandKeys) {
    if (frequencyDiff.value < bands[key]) return key;
  }
  return "";
});
</script>

<template>
  <div class="controls sm:grid grid-cols-[auto_80px] gap-2">
    <div class="row-span-2 xxxmax-w-[550px] space-y-4">
      <div class="sm:border border-gray-300 p-2 sm:p-4">
        <div class="flex mb-2 items-center gap-4">
          <h2 class="text-lg">Binaural Frequency</h2>
        </div>
        <div class="flex flex-wrap divide-x divide-solid divide-gray-400">
          <div
            v-for="[band, max] in Object.entries(bands)"
            :key="band"
            class="grow px-4 flex-[1_0_auto]"
            :class="{
              'bg-blue-100': currentBand === band,
              'bg-gray-100': currentBand !== band,
            }"
          >
            <div class="py-1 text-center flex flex-col text-sm">
              <span>
                {{ band }}
              </span>
              <span> {{ `< ${max} Hz` }}</span>
            </div>
          </div>
        </div>
        <div class="pt-3">
          <label class="block text-center text-slate-800" for="freqdiff">
            {{ frequencyDiff }}Hz</label
          >
          <input
            type="range"
            id="freqdiff"
            v-model.number="frequencyDiff"
            min="1"
            max="40"
            class="w-full"
          />
        </div>
      </div>
      <div>
        <TabContainer :tabs="['Notes', 'Hz']">
          <template #header>
            <h2 class="text-lg sm:px-2">Base Frequency</h2>
          </template>
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
