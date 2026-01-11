<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import TianzigeGrid from './TianzigeGrid.vue'

interface Poem {
  id: number
  contents: string
  type: string
  author: string
  title: string
}

const poems = ref<Poem[]>([])
const currentPoemIndex = ref(0)
const currentCharIndex = ref(0)
const gridSize = ref(120)
const showGuidelines = ref(true)
const loading = ref(true)

const currentPoem = computed(() => poems.value[currentPoemIndex.value])

// 将诗句按行分割成二维数组，每行也存储起始索引
interface LineInfo {
  chars: string[]
  startIndex: number
}

const poemLines = computed((): LineInfo[] => {
  if (!currentPoem.value) return []
  const lines = currentPoem.value.contents.split('\n').filter(line => line.length > 0)
  let idx = 0
  return lines.map(line => {
    const result = { chars: line.split(''), startIndex: idx }
    idx += line.length
    return result
  })
})

const allChars = computed(() => poemLines.value.flatMap(line => line.chars))

const currentChar = computed(() => allChars.value[currentCharIndex.value] || '')

async function loadPoems() {
  try {
    const res = await fetch('/asset/300.json')
    poems.value = await res.json()
    loading.value = false
  } catch (e) {
    console.error('Failed to load poems:', e)
    loading.value = false
  }
}

function nextPoem() {
  currentPoemIndex.value = Math.floor(Math.random() * poems.value.length)
  currentCharIndex.value = 0
}

function setChar(lineIndex: number, charIdx: number) {
  const line = poemLines.value[lineIndex]
  if (line) {
    currentCharIndex.value = line.startIndex + charIdx
  }
}

function isCurrentChar(lineIndex: number, charIdx: number): boolean {
  const line = poemLines.value[lineIndex]
  if (!line) return false
  return line.startIndex + charIdx === currentCharIndex.value
}

onMounted(loadPoems)
</script>

<template>
  <div class="min-h-screen bg-gray-100 py-8 px-4">
    <h1 class="text-4xl font-bold text-center text-gray-800 mb-8">古诗练字</h1>

    <div v-if="loading" class="text-center text-gray-500">
      加载中...
    </div>

    <template v-else>
      <!-- 控制面板 -->
      <div class="flex justify-center gap-4 mb-8 flex-wrap">
        <button
          @click="nextPoem"
          class="bg-blue-500 hover:bg-blue-600 text-white py-2 px-6 rounded transition"
        >
          随机抽诗
        </button>

        <div class="flex items-center gap-4 bg-white px-4 py-2 rounded shadow">
          <span class="text-sm text-gray-500">格子大小:</span>
          <input
            type="range"
            v-model.number="gridSize"
            min="50"
            max="180"
            step="5"
            class="w-32"
          />
        </div>

        <button
          @click="showGuidelines = !showGuidelines"
          class="bg-gray-200 hover:bg-gray-300 text-gray-700 py-2 px-4 rounded transition"
        >
          {{ showGuidelines ? '隐藏辅助线' : '显示辅助线' }}
        </button>
      </div>

      <!-- 诗歌信息 -->
      <div class="text-center mb-8">
        <h2 class="text-2xl font-bold text-gray-800">{{ currentPoem.title }}</h2>
        <p class="text-gray-500">{{ currentPoem.author }} · {{ currentPoem.type }}</p>
      </div>

      <!-- 田字格矩阵 -->
      <div class="flex flex-col items-center gap-4 mb-8">
        <div
          v-for="(line, lineIndex) in poemLines"
          :key="lineIndex"
          class="flex gap-2"
        >
          <TianzigeGrid
            v-for="(char, charIdx) in line.chars"
            :key="charIdx"
            :character="char"
            :size="gridSize"
            :show-guidelines="showGuidelines"
            class="cursor-pointer transition-all"
            :class="isCurrentChar(lineIndex, charIdx) ? 'ring-2 ring-blue-500 ring-offset-2' : ''"
            @click="setChar(lineIndex, charIdx)"
          />
        </div>
      </div>

      <!-- 当前字提示 -->
      <div class="text-center mb-8">
        <p class="text-sm text-gray-400">
          当前: {{ currentChar }} ({{ currentCharIndex + 1 }} / {{ allChars.length }})
        </p>
      </div>
    </template>
  </div>
</template>
