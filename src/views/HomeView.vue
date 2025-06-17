<template>
  <div class="bg-[var(--color-dark-background-lighter)] text-[#fefefe] flex flex-col min-h-screen">
    <!-- Header fijo -->
    <AppHeader title="" :dark-mode="false">
      <template #left-content>
        
      </template>
      <template #right-content>
        
        <button 
          class="history-button p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Historial"
          @click="startTransition"
          ref="historyButton"
        >
          <Folder class="w-6 h-6 stroke-white" />
        </button>
      </template>
    </AppHeader>

    <!-- Contenido principal con el botón de grabar centrado -->
    <div class="flex-1 flex justify-center items-center relative">
      <!-- Progress Circle -->
      <svg
        v-if="isRecording"
        class="absolute w-28 h-28 transform -rotate-90"
        viewBox="0 0 120 120"
      >
        <circle
          cx="60"
          cy="60"
          r="54"
          fill="none"
          stroke="#eee"
          stroke-width="8"
        />
        <circle
          cx="60"
          cy="60"
          r="54"
          fill="none"
          stroke="#111"
          stroke-width="8"
          stroke-linecap="round"
          :stroke-dasharray="circumference"
          :stroke-dashoffset="progressOffset"
          class="transition-all duration-2000 ease-linear progress-circle"
        />
      </svg>

      <!-- Efecto de chispas -->
      <SparklesEffect :show="showSparkles" />

      <!-- Texto de la cuenta atrás posicionado sobre el micro -->
      <p v-if="showCountdown" :key="countdownText" :class="{'countdown-text-animation': true, 'tada-animation': countdownText === '¡GRITA!'}" class="absolute text-white text-4xl font-bold -top-16 z-20">{{ countdownText }}</p>

      <button 
        class="bg-[#111] text-white rounded-full w-20 h-20 flex items-center justify-center hover:bg-[#333] transition-colors duration-200 shadow-sm z-10" 
        aria-label="Gravar"
        @click="toggleRecording"
      >
        <Mic v-if="!isRecording" class="w-7 h-7 stroke-white" />
        <Square v-else class="w-7 h-7 stroke-white" />
      </button>
    </div>

    <!-- Vista de historial con transición circular -->
    <div 
      v-show="isTransitioning" 
      class="fixed inset-0 z-40 flex flex-col bg-[var(--color-dark-background)] text-white view-transition"
      :style="transitionStyle"
    >
      <!-- Contenido de historial real será cargado en HistoryView.vue -->
      <div class="flex-1 p-4" :style="{ paddingTop: `calc(3.25rem + env(safe-area-inset-top))` }">
        <!-- Este div ya no contendrá el AppHeader, ya que HistoryView.vue lo tiene -->
      </div>
    </div>

    <!-- Modal para guardar audio -->
    <SaveAudioModal
      v-model="showSaveModal"
      :audio-blob="recordedAudio"
      :audio-url="recordedAudioUrl"
      :duration="recordingDuration"
      @saved="handleAudioSaved"
    />
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted, nextTick } from 'vue'
import { Folder, Mic, ArrowLeft, User, Square, Users } from 'lucide-vue-next'
import { useRouter } from 'vue-router'
import SaveAudioModal from '../components/SaveAudioModal.vue'
import AppHeader from '../components/AppHeader.vue'
import SparklesEffect from '../components/SparklesEffect.vue'
import { supabase } from '../supabase'

const router = useRouter()
const historyButton = ref(null)
const isTransitioning = ref(false)
const transitionOrigin = ref({ x: 0, y: 0 })
const circleSize = ref(24)
const isRecording = ref(false)
const mediaRecorder = ref(null)
const audioChunks = ref([])
const recordingProgress = ref(100)
const audioStream = ref(null)
const startTime = ref(0)
const isManualStop = ref(false)

const showSaveModal = ref(false)
const recordedAudio = ref(null)
const recordedAudioUrl = ref('')
const recordingDuration = ref(0)

const showCountdown = ref(false)
const countdownText = ref('')

const showSparkles = ref(false)

const circumference = computed(() => 2 * Math.PI * 54)
const progressOffset = computed(() => (recordingProgress.value / 100) * circumference.value)

const transitionStyle = computed(() => {
  const x = transitionOrigin.value.x
  const y = transitionOrigin.value.y
  const size = circleSize.value

  return {
    viewTransitionName: 'circular-reveal',
    '--circle-x': `${x}px`,
    '--circle-y': `${y}px`,
    '--circle-size': `${size}px`,
    '--circle-color': 'var(--color-dark-background)'
  }
})

