<template>
    <div class="flex items-center space-x-2">
        <button @click="togglePlay" class="p-1 focus:outline-none focus:ring-0 bg-gradient-to-br from-red-500 to-red-600 bg-clip-text text-white">
            <Play v-if="!isPlaying" class="w-5 h-5 fill-current" />
            <Pause v-else class="w-5 h-5 fill-current" />
        </button>
        
        <div class="flex-1 relative h-1 rounded-full cursor-pointer" 
             :class="darkMode ? 'bg-white/10' : 'bg-[white]/10'"
             @click="seek">
            <div class="absolute h-full rounded-full" 
                 :class="darkMode ? 'bg-white' : 'bg-white'"
                 :style="{ width: `${progress}%` }"></div>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import { Play, Pause } from 'lucide-vue-next'

const props = defineProps({
    src: {
        type: String,
        required: true
    },
    darkMode: {
        type: Boolean,
        default: false
    }
})

const audio = ref(null)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const progress = ref(0)
const error = ref(null)
let animationFrameId = null;

// Función para actualizar el progreso suavemente (para la barra)
const updateProgressBar = () => {
    if (audio.value && isPlaying.value) {
        currentTime.value = audio.value.currentTime;
        progress.value = (audio.value.currentTime / audio.value.duration) * 100 || 0;
        animationFrameId = requestAnimationFrame(updateProgressBar);
    }
};

// Inicializar el elemento de audio
onMounted(() => {
    audio.value = new Audio();
    
    // Configurar el audio para reproducción automática
    audio.value.preload = 'metadata';
    audio.value.crossOrigin = 'anonymous';
    
    audio.value.addEventListener('loadedmetadata', () => {
        console.log('Audio cargado:', {
            duration: audio.value?.duration,
            readyState: audio.value?.readyState,
            src: audio.value?.src,
            networkState: audio.value?.networkState,
            error: audio.value?.error
        });
        duration.value = audio.value?.duration || 0;
        error.value = null;
    });
    
    audio.value.addEventListener('error', (e) => {
        console.error('Error de audio:', {
            error: e,
            code: audio.value?.error?.code,
            message: audio.value?.error?.message,
            src: audio.value?.src,
            networkState: audio.value?.networkState,
            readyState: audio.value?.readyState
        });
        isPlaying.value = false;
        error.value = true;
    });
    
    audio.value.addEventListener('play', () => {
        console.log('Reproducción iniciada:', {
            src: audio.value?.src,
            currentTime: audio.value?.currentTime,
            duration: audio.value?.duration
        });
        isPlaying.value = true;
        error.value = false;
        updateProgressBar();
    });
    
    audio.value.addEventListener('pause', () => {
        console.log('Reproducción pausada:', {
            src: audio.value?.src,
            currentTime: audio.value?.currentTime
        });
        isPlaying.value = false;
        cancelAnimationFrame(animationFrameId);
    });
    
    audio.value.addEventListener('ended', () => {
        console.log('Reproducción finalizada:', {
            src: audio.value?.src,
            duration: audio.value?.duration
        });
        isPlaying.value = false;
        currentTime.value = 0;
        progress.value = 0;
        cancelAnimationFrame(animationFrameId);
    });
    
    // Actualizar la barra de progreso
    audio.value.addEventListener('timeupdate', () => {
        if (audio.value) {
            currentTime.value = audio.value.currentTime;
        }
    });
    
    // Cargar la URL del audio
    if (props.src) {
        audio.value.src = props.src;
        audio.value.load();
    }
});

// Limpiar el evento al desmontar el componente
onUnmounted(() => {
    if (audio.value) {
        audio.value.pause();
        cancelAnimationFrame(animationFrameId);
        audio.value = null;
    }
});

// Reproducir/Pausar
const togglePlay = async () => {
    if (!audio.value) {
        console.error('No hay elemento de audio disponible');
        return;
    }

    try {
        if (isPlaying.value) {
            console.log('Pausando reproducción');
            await audio.value.pause();
        } else {
            console.log('Intentando reproducir:', {
                src: audio.value.src,
                readyState: audio.value.readyState,
                networkState: audio.value.networkState
            });
            
            // Intentar reproducir
            const playPromise = audio.value.play();
            if (playPromise !== undefined) {
                playPromise.catch(error => {
                    console.error('Error al reproducir:', {
                        error,
                        src: audio.value?.src,
                        readyState: audio.value?.readyState,
                        networkState: audio.value?.networkState
                    });
                    isPlaying.value = false;
                    error.value = true;
                });
            }
        }
    } catch (error) {
        console.error('Error al controlar la reproducción:', {
            error,
            src: audio.value?.src,
            readyState: audio.value?.readyState,
            networkState: audio.value?.networkState
        });
        isPlaying.value = false;
        error.value = true;
    }
};

// Buscar en el audio
const seek = (event) => {
    if (!audio.value || !audio.value.duration) return;
    const progressBar = event.currentTarget;
    const clickX = event.clientX - progressBar.getBoundingClientRect().left;
    const width = progressBar.offsetWidth;
    const seekTime = (clickX / width) * audio.value.duration;
    audio.value.currentTime = seekTime;
};

// Formatear el tiempo a MM:SS
const formatTime = (time) => {
    const minutes = Math.floor(time / 60)
    const seconds = Math.floor(time % 60)
    return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
}

// Observar cambios en la prop src
watch(() => props.src, (newSrc) => {
    console.log('Nueva URL de audio:', newSrc);
    if (audio.value) {
        audio.value.pause();
        cancelAnimationFrame(animationFrameId);
        isPlaying.value = false;
        currentTime.value = 0;
        progress.value = 0;
        error.value = null;
        audio.value.src = newSrc;
        audio.value.load();
    }
}, { immediate: true });
</script>

<style scoped>
/* Estilos específicos para este componente, aunque Tailwind CSS ya hace mucho */
</style>