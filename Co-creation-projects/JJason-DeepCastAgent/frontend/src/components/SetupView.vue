<template>
  <div class="min-h-screen flex items-center justify-center p-6">
    <div class="w-full max-w-xl">
      <div class="text-center mb-12">
        <div class="text-6xl mb-6">🎙️</div>
        <h1 class="text-6xl font-bold mb-4 text-transparent bg-clip-text bg-gradient-to-r from-blue-400 via-indigo-400 to-purple-500">DeepCast</h1>
        <p class="text-xl text-gray-400">进行深度研究并转化为引人入胜的播客</p>
      </div>

      <div class="card glass-panel rounded-2xl">
        <form @submit.prevent="$emit('start', topic)" class="card-body p-8">
          <div class="form-control mb-6">
            <textarea
              v-model="topic"
              class="w-full textarea textarea-bordered h-32 text-lg leading-relaxed resize-none macos-input rounded-xl"
              placeholder="💡 请输入播客主题（例如：AI Agent 的发展趋势）"
              required
              @keydown.enter.prevent="$emit('start', topic)"
            ></textarea>
          </div>

          <div class="alert bg-blue-500/10 border border-blue-500/20 mb-8 rounded-xl">
            <span class="text-sm text-blue-300 font-medium">🔍 使用混合搜索引擎 (Tavily + SerpApi)</span>
          </div>

          <button
            class="btn btn-lg w-full font-semibold rounded-xl macos-btn-primary border-0"
            :disabled="!topic.trim()"
          >
            ✨ 开始制作播客
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";

const topic = defineModel<string>("topic", { required: true });

defineEmits<{
  start: [topic: string];
}>();
</script>

<style scoped>
.glass-panel {
  background: rgba(30, 30, 30, 0.7);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
}

.macos-input {
  background: rgba(0, 0, 0, 0.2) !important;
  border: 1px solid rgba(255, 255, 255, 0.1) !important;
  color: #fff !important;
  transition: all 0.3s ease;
}
.macos-input:focus {
  background: rgba(0, 0, 0, 0.4) !important;
  border-color: #0A84FF !important;
  box-shadow: 0 0 0 3px rgba(10, 132, 255, 0.2);
  outline: none;
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
</style>
