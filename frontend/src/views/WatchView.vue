<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import VideoPlayer from '@/components/VideoPlayer.vue'
import { apiService } from '@/services/api'
import { ProcessingStatus, type RecordingDetailResponse } from '@/types/recording'

const route = useRoute()
const router = useRouter()

const uuid = ref(route.params.uuid as string)
const recording = ref<RecordingDetailResponse | null>(null)
const loading = ref(true)
const error = ref<string | null>(null)
const isProcessing = ref(false)
let pollingInterval: number | null = null

async function loadRecording() {
  try {
    loading.value = true
    recording.value = await apiService.getRecordingMetadata(uuid.value)

    // Check if still processing
    if (recording.value.processingStatus === ProcessingStatus.PROCESSING) {
      isProcessing.value = true
      startPolling()
    } else {
      isProcessing.value = false
      stopPolling()
    }

    error.value = null
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Failed to load recording'
    stopPolling()
  } finally {
    loading.value = false
  }
}

function startPolling() {
  // Poll every 3 seconds
  if (!pollingInterval) {
    pollingInterval = window.setInterval(async () => {
      try {
        const updatedRecording = await apiService.getRecordingMetadata(uuid.value)
        recording.value = updatedRecording

        if (updatedRecording.processingStatus !== ProcessingStatus.PROCESSING) {
          isProcessing.value = false
          stopPolling()
        }
      } catch (err) {
        console.error('Polling error:', err)
      }
    }, 3000)
  }
}

function stopPolling() {
  if (pollingInterval) {
    clearInterval(pollingInterval)
    pollingInterval = null
  }
}

onMounted(() => {
  loadRecording()
})

onUnmounted(() => {
  stopPolling()
})

function goToRecord() {
  router.push('/record')
}

function formatFileSize(bytes: number): string {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

function formatDate(dateString: string): string {
  const date = new Date(dateString)
  return date.toLocaleString()
}
</script>

<template>
  <div class="max-w-6xl mx-auto px-6 py-12">
    <h1 class="text-5xl font-bold text-center bg-gradient-to-r from-indigo-600 to-purple-600 bg-clip-text text-transparent mb-12">
      Watch Recording
    </h1>

    <div v-if="loading" class="text-center py-24">
      <div class="inline-block w-16 h-16 border-4 border-indigo-200 border-t-indigo-600 rounded-full animate-spin mb-6"></div>
      <p class="text-xl text-gray-600 font-medium">Loading recording...</p>
    </div>

    <div v-else-if="error" class="text-center py-16 bg-white rounded-2xl shadow-xl p-12 border border-red-100">
      <div class="inline-flex items-center justify-center w-20 h-20 bg-red-100 rounded-full mb-6">
        <svg class="w-10 h-10 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
        </svg>
      </div>
      <p class="text-red-600 text-xl font-semibold mb-6">{{ error }}</p>
      <button
        @click="goToRecord"
        class="px-8 py-3 bg-gradient-to-r from-emerald-500 to-green-600 text-white font-semibold rounded-lg hover:shadow-lg transform hover:scale-105 transition-all duration-200"
      >
        Back to Recorder
      </button>
    </div>

    <div v-else-if="recording" class="flex flex-col gap-8">
      <!-- Processing Status Message -->
      <div v-if="recording.processingStatus === 'PROCESSING'" class="text-center py-16 bg-gradient-to-br from-indigo-500 via-purple-500 to-pink-500 rounded-2xl shadow-2xl text-white">
        <div class="inline-block w-16 h-16 border-4 border-white/30 border-t-white rounded-full animate-spin mb-8"></div>
        <h2 class="text-3xl font-bold mb-4">Processing Video...</h2>
        <p class="text-lg mb-2 opacity-95">Your video is being re-encoded to ensure optimal playback. This may take a minute.</p>
        <p class="text-sm opacity-80 italic">The page will update automatically when processing is complete.</p>
      </div>

      <!-- Failed Status Message -->
      <div v-else-if="recording.processingStatus === 'FAILED'" class="text-center py-16 bg-white rounded-2xl shadow-xl p-12 border border-red-100">
        <div class="inline-flex items-center justify-center w-20 h-20 bg-red-100 rounded-full mb-6">
          <svg class="w-10 h-10 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"></path>
          </svg>
        </div>
        <h2 class="text-3xl font-bold text-gray-800 mb-4">Processing Failed</h2>
        <p class="text-red-600 text-lg mb-6">{{ recording.processingError || 'An error occurred while processing your video.' }}</p>
        <button
          @click="goToRecord"
          class="px-8 py-3 bg-gradient-to-r from-orange-500 to-red-500 text-white font-semibold rounded-lg hover:shadow-lg transform hover:scale-105 transition-all duration-200"
        >
          Try Recording Again
        </button>
      </div>

      <!-- Video Player (only show when READY) -->
      <VideoPlayer v-else :uuid="uuid" />

      <div class="bg-white rounded-2xl shadow-xl p-8 border border-gray-100">
        <h2 class="text-2xl font-bold text-gray-800 mb-6">Recording Details</h2>
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <div class="flex flex-col gap-1">
            <span class="text-sm font-semibold text-gray-500 uppercase tracking-wide">Filename</span>
            <span class="text-base text-gray-800 font-medium">{{ recording.filename }}</span>
          </div>
          <div class="flex flex-col gap-1">
            <span class="text-sm font-semibold text-gray-500 uppercase tracking-wide">Size</span>
            <span class="text-base text-gray-800 font-medium">{{ formatFileSize(recording.fileSize) }}</span>
          </div>
          <div class="flex flex-col gap-1">
            <span class="text-sm font-semibold text-gray-500 uppercase tracking-wide">Type</span>
            <span class="text-base text-gray-800 font-medium">{{ recording.contentType }}</span>
          </div>
          <div class="flex flex-col gap-1">
            <span class="text-sm font-semibold text-gray-500 uppercase tracking-wide">Status</span>
            <span
              class="text-base font-bold"
              :class="{
                'text-amber-600': recording.processingStatus === 'PROCESSING',
                'text-emerald-600': recording.processingStatus === 'READY',
                'text-red-600': recording.processingStatus === 'FAILED'
              }"
            >
              {{ recording.processingStatus }}
            </span>
          </div>
          <div class="flex flex-col gap-1">
            <span class="text-sm font-semibold text-gray-500 uppercase tracking-wide">Created</span>
            <span class="text-base text-gray-800 font-medium">{{ formatDate(recording.createdAt) }}</span>
          </div>
          <div class="flex flex-col gap-1">
            <span class="text-sm font-semibold text-gray-500 uppercase tracking-wide">UUID</span>
            <span class="text-sm text-gray-800 font-mono break-all">{{ recording.uuid }}</span>
          </div>
        </div>
      </div>

      <div class="text-center">
        <button
          @click="goToRecord"
          class="px-8 py-4 bg-gradient-to-r from-emerald-500 to-green-600 text-white font-semibold rounded-xl shadow-lg hover:shadow-xl transform hover:scale-105 transition-all duration-200"
        >
          Create New Recording
        </button>
      </div>
    </div>
  </div>
</template>
