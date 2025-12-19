<script setup lang="ts">
import { ref } from 'vue'
import { useMediaRecorder } from '@/composables/useMediaRecorder'
import { useRecordingStore } from '@/stores/recording'

const emit = defineEmits<{
  recordingStopped: [blob: Blob]
}>()

const { isRecording, recordingTime, error, previewUrl, startRecording, stopRecording, formatTime } = useMediaRecorder()
const recordingStore = useRecordingStore()
const isStopping = ref(false)

async function handleStart() {
  try {
    await startRecording()
  } catch (err) {
    console.error('Failed to start recording:', err)
  }
}

async function handleStop() {
  if (isStopping.value) return // Prevent double-click

  isStopping.value = true
  try {
    const blob = await stopRecording()
    recordingStore.setCurrentRecording(blob)
    emit('recordingStopped', blob)
  } catch (err) {
    console.error('Failed to stop recording:', err)
  } finally {
    isStopping.value = false
  }
}
</script>

<template>
  <div class="max-w-4xl mx-auto p-6">
    <div v-if="error" class="bg-red-50 border border-red-200 text-red-700 px-6 py-4 rounded-xl mb-6">
      {{ error }}
    </div>

    <div class="flex flex-col items-center gap-6 mb-8">
      <button
        v-if="!isRecording"
        @click="handleStart"
        class="group relative px-12 py-5 bg-gradient-to-r from-emerald-500 to-green-600 text-white text-lg font-bold rounded-2xl shadow-xl hover:shadow-2xl transform hover:scale-105 transition-all duration-300"
      >
        <span class="flex items-center gap-3">
          <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
            <circle cx="12" cy="12" r="10" />
          </svg>
          Start Recording
        </span>
      </button>

      <div v-else class="flex flex-col items-center gap-6">
        <div class="flex items-center gap-3 bg-white px-8 py-4 rounded-2xl shadow-lg border border-red-200">
          <span class="relative flex h-4 w-4">
            <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-red-400 opacity-75"></span>
            <span class="relative inline-flex rounded-full h-4 w-4 bg-red-500"></span>
          </span>
          <span class="text-3xl font-bold font-mono text-gray-800 tabular-nums">{{ formatTime(recordingTime) }}</span>
        </div>
        <button
          @click="handleStop"
          :disabled="isStopping"
          class="px-12 py-5 bg-gradient-to-r from-red-500 to-rose-600 text-white text-lg font-bold rounded-2xl shadow-xl hover:shadow-2xl transform hover:scale-105 transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100"
        >
          <span class="flex items-center gap-3">
            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
              <rect x="6" y="6" width="12" height="12" rx="2" />
            </svg>
            {{ isStopping ? 'Stopping...' : 'Stop Recording' }}
          </span>
        </button>
      </div>
    </div>

    <div v-if="previewUrl && !isRecording" class="mt-8">
      <h3 class="text-2xl font-bold text-gray-800 mb-4">Preview</h3>
      <div class="bg-white rounded-2xl shadow-xl p-4 border border-gray-100">
        <video :src="previewUrl" controls class="w-full rounded-lg"></video>
      </div>
    </div>
  </div>
</template>
