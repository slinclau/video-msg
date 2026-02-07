<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

const props = defineProps<{
  stream: MediaStream | null
}>()

const videoEl = ref<HTMLVideoElement>()

function attachStream() {
  if (videoEl.value) {
    videoEl.value.srcObject = props.stream || null
  }
}

watch(() => props.stream, attachStream)
onMounted(attachStream)
</script>

<template>
  <div v-if="stream" class="fixed bottom-6 left-6 z-50">
    <div class="w-36 h-36 rounded-full overflow-hidden border-4 border-white shadow-2xl ring-2 ring-black/10">
      <video
        ref="videoEl"
        autoplay
        playsinline
        muted
        class="w-full h-full object-cover"
        style="transform: scaleX(-1)"
      />
    </div>
  </div>
</template>
