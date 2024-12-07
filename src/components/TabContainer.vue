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
  <div>
    <div class="border border-gray-300">
      <div class="flex border border-gray-300 w-fit">
        <button
          v-for="tab in props.tabs"
          :key="tab"
          class="px-2 py-1"
          :class="[activeTab === tab ? ' ' : 'bg-gray-300 hover:bg-gray-100']"
          @click="switchTab(tab)"
        >
          {{ tab }}
        </button>
      </div>
      <div class="p-2">
        <template v-for="tab in tabs" :key="tab">
          <div v-show="activeTab === tab" class="min-h-[90px] flex flex-col">
            <slot :name="tab"></slot>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