const startTransition = () => {
  if (!historyButton.value) return
  
  const rect = historyButton.value.getBoundingClientRect()
  transitionOrigin.value = {
    x: rect.left + rect.width / 2,
    y: rect.top + rect.height / 2
  }
  
  isTransitioning.value = true
  router.push('/history');

  const startTime = performance.now()
  const duration = 1200
  const maxSize = Math.max(window.innerWidth, window.innerHeight) * 4
  
  const animate = (currentTime) => {
    const elapsed = currentTime - startTime
    const progress = Math.min(elapsed / duration, 1)
    
    const easedProgress = progress < 0.5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2
    
    circleSize.value = 24 + (maxSize - 24) * easedProgress
    
    if (progress < 1) {
      requestAnimationFrame(animate)
    } else {
      isTransitioning.value = false;
    }
  }
  
  requestAnimationFrame(animate)
}

const goBack = () => {
  router.push('/')
}

const goToProfile = () => {
  router.push('/profile')
}

const goToGroups = () => {
  router.push('/groups')
}

// Añadir una variable para el ID de la animación
let animationFrameId = null

const startRecording = async () => {
  try {
    // Verificar si el navegador soporta getUserMedia
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      throw new Error('Tu navegador no soporta la grabación de audio');
    }

    // Solicitar permisos de audio
    const stream = await navigator.mediaDevices.getUserMedia({ 
      audio: {
        echoCancellation: true,
        noiseSuppression: true,
        autoGainControl: true
      } 
    });
    
    audioStream.value = stream;
    
    // Intentar usar el formato más compatible con iOS
    const mimeType = 'audio/mp4';
    const options = { mimeType };
    
    try {
      mediaRecorder.value = new MediaRecorder(stream, options);
    } catch (e) {
      console.log('No se pudo usar audio/mp4, intentando con audio/webm');
      mediaRecorder.value = new MediaRecorder(stream, {
        mimeType: 'audio/webm;codecs=opus'
      });
    }
    
    audioChunks.value = []
    recordingProgress.value = 100
    isManualStop.value = false

    mediaRecorder.value.ondataavailable = (event) => {
      console.log('Datos de audio recibidos:', event.data.size, 'bytes')
      if (!isManualStop.value) {
        audioChunks.value.push(event.data)
      }
    }

    mediaRecorder.value.onstop = () => {
      console.log('MediaRecorder onstop triggered')
      if (!isManualStop.value) {
        console.log('Número de chunks:', audioChunks.value.length)
        console.log('Tamaño total de chunks:', audioChunks.value.reduce((acc, chunk) => acc + chunk.size, 0), 'bytes')
        
        // Crear el Blob con el tipo MIME correcto
        recordedAudio.value = new Blob(audioChunks.value, { 
          type: mediaRecorder.value.mimeType
        })
        
        console.log('Blob creado:', recordedAudio.value.size, 'bytes')
        console.log('Tipo MIME del Blob:', recordedAudio.value.type)
        
        const actualDuration = (performance.now() - startTime.value) / 1000
        console.log('Duración real de la grabación:', actualDuration.toFixed(2), 'segundos')

        // Crear URL del audio
        if (recordedAudio.value) {
          recordedAudioUrl.value = URL.createObjectURL(recordedAudio.value)
        }

        // Esperar a que termine el efecto de sellado antes de mostrar el modal
        setTimeout(() => {
          showSaveModal.value = true
        }, 1200)

        // Ocultar el efecto de sellado después de que termine
        setTimeout(() => {
          showSparkles.value = false
        }, 1000)
      }
    }

    // Configurar para recibir datos cada 100ms
    mediaRecorder.value.start(100)
    isRecording.value = true

    startTime.value = performance.now();
    const duration = 2500; // 2.5 segundos

    console.log('Iniciando grabación y animación');

    const animateProgress = (currentTime) => {
      const elapsed = currentTime - startTime.value
      const progress = Math.min(elapsed / duration, 1)
      
      // Asegurarnos de que el progreso llegue a 0
      recordingProgress.value = Math.max(0, 100 - (progress * 100))

      if (progress < 1) {
        animationFrameId = requestAnimationFrame(animateProgress)
      } else {
        // Forzar el progreso a 0 al final y esperar un momento antes de detener
        recordingProgress.value = 0
        // Mostrar el efecto de sellado
        showSparkles.value = true
        setTimeout(() => {
          console.log('Animación completada - activando acciones post-animación')
          handlePostAnimation()
          // Ocultar el efecto después de la animación
          setTimeout(() => {
            showSparkles.value = false
          }, 600)
        }, 100)
      }
    }
    animationFrameId = requestAnimationFrame(animateProgress)

  } catch (error) {
    console.error('Error al iniciar la grabación:', error);
    if (error.name === 'NotAllowedError') {
      alert('Por favor, permite el acceso al micrófono en la configuración de tu dispositivo para poder grabar.');
    } else if (error.name === 'NotFoundError') {
      alert('No se encontró ningún dispositivo de audio. Por favor, verifica que tu dispositivo tenga un micrófono.');
    } else {
      alert('Error al iniciar la grabación: ' + error.message);
    }
  }
}

