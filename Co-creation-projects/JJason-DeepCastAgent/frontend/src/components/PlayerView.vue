<template>
  <div class="hero min-h-screen">
    <div class="hero-content flex-col lg:flex-row-reverse gap-8 w-full max-w-6xl items-start">
      <!-- Right: Report -->
      <div class="card glass-panel shadow-2xl flex-1 h-[70vh] w-full lg:w-3/5 overflow-hidden rounded-2xl border border-white/10">
        <div class="card-body p-0 flex flex-col h-full bg-black/20">
          <div class="p-6 border-b border-white/10 sticky top-0 z-10 bg-black/40 backdrop-blur-md">
            <div class="flex items-center justify-between">
              <h2 class="card-title text-white">📄 研究报告</h2>
              <button class="btn btn-xs btn-ghost text-white/50" @click="$emit('downloadReport')">下载</button>
            </div>
          </div>
          <div class="overflow-y-auto p-8 custom-scrollbar flex-1 text-gray-200">
            <article class="prose prose-sm prose-invert max-w-none" v-html="renderedReport"></article>
          </div>
        </div>
      </div>

      <!-- Left: Player -->
      <div class="card glass-panel shadow-2xl flex-shrink-0 w-full lg:w-2/5 text-center h-auto rounded-2xl border border-white/10">
        <figure class="px-10 pt-12 pb-4">
          <div class="avatar placeholder">
            <div class="bg-black/40 text-white rounded-full w-48 h-48 ring-4 ring-white/10 shadow-[0_0_50px_rgba(0,0,0,0.5)] flex items-center justify-center relative overflow-hidden backdrop-blur-md">
              <!-- Vinyl Animation -->
              <div class="absolute inset-0 border-[2px] border-white/5 rounded-full" style="margin: 2px"></div>
              <div class="absolute inset-0 border-[2px] border-white/5 rounded-full" style="margin: 10px"></div>
              <div class="absolute inset-0 border-[2px] border-white/5 rounded-full" style="margin: 20px"></div>
              <div class="absolute inset-0 border-[10px] border-black/60 rounded-full opacity-40" :class="{ 'animate-spin': isPlaying }" style="animation-duration: 4s;"></div>
              <!-- Center Label -->
              <div class="z-10 w-16 h-16 rounded-full bg-gradient-to-tr from-blue-500 to-purple-500 shadow-inner flex items-center justify-center">
                <span class="text-xl font-bold text-white">DC</span>
              </div>
            </div>
          </div>
        </figure>
        <div class="card-body items-center text-center pt-2">
          <h2 class="card-title text-2xl text-white font-bold drop-shadow-md">{{ topic }}</h2>
          <p class="text-blue-200/60 text-sm font-medium tracking-widest uppercase mb-6">DeepCast Original</p>

          <div class="w-full bg-black/30 rounded-xl p-4 border border-white/5 shadow-inner">
            <audio
              ref="audioPlayer"
              :src="audioUrl"
              controls
              class="w-full"
              @play="isPlaying = true"
              @pause="isPlaying = false"
            ></audio>
          </div>

          <div class="card-actions mt-8 w-full gap-3 flex-col">
            <a :href="audioUrl" download class="btn macos-btn-primary w-full border-0 rounded-xl text-lg h-12">
              ⬇️ 下载 MP3
            </a>
            <button class="btn btn-ghost text-white/50 hover:text-white w-full" @click="$emit('reset')">
              🪄 制作新播客
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";
import MarkdownIt from "markdown-it";

const md = new MarkdownIt();

const props = defineProps<{
  topic: string;
  audioUrl: string;
  reportMarkdown: string;
}>();

defineEmits<{
  reset: [];
  downloadReport: [];
}>();

const isPlaying = ref(false);
const audioPlayer = ref<HTMLAudioElement | null>(null);

const renderedReport = computed(() => md.render(props.reportMarkdown));
</script>

<style scoped>
.glass-panel {
  background: rgba(30, 30, 30, 0.7);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
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

.custom-scrollbar::-webkit-scrollbar { width: 8px; height: 8px; }
.custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
.custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(255, 255, 255, 0.15); border-radius: 4px; }
.custom-scrollbar::-webkit-scrollbar-thumb:hover { background: rgba(255, 255, 255, 0.25); }
</style>
