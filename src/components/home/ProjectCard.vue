<template>
  <div
    class="p-4 border border-neutral-700/50 rounded-lg bg-neutral-800/40 cursor-pointer transition-colors duration-[160ms] hover:border-yellow-500/60 hover:bg-neutral-700/50"
    @click="emit('open', project.path, editorHint)"
  >
    <div class="min-w-0 w-full">
      <div class="flex justify-between items-center gap-2">
        <!-- Editor icon (picker trigger) + project name -->
        <div class="flex items-center gap-[0.3rem] min-w-0" @click.stop>
          <EditorPicker
            :editors="availableEditors"
            :selected-id="editorHint"
            :explicit="editorExplicit"
            :loading="loadingEditors"
            @open="loadEditors"
            @select="setEditor"
            @clear="clearEditor"
          />

          <p
            class="m-0 text-white text-sm font-semibold truncate"
            @click.stop="emit('open', project.path, editorHint)"
          >
            {{ project.name }}
          </p>
        </div>

        <!-- Branch -->
        <IconLabel
          v-if="project.branch"
          :icon="mdiSourceBranch"
          :clickable="true"
          title="Open repository"
          class="text-[0.8rem] text-gray-500 whitespace-nowrap shrink-0 transition-colors duration-150 hover:text-gray-300"
          @click.stop="openRepository"
        >
          {{ project.branch }}
        </IconLabel>
      </div>

      <div class="flex items-center gap-2 mt-[0.1rem] min-w-0">
        <IconButton
          :icon="mdiMicrosoftVisualStudioCode"
          :size="13"
          title="Open in VS Code"
          custom-class="h-5 w-5 rounded border border-yellow-500/60 bg-yellow-500/10 text-yellow-400 transition-colors duration-150 hover:bg-yellow-500/20"
          @click.stop="openInVSCode"
        ></IconButton>
        <IconLabel
          :icon="mdiFolder"
          :clickable="true"
          :tooltip="relativePath"
          class="text-[0.8rem] font-bold text-gray-500 min-w-0 overflow-hidden transition-colors duration-150 hover:text-gray-300"
          title="Open folder"
          @click.stop="openFolder"
        >
          {{ relativePath }}
        </IconLabel>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import {
  mdiSourceBranch,
  mdiFolder,
  mdiMicrosoftVisualStudioCode,
} from "@mdi/js";
import IconLabel from "@/components/common/IconLabel.vue";
import EditorPicker from "@/components/common/EditorPicker.vue";
import { type EditorInfo } from "@/util/editors";
import IconButton from "../common/IconButton.vue";

const props = defineProps<{
  project: {
    name: string;
    path: string;
    branch?: string;
    group: string;
    editorHint?: string;
    editorExplicit?: boolean;
  };
}>();

const emit = defineEmits<{
  open: [path: string, editorHint?: string];
}>();

// autoHint is the hint from workspace file detection (no explicit override).
// We preserve it so "reset to auto-detect" can restore it without a reload.
const autoHint = props.project.editorExplicit
  ? undefined
  : props.project.editorHint;
const editorHint = ref(props.project.editorHint);
const editorExplicit = ref(props.project.editorExplicit ?? false);
const availableEditors = ref<EditorInfo[]>([]);
const loadingEditors = ref(false);

const relativePath = computed(() => {
  const p = props.project.path.replace(/\\/g, "/");
  const parts = p.split("/");
  return parts.length >= 2 ? parts[parts.length - 2] : p;
});

async function loadEditors() {
  if (availableEditors.value.length === 0) {
    loadingEditors.value = true;
    try {
      availableEditors.value = await window.api.fs.getInstalledEditors();
    } finally {
      loadingEditors.value = false;
    }
  }
}

async function setEditor(id: string) {
  await (window.api.settings as any).setProjectEditor(props.project.path, id);
  editorHint.value = id;
  editorExplicit.value = true;
}

async function clearEditor() {
  await (window.api.settings as any).setProjectEditor(props.project.path, null);
  editorHint.value = autoHint;
  editorExplicit.value = false;
}

async function openInVSCode() {
  await window.api.fs.openInEditor(props.project.path, "vscode");
}

async function openFolder() {
  await (window.api.fs as any).openFolder(props.project.path);
}

async function openRepository() {
  await (window.api.fs as any).openRepository(props.project.path);
}
</script>
