<script setup lang="ts">
import { ref } from "vue";

const props = defineProps<{
  tabs: string[];
}>();

const activeTab = ref(props.tabs[0]);

const switchTab = (tab: string) => {
  activeTab.value = tab;
};
</script>

<template>
  <div class="">
    <div class="">
      <button
        v-for="tab in props.tabs"
        :key="tab"
        class="px-2 py-1"
        :class="[activeTab === tab ? '  bg-blue-200' : ' bg-gray-200']"
        @click="switchTab(tab)"
      >
        {{ tab }}
      </button>
    </div>
    <div class=" ">
      <template v-for="tab in tabs" :key="tab">
        <div
          v-show="activeTab === tab"
          class="min-h-[100px] w-full flex items-center"
        >
          <slot :name="tab"></slot>
        </div>
      </template>
    </div>
  </div>
</template>
