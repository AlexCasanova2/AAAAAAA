<template>
  <div 
    class="fixed inset-0 pointer-events-none"
    :class="{ 'z-50': isTransitioning }"
  >
    <div 
      v-show="isTransitioning"
      class="absolute rounded-full bg-[#111] transition-circle"
      :style="positionStyle"
      @animationend="onAnimationEnd"
      @animationprogress="onAnimationProgress"
    ></div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  isTransitioning: Boolean,
  originX: {
    type: Number,
    default: 0
  },
  originY: {
    type: Number,
    default: 0
  }
})

const emit = defineEmits(['animationEnd', 'animationProgress'])

const positionStyle = computed(() => ({
  left: `${props.originX}px`,
  top: `${props.originY}px`,
  transform: 'translate(-50%, -50%)'
}))

const onAnimationEnd = () => {
  emit('animationEnd')
}

const onAnimationProgress = (event) => {
  emit('animationProgress', event)
}
</script>

<style scoped>
.transition-circle {
  width: 24px;
  height: 24px;
  animation: expand 800ms cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

@keyframes expand {
  0% {
    width: 24px;
    height: 24px;
  }
  100% {
    width: 800vw;
    height: 800vw;
  }
}
</style> 