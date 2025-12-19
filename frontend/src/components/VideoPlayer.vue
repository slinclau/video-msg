<script setup lang="ts">
import { computed } from 'vue'
import { apiService } from '@/services/api'

const props = defineProps<{
  uuid: string
}>()

const streamUrl = computed(() => apiService.getStreamUrl(props.uuid))
</script>

<template>
  <div class="w-full max-w-6xl mx-auto">
    <div class="bg-white rounded-2xl shadow-2xl p-4 border border-gray-100">
      <video
        :src="streamUrl"
        controls
        class="w-full h-auto rounded-xl bg-black shadow-lg"
      >
        Your browser does not support the video tag.
      </video>
    </div>
  </div>
</template>
