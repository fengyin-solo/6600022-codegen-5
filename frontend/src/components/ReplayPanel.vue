<template>
  <div class="bg-gray-900 rounded-xl p-4 border border-gray-700">
    <h3 class="text-lg font-bold text-green-400 mb-3">棋谱回放</h3>

    <div v-if="store.status !== 'replaying'">
      <p v-if="store.gameRecords.length === 0" class="text-gray-500 text-sm">暂无棋谱记录</p>
      <div v-else class="space-y-2 max-h-64 overflow-y-auto">
        <div
          v-for="record in store.gameRecords"
          :key="record.id"
          class="bg-gray-800 rounded-lg p-3 cursor-pointer hover:bg-gray-700 transition-colors border border-gray-700"
          @click="store.startReplay(record)"
        >
          <div class="flex justify-between items-center">
            <span class="text-sm text-gray-300">{{ record.createdAt }}</span>
            <span
              class="text-xs px-2 py-0.5 rounded-full"
              :class="record.winner === 1 ? 'bg-gray-700 text-gray-200' : record.winner === 2 ? 'bg-white text-black' : 'bg-yellow-600 text-white'"
            >
              {{ record.winner === 1 ? '黑棋胜' : record.winner === 2 ? '白棋胜' : '平局' }}
            </span>
          </div>
          <div class="flex justify-between items-center mt-1">
            <span class="text-xs text-gray-500">共 {{ record.moves.length }} 手</span>
            <span v-if="record.annotations && record.annotations.length > 0" class="text-xs text-yellow-400 flex items-center gap-1">
              <svg class="w-3 h-3" fill="currentColor" viewBox="0 0 24 24"><path d="M21.99 4c0-1.1-.89-2-1.99-2H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h14l4 4-.01-18zM18 14H6v-2h12v2zm0-3H6V9h12v2zm0-3H6V6h12v2z"/></svg>
              {{ record.annotations.length }} 条批注
            </span>
          </div>
        </div>
      </div>
    </div>

    <div v-else>
      <div class="text-center mb-3">
        <span class="text-gray-400 text-sm">第 {{ store.replayIndex }} / {{ store.replayMoves.length }} 手</span>
      </div>

      <div class="w-full bg-gray-800 rounded-full h-2 mb-4">
        <div
          class="bg-green-500 h-2 rounded-full transition-all"
          :style="{ width: `${(store.replayIndex / store.replayMoves.length) * 100}%` }"
        />
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <button @click="store.replayGoToStart()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="回到开始">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7"/></svg>
        </button>
        <button @click="store.replayStepBack()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="上一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <button @click="store.toggleReplayPlay()" class="p-3 bg-green-600 rounded-lg hover:bg-green-500 text-white transition-colors" :title="store.isReplayPlaying ? '暂停' : '播放'">
          <svg v-if="!store.isReplayPlaying" class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
          <svg v-else class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M6 4h4v16H6zM14 4h4v16h-4z"/></svg>
        </button>
        <button @click="store.replayStepForward()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="下一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
        <button @click="store.replayGoToEnd()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="跳到结尾">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 5l7 7-7 7M5 5l7 7-7 7"/></svg>
        </button>
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <span class="text-xs text-gray-500">速度:</span>
        <button
          v-for="speed in speeds"
          :key="speed.value"
          @click="store.setReplaySpeed(speed.value)"
          class="px-2 py-1 text-xs rounded transition-colors"
          :class="store.replaySpeed === speed.value ? 'bg-green-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          {{ speed.label }}
        </button>
      </div>

      <div class="border-t border-gray-700 pt-4 mb-4">
        <div class="flex items-center justify-between mb-2">
          <h4 class="text-sm font-bold text-yellow-400 flex items-center gap-2">
            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24"><path d="M21.99 4c0-1.1-.89-2-1.99-2H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h14l4 4-.01-18zM18 14H6v-2h12v2zm0-3H6V9h12v2zm0-3H6V6h12v2z"/></svg>
            复盘批注
          </h4>
          <span class="text-xs text-gray-500">
            {{ store.replayAnnotations.length }} 条批注
          </span>
        </div>

        <div v-if="store.replayIndex === 0" class="text-gray-500 text-xs text-center py-4">
          请先走到第 1 手再添加批注
        </div>

        <div v-else>
          <div v-if="store.currentReplayAnnotation && !isEditing" class="bg-gray-800 rounded-lg p-3 mb-3">
            <div class="text-xs text-gray-400 mb-1">
              第 {{ store.currentReplayAnnotation.moveIndex }} 手批注
            </div>
            <p class="text-gray-200 text-sm whitespace-pre-wrap">{{ store.currentReplayAnnotation.content }}</p>
            <div class="flex gap-2 mt-3">
              <button
                @click="startEditing"
                class="flex-1 py-1.5 bg-blue-600/20 border border-blue-600/50 text-blue-400 rounded text-xs hover:bg-blue-600/30 transition-colors"
              >
                编辑
              </button>
              <button
                @click="handleDelete"
                class="flex-1 py-1.5 bg-red-600/20 border border-red-600/50 text-red-400 rounded text-xs hover:bg-red-600/30 transition-colors"
              >
                删除
              </button>
            </div>
          </div>

          <div v-else-if="!store.currentReplayAnnotation && !isEditing" class="text-center py-3">
            <button
              @click="startEditing"
              class="w-full py-2 bg-yellow-600/20 border border-yellow-600/50 text-yellow-400 rounded-lg text-sm hover:bg-yellow-600/30 transition-colors"
            >
              + 添加批注
            </button>
          </div>

          <div v-if="isEditing" class="space-y-2">
            <textarea
              v-model="editContent"
              class="w-full h-24 bg-gray-800 border border-gray-600 rounded-lg p-2 text-gray-200 text-sm resize-none focus:outline-none focus:border-yellow-500"
              placeholder="输入你的复盘批注..."
              autofocus
            />
            <div class="flex gap-2">
              <button
                @click="handleSave"
                class="flex-1 py-1.5 bg-green-600 text-white rounded text-xs hover:bg-green-500 transition-colors"
              >
                保存
              </button>
              <button
                @click="cancelEditing"
                class="flex-1 py-1.5 bg-gray-700 text-gray-300 rounded text-xs hover:bg-gray-600 transition-colors"
              >
                取消
              </button>
            </div>
          </div>
        </div>

        <div v-if="store.replayAnnotations.length > 0" class="mt-4">
          <div class="text-xs text-gray-500 mb-2">批注列表</div>
          <div class="space-y-1 max-h-32 overflow-y-auto">
            <div
              v-for="annotation in sortedAnnotations"
              :key="annotation.id"
              class="bg-gray-800/50 rounded p-2 cursor-pointer hover:bg-gray-800 transition-colors text-xs"
              @click="jumpToAnnotation(annotation)"
            >
              <div class="flex justify-between items-center">
                <span class="text-yellow-400 font-medium">第 {{ annotation.moveIndex }} 手</span>
                <span class="text-gray-500 text-xs">{{ formatTime(annotation.updatedAt) }}</span>
              </div>
              <p class="text-gray-400 truncate mt-0.5">{{ annotation.content }}</p>
            </div>
          </div>
        </div>
      </div>

      <button
        @click="store.stopReplay()"
        class="w-full py-2 bg-red-600/20 border border-red-600/50 text-red-400 rounded-lg hover:bg-red-600/30 transition-colors text-sm"
      >
        退出回放
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { useGameStore } from '../store/game';

