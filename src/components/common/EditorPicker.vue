<template>
  <div class="relative shrink-0">
    <button
      type="button"
      :title="badgeTitle"
      :class="[
        'h-5 w-5 inline-flex items-center justify-center rounded border transition-colors duration-150 cursor-pointer',
        explicit
          ? 'border-yellow-500/60 bg-yellow-500/10 text-yellow-400 hover:bg-yellow-500/20'
          : selectedId
            ? 'border-yellow-500/30 bg-yellow-500/5 text-yellow-500/60 hover:bg-yellow-500/10 hover:text-yellow-400'
            : 'border-dashed border-neutral-600/50 text-neutral-600 hover:border-yellow-500/30 hover:text-yellow-500/50',
      ]"
      @click="togglePicker"
    >
      <svg width="13" height="13" viewBox="0 0 24 24" aria-hidden="true">
        <path :d="currentIcon" fill="currentColor" />
      </svg>
    </button>

    <div
      v-if="open"
      class="absolute left-0 top-full mt-1 z-50 min-w-[11rem] bg-neutral-900 border border-neutral-700/60 rounded-lg shadow-xl overflow-hidden"
    >
      <div v-if="loading" class="px-3 py-2 text-[0.8rem] text-gray-500">
        Loading…
      </div>
      <template v-else>
        <button
          v-for="editor in editors"
          :key="editor.id"
          type="button"
          :class="[
            'w-full flex items-center gap-2.5 px-3 py-2 text-left text-[0.82rem] transition-colors duration-100 cursor-pointer',
            selectedId === editor.id && explicit
              ? 'bg-yellow-500/10 text-yellow-400'
              : 'text-gray-300 hover:bg-neutral-700/60',
          ]"
          @click="select(editor.id)"
        >
          <svg width="14" height="14" viewBox="0 0 24 24" aria-hidden="true" class="shrink-0">
            <path :d="editorIcon(editor.id)" fill="currentColor" />
          </svg>
          {{ editor.name }}
          <svg v-if="selectedId === editor.id && explicit" width="12" height="12" viewBox="0 0 24 24" aria-hidden="true" class="ml-auto shrink-0">
            <path :d="mdiCheck" fill="currentColor" />
          </svg>
        </button>
        <div v-if="explicit" class="border-t border-neutral-700/50">
          <button
            type="button"
            class="w-full flex items-center gap-2.5 px-3 py-2 text-left text-[0.82rem] text-gray-500 hover:bg-neutral-700/60 hover:text-gray-300 transition-colors duration-100 cursor-pointer"
            @click="clear"
          >
            <svg width="14" height="14" viewBox="0 0 24 24" aria-hidden="true" class="shrink-0">
              <path :d="mdiClose" fill="currentColor" />
            </svg>
            Reset to auto-detect
          </button>
        </div>
      </template>
    </div>

    <!-- Click-outside overlay -->
    <div v-if="open" class="fixed inset-0 z-40" @click="open = false" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { mdiCheck, mdiClose } from "@mdi/js";
import { type EditorInfo, editorIcon } from "@/util/editors";

const props = defineProps<{
  editors: EditorInfo[];
  selectedId?: string;
  explicit?: boolean;
  loading?: boolean;
}>();

const emit = defineEmits<{
  open: [];
  select: [id: string];
  clear: [];
}>();

const open = ref(false);

const currentIcon = computed(() => editorIcon(props.selectedId ?? ""));

const badgeTitle = computed(() => {
  if (props.explicit) return "Set by .shelf — click to change";
  if (props.selectedId) return "Auto-detected — click to override";
  return "Set editor for this project";
});

function togglePicker() {
  open.value = true;
  emit("open");
}

function select(id: string) {
  emit("select", id);
  open.value = false;
}

function clear() {
  emit("clear");
  open.value = false;
}
</script>