const handlePostAnimation = () => {
  console.log('HomeView: handlePostAnimation llamado. Deteniendo grabación.')
  stopRecording(false) // Parada automática
}

const stopRecording = (manualStop = false) => {
  console.log('stopRecording llamado, isRecording:', isRecording.value, 'isManualStop:', manualStop)
  if (mediaRecorder.value && isRecording.value) {
    console.log('Deteniendo MediaRecorder')
    
    // Si es una parada manual, limpiar los datos antes de detener
    if (manualStop) {
      isManualStop.value = true // Establecer el flag antes de detener
      audioChunks.value = [] // Limpiar los chunks de audio
      recordedAudio.value = null
      recordedAudioUrl.value = ''
      recordingDuration.value = 0
      showSparkles.value = false
      recordingProgress.value = 100 // Resetear el progreso
    }

    mediaRecorder.value.stop()
    isRecording.value = false

    if (audioStream.value) {
      console.log('Deteniendo pistas de audio')
      audioStream.value.getTracks().forEach(track => track.stop())
      audioStream.value = null
    }
  }
}

const handleAudioSaved = (savedAudio) => {
  console.log('Audio guardado:', savedAudio)
  // Cerrar el modal
  showSaveModal.value = false
  // Limpiar las referencias DESPUÉS de que el modal se cierre completamente (para evitar warnings)
  nextTick(() => {
    recordedAudio.value = null
    recordedAudioUrl.value = ''
    recordingDuration.value = 0
  })
}

const handleCancelSave = () => {
  showSaveModal.value = false
  recordedAudio.value = null
}

const toggleRecording = () => {
  if (isRecording.value) {
    // Si está grabando, detener la grabación manualmente
    stopRecording(true)
    // Cancelar cualquier animación pendiente
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId)
      animationFrameId = null
    }
  } else {
    startCountdown()
  }
}

const startCountdown = () => {
  showCountdown.value = true;
  countdownText.value = '¿Estás preparado?';

  setTimeout(() => {
    countdownText.value = '3';
    setTimeout(() => {
      countdownText.value = '2';
      setTimeout(() => {
        countdownText.value = '1';
        setTimeout(() => {
          countdownText.value = '¡GRITA!';
          startRecording(); // Iniciar la grabación inmediatamente al mostrar ¡GRITA!
          setTimeout(() => {
            showCountdown.value = false;
          }, 800); // Duración de "¡GRITA!" antes de ocultar
        }, 1200); // Duración de 1 antes de ¡GRITA! (Aumentado a 1200ms)
      }, 1200); // Duración de 2 antes de 1 (Aumentado a 1200ms)
    }, 1200); // Duración de 3 antes de 2 (Aumentado a 1200ms)
  }, 1200); // Duración de "¿Estás preparado?" antes de 3 (Aumentado a 1200ms)
};

onUnmounted(() => {
  if (audioStream.value) {
    audioStream.value.getTracks().forEach(track => track.stop());
  }
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
  }
})
</script>

<style>
@keyframes circular-reveal {
  from {
    clip-path: circle(0px at var(--circle-x) var(--circle-y));
  }
  to {
    clip-path: circle(var(--circle-size) at var(--circle-x) var(--circle-y));
  }
}

.view-transition {
  animation: circular-reveal 1200ms cubic-bezier(0.4, 0, 0.2, 1) forwards;
  background-color: var(--circle-color);
}

.progress-circle {
  transition: stroke-dashoffset 100ms linear;
}

/* Keyframes para la animación de caída y rebote del texto de cuenta atrás */
@keyframes fallAndBounce {
  0% {
    opacity: 0;
    transform: translateY(-30px); 
  }
  60% {
    opacity: 1;
    transform: translateY(0);
  }
  75% {
    transform: translateY(-5px); /* Rebote más sutil */
  }
  100% {
    opacity: 1;
    transform: translateY(0); 
  }
}

.countdown-text-animation {
  animation: fallAndBounce 0.5s ease-out; /* Animación más rápida */
}

/* Keyframes para la animación 'tada' de ¡GRITA! */
@keyframes tada {
  from {
    transform: scale3d(1, 1, 1);
  }

  10%, 20% {
    transform: scale3d(0.9, 0.9, 0.9) rotate3d(0, 0, 1, -3deg);
  }

  30%, 50%, 70%, 90% {
    transform: scale3d(1.1, 1.1, 1.1) rotate3d(0, 0, 1, 3deg);
  }

  40%, 60%, 80% {
    transform: scale3d(1.1, 1.1, 1.1) rotate3d(0, 0, 1, -3deg);
  }

  to {
    transform: scale3d(1, 1, 1);
  }
}

.tada-animation {
  animation: tada 1s ease-in-out;
}
</style>

<style scoped>
/* No es necesario .safe-area-top aquí ya que AppHeader lo maneja */
</style> 