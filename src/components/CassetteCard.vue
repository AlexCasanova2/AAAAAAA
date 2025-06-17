<template>
  <div class="cassette-card" :class="{ 'is-minimized': isMinimized }" ref="cassetteRef">
    <div class="cassette-body">
      <div class="cassette-wheels">
        <div class="wheel left"></div>
        <div class="wheel right"></div>
      </div>
      <div class="cassette-details">
        <div class="cassette-label">{{ title }}</div>
        <div class="cassette-time">{{ duration }}s</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import gsap from 'gsap'

const props = defineProps({
  title: {
    type: String,
    default: 'Nueva grabación'
  },
  duration: {
    type: Number,
    default: 0
  },
  isMinimized: {
    type: Boolean,
    default: false
  }
})

const cassetteRef = ref(null)

const animateToCorner = async (targetElement) => {
  if (!cassetteRef.value || !targetElement) return

  const timeline = gsap.timeline()
  
  timeline
    .to(cassetteRef.value, {
      scale: 0.5,
      duration: 0.5,
      ease: 'power2.inOut'
    })
    .to(cassetteRef.value, {
      x: targetElement.offsetLeft - cassetteRef.value.offsetLeft,
      y: targetElement.offsetTop - cassetteRef.value.offsetTop,
      duration: 0.8,
      ease: 'power2.inOut'
    })
    .to(cassetteRef.value, {
      scale: 0,
      opacity: 0,
      duration: 0.3,
      ease: 'power2.in'
    })
}

defineExpose({
  animateToCorner
})
</script>

<style scoped>
.cassette-card {
  background: #2c3e50;
  border-radius: 8px;
  padding: 16px;
  width: 200px;
  height: 120px;
  position: relative;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.cassette-body {
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.cassette-wheels {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
}

.wheel {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #34495e;
  border: 4px solid #2c3e50;
  position: relative;
}

.wheel::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #2c3e50;
}

.cassette-details {
  text-align: center;
  color: #ecf0f1;
}

.cassette-label {
  font-size: 14px;
  margin-bottom: 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.cassette-time {
  font-size: 12px;
  color: #95a5a6;
}

.is-minimized {
  transform: scale(0.5);
}
</style> 