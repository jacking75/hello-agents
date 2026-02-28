<template>
  <div class="min-h-screen p-6">
    <div class="max-w-7xl mx-auto">
      <!-- Navbar / Header -->
      <div class="nav-glass rounded-xl shadow-lg mb-6 px-6 py-4">
        <div class="flex items-center justify-between gap-4">
          <div class="flex items-center gap-3">
            <span class="text-3xl filter drop-shadow-md">🎙️</span>
            <span class="text-2xl font-bold text-white tracking-tight">DeepCast</span>
          </div>
          <div class="flex items-center gap-3">
            <button v-if="reportReady" class="btn btn-ghost btn-sm text-blue-300 hover:bg-white/5" @click="$emit('downloadReport')">
              📄 下载研究报告
            </button>
            <button v-if="!podcastReady" class="btn btn-ghost btn-sm text-red-400 hover:bg-white/5" @click="$emit('cancel')">
              取消制作
            </button>
          </div>
        </div>
      </div>

      <!-- Main Content -->
      <div class="grid grid-cols-1 lg:grid-cols-4 gap-6">

        <!-- Left Column: Progress Steps -->
        <div class="lg:col-span-1">
          <div class="card glass-panel h-[500px] rounded-xl">
            <div class="card-body p-6 relative overflow-hidden">
              <!-- Decorative element -->
              <div class="absolute top-0 right-0 -mr-8 -mt-8 w-40 h-40 bg-blue-500/20 rounded-full blur-3xl"></div>
              <div class="absolute bottom-0 left-0 -ml-8 -mb-8 w-40 h-40 bg-purple-500/20 rounded-full blur-3xl"></div>

              <h2 class="text-xl font-bold text-white mb-8 flex items-center justify-center gap-3 z-10">
                <div class="p-2 bg-white/10 rounded-lg backdrop-blur-md shadow-inner border border-white/5">
                  <span v-if="productionStage === 'done'" class="text-2xl">✅</span>
                  <span v-else class="text-3xl animate-spin-slow inline-block">🔄</span>
                </div>
                <span class="tracking-wide">制作流程</span>
              </h2>

              <div class="flex-1 w-full flex justify-center pl-4">
                <ul class="steps steps-vertical font-medium w-full h-full justify-evenly">
                  <li class="step gap-3" :class="getStepClass('research')">
                    <div class="flex flex-col text-left py-2 min-w-[120px]">
                      <div class="flex items-center gap-2">
                        <span class="text-xl filter drop-shadow" :class="{ 'animate-bounce': productionStage === 'research' }">🔍</span>
                        <span class="font-bold text-gray-200">深度研究</span>
                      </div>
                      <span class="text-xs text-gray-400 font-normal ml-8 mt-1">网络搜索 & 信息聚合</span>
                    </div>
                  </li>
                  <li class="step gap-3" :class="getStepClass('script')">
                    <div class="flex flex-col text-left py-2 min-w-[120px]">
                      <div class="flex items-center gap-2">
                        <span class="text-xl filter drop-shadow" :class="{ 'animate-bounce': productionStage === 'script' }">✍️</span>
                        <span class="font-bold text-gray-200">剧本创作</span>
                      </div>
                      <span class="text-xs text-gray-400 font-normal ml-8 mt-1">生成对话 & 角色分配</span>
                    </div>
                  </li>
                  <li class="step gap-3" :class="getStepClass('audio')">
                    <div class="flex flex-col text-left py-2 min-w-[120px]">
                      <div class="flex items-center gap-2">
                        <span class="text-xl filter drop-shadow" :class="{ 'animate-bounce': productionStage === 'audio' }">🎵</span>
                        <span class="font-bold text-gray-200">音频合成</span>
                      </div>
                      <span class="text-xs text-gray-400 font-normal ml-8 mt-1">TTS 语音生成 & 拼接</span>
                    </div>
                  </li>
                  <li class="step gap-3" :class="{ 'step-primary': podcastReady || productionStage === 'done' }">
                    <div class="flex flex-col text-left py-2 min-w-[120px]">
                      <div class="flex items-center gap-2">
                        <span class="text-xl filter drop-shadow" :class="{ 'animate-pulse': podcastReady }">🎉</span>
                        <span class="font-bold text-gray-200">完成</span>
                      </div>
                      <span class="text-xs text-gray-400 font-normal ml-8 mt-1">播放 & 下载播客</span>
                    </div>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>

        <!-- Right Column: Logs & Output -->
        <div class="lg:col-span-3 flex flex-col gap-4">

          <!-- macOS Style Terminal -->
          <TerminalLog ref="terminalRef" :logs="logs" :is-waiting="isWaiting" :waiting-dots="waitingDots" />

          <!-- Result Actions -->
          <div v-if="podcastReady" class="flex gap-4">
            <a :href="audioUrl" download class="btn macos-btn-primary flex-1 btn-lg text-lg rounded-xl border-0">
              ⬇️ 下载 MP3
            </a>
            <button class="btn glass text-white flex-1 btn-lg text-lg rounded-xl" @click="$emit('goPlayer')">
              🎧 进入播放器
            </button>
          </div>

          <!-- Inline Player -->
          <div v-if="podcastReady" class="card glass-panel rounded-xl mt-2">
            <div class="card-body p-4">
              <div class="flex items-center gap-3 mb-2">
                <span class="text-xl">🎧</span>
                <h3 class="text-sm font-bold text-gray-200">快速试听</h3>
              </div>
              <audio class="w-full opacity-90 hover:opacity-100 transition-opacity" :src="audioUrl" controls></audio>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import TerminalLog from "./TerminalLog.vue";
