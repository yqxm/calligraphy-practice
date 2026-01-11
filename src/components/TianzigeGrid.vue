<script setup lang="ts">
interface Props {
  character: string
  size?: number
  showGuidelines?: boolean
}

withDefaults(defineProps<Props>(), {
  size: 200,
  showGuidelines: true,
})
</script>

<template>
  <div
    class="relative bg-white rounded-lg shrink-0"
    :style="{ width: `${size}px`, height: `${size}px` }"
    style="border: 2px solid #f87171;"
  >
    <div v-if="showGuidelines" class="absolute inset-0">
      <div class="absolute left-1/2 top-0 bottom-0 -translate-x-1/2" style="border-left: none; width: 2px; height: 100%; background: repeating-linear-gradient(0deg, #fca5a5 0, #fca5a5 12px, transparent 12px, transparent 20px);"></div>
      <div class="absolute top-1/2 left-0 right-0 -translate-y-1/2" style="border-top: none; height: 2px; background: repeating-linear-gradient(90deg, #fca5a5 0, #fca5a5 12px, transparent 12px, transparent 20px);"></div>
    </div>
    <div
      class="absolute inset-0 flex items-center justify-center text-gray-800 select-none"
      :style="{ fontSize: `${size * 0.6}px`, fontFamily: 'LXGWWenKai, serif' }"
    >
      {{ character }}
    </div>
  </div>
</template>