const store = useGameStore();

const speeds = [
  { label: '慢', value: 2000 },
  { label: '中', value: 1000 },
  { label: '快', value: 500 },
  { label: '极快', value: 200 },
];

const isEditing = ref(false);
const editContent = ref('');

const sortedAnnotations = computed(() => {
  return [...store.replayAnnotations].sort((a, b) => a.moveIndex - b.moveIndex);
});

watch(() => store.replayIndex, () => {
  isEditing.value = false;
  editContent.value = '';
});

function startEditing() {
  isEditing.value = true;
  editContent.value = store.currentReplayAnnotation?.content || '';
}

function cancelEditing() {
  isEditing.value = false;
  editContent.value = '';
}

function handleSave() {
  if (!editContent.value.trim()) {
    if (store.currentReplayAnnotation) {
      store.deleteAnnotation(store.currentReplayAnnotation.id);
    }
    cancelEditing();
    return;
  }
  store.addAnnotation(editContent.value.trim());
  cancelEditing();
}

function handleDelete() {
  if (store.currentReplayAnnotation) {
    if (confirm('确定要删除这条批注吗？')) {
      store.deleteAnnotation(store.currentReplayAnnotation.id);
    }
  }
}

function jumpToAnnotation(annotation: { moveIndex: number }) {
  const targetIndex = annotation.moveIndex;
  if (targetIndex > store.replayIndex) {
    while (store.replayIndex < targetIndex) {
      store.replayStepForward();
    }
  } else if (targetIndex < store.replayIndex) {
    while (store.replayIndex > targetIndex) {
      store.replayStepBack();
    }
  }
}

function formatTime(timestamp: number): string {
  const date = new Date(timestamp);
  return date.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' });
}
</script>
