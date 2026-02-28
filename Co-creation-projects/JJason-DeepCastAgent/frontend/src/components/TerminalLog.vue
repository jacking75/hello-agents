<template>
  <div class="macos-terminal rounded-xl shadow-2xl overflow-hidden" style="height: 500px;">
    <!-- macOS Title Bar -->
    <div class="macos-titlebar bg-gradient-to-b from-[#3d3d3d] to-[#2d2d2d] px-4 py-3 flex items-center shrink-0 border-b border-[#1a1a1a]">
      <!-- Traffic Lights -->
      <div class="flex items-center gap-2 mr-4">
        <div class="w-3 h-3 rounded-full bg-[#ff5f57] shadow-inner hover:brightness-110 cursor-pointer transition-all" title="关闭"></div>
        <div class="w-3 h-3 rounded-full bg-[#febc2e] shadow-inner hover:brightness-110 cursor-pointer transition-all" title="最小化"></div>
        <div class="w-3 h-3 rounded-full bg-[#28c840] shadow-inner hover:brightness-110 cursor-pointer transition-all" title="最大化"></div>
      </div>
      <!-- Title -->
      <div class="flex-1 text-center">
        <span class="text-[#9a9a9a] text-sm font-medium tracking-wide">deepcast — zsh — {{ logs.length }} lines</span>
      </div>
      <!-- Placeholder for symmetry -->
      <div class="w-16"></div>
    </div>
    <!-- Terminal Content -->
    <div class="bg-[#1e1e1e] overflow-y-auto p-4 flex-1 font-mono text-sm custom-scrollbar terminal-content" ref="logContainer" style="height: calc(100% - 44px);">
      <!-- Welcome Message -->
      <div v-if="logs.length === 0 && !isWaiting" class="text-[#6a9955] mb-2">
        <span class="text-[#569cd6]">deepcast</span><span class="text-[#d4d4d4]">@</span><span class="text-[#4ec9b0]">studio</span> <span class="text-[#d4d4d4]">~</span> <span class="text-[#dcdcaa]">ready</span>
      </div>
      <!-- Log Entries -->
      <div v-for="(log, i) in logs" :key="i" class="mb-1 leading-relaxed" :class="getLogClass(log.message)">
        <span class="text-[#6a6a6a] mr-2 text-xs select-none">[{{ log.time }}]</span>
        <span class="terminal-text">{{ log.message }}</span>
      </div>
      <!-- Waiting States -->
      <div v-if="isWaiting && logs.length === 0" class="text-[#dcdcaa] text-center mt-8">
        <span class="inline-block animate-pulse">⏳ 正在初始化...</span>
      </div>
      <div v-else-if="isWaiting" class="text-[#dcdcaa] mt-2 flex items-center gap-2">
        <span class="inline-block w-2 h-4 bg-[#569cd6] animate-blink"></span>
        <span>处理中{{ waitingDots }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, watch, nextTick } from "vue";

export interface LogEntry {
  time: string;
  message: string;
}

defineProps<{
  logs: LogEntry[];
  isWaiting: boolean;
  waitingDots: string;
}>();

const logContainer = ref<HTMLElement | null>(null);

/** 当 logs 变化时自动滚动到底部 */
watch(
  () => logContainer.value,
  () => {},
  { flush: "post" }
);

defineExpose({ scrollToBottom });

function scrollToBottom() {
  nextTick(() => {
    if (logContainer.value) {
      logContainer.value.scrollTop = logContainer.value.scrollHeight;
    }
  });
}

function getLogClass(message: string): string {
  if (message.includes("[STAGE]")) return "terminal-stage";
  if (message.includes("[TASK")) return "terminal-info";
  if (message.includes("[TOOL]")) return "terminal-tool";
  if (message.includes("[SOURCES]")) return "terminal-warning";
  if (message.includes("✅") || message.includes("status=completed")) return "terminal-success";
  if (message.includes("❌") || message.includes("ERROR") || message.includes("failed")) return "terminal-error";
  if (message.includes("⚠️") || message.includes("WARNING")) return "terminal-warning";
  if (message.includes("INFO:")) return "terminal-muted";
  if (message.includes("━")) return "terminal-divider";
  return "terminal-default";
}
</script>

<style scoped>
.macos-terminal {
  background: #1e1e1e;
  border: 1px solid #3d3d3d;
  box-shadow:
    0 22px 70px 4px rgba(0, 0, 0, 0.56),
    0 0 0 1px rgba(0, 0, 0, 0.3);
}

.macos-titlebar {
  -webkit-app-region: drag;
  user-select: none;
}

.terminal-content {
  font-family: 'SF Mono', 'Monaco', 'Menlo', 'Consolas', monospace;
  font-size: 13px;
  line-height: 1.6;
}

.terminal-stage { color: #569cd6; font-weight: 600; padding-bottom: 2px; margin-bottom: 2px; }
.terminal-info { color: #4fc1ff; }
.terminal-tool { color: #c586c0; }
.terminal-success { color: #4ec9b0; }
.terminal-error { color: #f14c4c; }
.terminal-warning { color: #dcdcaa; }
.terminal-muted { color: #6a9955; }
.terminal-divider { color: #3d3d3d; opacity: 0.8; }
.terminal-default { color: #d4d4d4; }

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}
.animate-blink { animation: blink 1s step-end infinite; }

.custom-scrollbar::-webkit-scrollbar { width: 8px; height: 8px; }
.custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
.custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(255, 255, 255, 0.15); border-radius: 4px; }
.custom-scrollbar::-webkit-scrollbar-thumb:hover { background: rgba(255, 255, 255, 0.25); }
.terminal-content:not(:hover)::-webkit-scrollbar-thumb { background: transparent; }
</style>
