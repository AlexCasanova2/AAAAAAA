<template>
  <div 
    class="fixed top-0 left-0 right-0 z-10 w-full p-4 flex justify-between items-center"
    :class="backgroundColorClass"
    :style="{ paddingTop: `calc(1rem + env(safe-area-inset-top))` }"
  >
    <slot name="left-content"></slot>
    <h1 class="text-xl font-medium" :class="textColorClass">
      {{ title }}
    </h1>
    <slot name="right-content"></slot>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  title: {
    type: String,
    required: true
  },
  darkMode: {
    type: Boolean,
    default: false
  }
})

const backgroundColorClass = computed(() => (
  props.darkMode ? 'bg-[var(--color-dark-background)]' : 'bg-[#fefefe]'
))

const textColorClass = computed(() => (
  props.darkMode ? 'text-white' : 'text-[var(--color-dark-background)]'
))
</script>

<style scoped>
.header-right-content {
  opacity: 1 !important;
  visibility: visible !important;
}
</style> 