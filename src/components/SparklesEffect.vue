<template>
  <div class="seal-effect-container" v-if="show">
    <!-- Flash central -->
    <div class="flash"></div>
    
    <!-- Chispas eléctricas -->
    <div v-for="n in 6" :key="n" class="spark" :style="getSparkStyle(n)"></div>
    
    <!-- Partículas tipo estrella -->
    <div v-for="n in 8" :key="'star-'+n" class="star" :style="getStarStyle(n)"></div>
    
    <!-- Efecto de impacto -->
    <div class="impact-ring"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  show: {
    type: Boolean,
    default: false
  }
})

const getSparkStyle = (index) => {
  const angle = (index * 60) % 360
  const delay = index * 50
  return {
    '--angle': `${angle}deg`,
    '--delay': `${delay}ms`
  }
}

const getStarStyle = (index) => {
  const angle = (index * 45) % 360
  const delay = index * 30
  return {
    '--angle': `${angle}deg`,
    '--delay': `${delay}ms`
  }
}
</script>

<style scoped>
.seal-effect-container {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

/* Flash central */
.flash {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100px;
  height: 100px;
  background: radial-gradient(circle, rgba(255,255,255,0.8) 0%, rgba(255,255,255,0) 70%);
  border-radius: 50%;
  animation: flash 300ms ease-out forwards;
}

/* Chispas eléctricas */
.spark {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 2px;
  height: 20px;
  background: linear-gradient(to top, #FFD700, #FFA500);
  transform-origin: center;
  animation: spark 800ms ease-out forwards;
  animation-delay: var(--delay);
  opacity: 0;
  box-shadow: 0 0 8px #FFD700;
}

/* Partículas tipo estrella */
.star {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 4px;
  height: 4px;
  background: #FFD700;
  clip-path: polygon(50% 0%, 61% 35%, 98% 35%, 68% 57%, 79% 91%, 50% 70%, 21% 91%, 32% 57%, 2% 35%, 39% 35%);
  transform-origin: center;
  animation: star 1000ms ease-out forwards;
  animation-delay: var(--delay);
  opacity: 0;
}

/* Anillo de impacto */
.impact-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 120px;
  height: 120px;
  border: 2px solid #FFD700;
  border-radius: 50%;
  animation: impactRing 1000ms ease-out forwards;
  opacity: 0;
}

@keyframes flash {
  0% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0);
  }
  50% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1.2);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(1.5);
  }
}

@keyframes spark {
  0% {
    transform: translate(-50%, -50%) rotate(var(--angle)) translateY(0);
    opacity: 0;
  }
  20% {
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) rotate(var(--angle)) translateY(-120px);
    opacity: 0;
  }
}

@keyframes star {
  0% {
    transform: translate(-50%, -50%) rotate(var(--angle)) translateY(0) scale(0);
    opacity: 0;
  }
  50% {
    opacity: 1;
    transform: translate(-50%, -50%) rotate(var(--angle)) translateY(-60px) scale(1);
  }
  100% {
    transform: translate(-50%, -50%) rotate(var(--angle)) translateY(-120px) scale(0);
    opacity: 0;
  }
}

@keyframes impactRing {
  0% {
    transform: translate(-50%, -50%) scale(0);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) scale(1.5);
    opacity: 0;
  }
}
</style> 