import type { LogEntry } from "./TerminalLog.vue";

export type ProductionStage = "research" | "script" | "audio" | "done";

const props = defineProps<{
  logs: LogEntry[];
  isWaiting: boolean;
  waitingDots: string;
  productionStage: ProductionStage;
  reportReady: boolean;
  podcastReady: boolean;
  audioUrl: string;
}>();

defineEmits<{
  cancel: [];
  downloadReport: [];
  goPlayer: [];
}>();

const terminalRef = ref<InstanceType<typeof TerminalLog> | null>(null);

function scrollTerminal() {
  terminalRef.value?.scrollToBottom();
}

defineExpose({ scrollTerminal });

function getStepClass(step: ProductionStage) {
  const stepsOrder: ProductionStage[] = ["research", "script", "audio", "done"];
  const currentIdx = stepsOrder.indexOf(props.productionStage);
  const stepIdx = stepsOrder.indexOf(step);

  if (currentIdx > stepIdx) return "step-primary";
  if (currentIdx === stepIdx) return "step-primary font-bold";
  return "";
}
</script>

<style scoped>
.glass-panel {
  background: rgba(30, 30, 30, 0.7);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
}

.nav-glass {
  background: rgba(40, 40, 40, 0.85);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.macos-btn-primary {
  background: linear-gradient(180deg, #0A84FF 0%, #007AFF 100%);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 1px 2px rgba(0,0,0,0.2), inset 0 1px 1px rgba(255,255,255,0.2);
  transition: all 0.2s;
}
.macos-btn-primary:hover {
  filter: brightness(1.05);
  transform: translateY(-0.5px);
  box-shadow: 0 4px 12px rgba(0, 122, 255, 0.3), inset 0 1px 1px rgba(255,255,255,0.2);
}
.macos-btn-primary:active { transform: translateY(0.5px); filter: brightness(0.95); }
.macos-btn-primary:disabled { opacity: 0.5; filter: grayscale(0.5); transform: none; cursor: not-allowed; }

@keyframes spin-slow {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
.animate-spin-slow { animation: spin-slow 3s linear infinite; }
</style>
