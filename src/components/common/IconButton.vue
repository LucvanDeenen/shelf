<template>
  <div class="group relative inline-flex">
    <button
      :class="['inline-flex items-center justify-center cursor-pointer p-0 bg-transparent', customClass]"
      :title="tooltipText"
      :aria-label="tooltipText"
      @click="$emit('click', $event)"
      type="button"
    >
      <svg :width="size" :height="size" viewBox="0 0 24 24">
        <path :d="icon" fill="currentColor" />
      </svg>
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";

const props = withDefaults(
  defineProps<{
    icon: string;
    title?: string;
    tooltip?: string;
    customClass?: string;
    size?: number;
  }>(),
  { size: 20 },
);

const tooltipText = computed(() => props.tooltip ?? props.title ?? "");
</script>
