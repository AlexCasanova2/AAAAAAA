<template>
  <div class="h-[100dvh] bg-[var(--color-dark-background)] text-white flex flex-col">
    <!-- Header fijo -->
    <AppHeader title="Historial" :dark-mode="true">
      <template #right-content>
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Tornar"
          @click="startBackTransition"
          ref="backButton"
        >
          <ArrowLeft class="w-6 h-6 stroke-white" />
        </button>
      </template>
    </AppHeader>

    <!-- Contenido principal con padding superior para compensar el header fijo -->
    <div class="flex-1 p-4" :style="{ paddingTop: `calc(4rem + env(safe-area-inset-top))` }">
      <div v-if="loading" class="flex justify-center items-center h-full">
        <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-white"></div>
      </div>
      <div v-else-if="groupedRecordings.length === 0" class="flex justify-center items-center h-full">
        <p class="text-white/70">No hay grabaciones</p>
      </div>
      <div v-else class="space-y-4">
        <div 
          v-for="(group, index) in groupedRecordings" 
          :key="group.month"
          class="rounded-lg p-4"
          :style="{ 'animation-delay': `${index * 50}ms` }"
        >
          <h3 class="font-medium text-lg mb-3">{{ group.month }}</h3>
          <div class="space-y-3">
            <div 
              v-for="(recording, recordingIndex) in group.recordings" 
              :key="recording.id"
              class="bg-gradient-to-br from-gray-800 to-gray-900 rounded-lg p-5 transition-all duration-300 ease-in-out shadow-lg border border-white/10 animate-fade-in-up hover:scale-[1.01] hover:shadow-xl"
              :style="{ 'animation-delay': `${recordingIndex * 50}ms` }"
            >
              <div class="flex justify-between items-start mb-2">
                <div>
                  <h4 class="font-semibold text-xl">{{ recording.title }}</h4>
                  <p class="text-gray-300 text-sm">{{ formatTime(recording.createdAt) }}</p>
                </div>
                <button 
                  @click="deleteRecording(recording)"
                  class="p-2 hover:bg-white/20 rounded-full transition-colors duration-200"
                  :disabled="isDeleting === recording.id"
                >
                  <Trash2 v-if="isDeleting !== recording.id" class="w-5 h-5 stroke-white/70 hover:stroke-white" />
                  <Loader2 v-else class="w-5 h-5 stroke-white animate-spin" />
                </button>
              </div>
              <AudioPlayer :src="recording.filePath" class="w-full" />
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Vista de inicio con transición circular -->
    <div 
      v-show="isTransitioning" 
      class="fixed inset-0 z-40 flex flex-col bg-[#fefefe] text-[#111] view-transition"
      :style="transitionStyle"
    >
      <!-- Contenido de HomeView que se revela -->
      <div class="flex-1 flex justify-center items-center" :style="{ paddingTop: `calc(4rem + env(safe-area-inset-top))` }">
        <!-- Este div ya no contendrá el AppHeader ni los botones de HomeView, ya que HomeView.vue los tiene -->
      </div>
    </div>

    <!-- Modal de confirmación -->
    <ConfirmModal
      v-model="showConfirmModal"
      title="Aviso, estás a punto de borrar tu ¡Grito!, ¿estás seguro?"
      message="Esta acción no se puede deshacer."
      confirm-text="Sí, estoy seguro"
      cancel-text="Cambié de opinión"
      reasons-title="¿Por qué quieres borrar tu Grito?"
      :reasons="[    { value: 'No me gusta mi grito', label: 'No me gusta mi grito' },
        { value: 'Ha habido un error en la grabacion', label: 'Ha habido un error en la grabación' },
        { value: 'Contenido inapropiado', label: 'Contenido inapropiado' },
        { value: 'Otros', label: 'Otros' }    ]"
      @confirm="handleConfirmDelete"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { ArrowLeft, Folder, Mic, User, Trash2, Loader2 } from 'lucide-vue-next'
import { useRouter } from 'vue-router'
import { supabase } from '../supabase'
import AppHeader from '../components/AppHeader.vue'
import AudioPlayer from '../components/AudioPlayer.vue'
import ConfirmModal from '../components/ConfirmModal.vue'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL; // Importamos la URL de Supabase
const SUPABASE_BUCKET_NAME = 'gritos'; // Nombre del bucket

const router = useRouter()
const backButton = ref(null)
const isTransitioning = ref(false)
const transitionOrigin = ref({ x: 0, y: 0 })
const circleSize = ref(24)
const groupedRecordings = ref([])
const loading = ref(true)
const isDeleting = ref(null)
const showConfirmModal = ref(false)
const recordingToDelete = ref(null)

const loadRecordings = async () => {
  loading.value = true
  try {
    const { data, error } = await supabase
      .from('gritos')
      .select('id, audio_url, created_at, name')
      .order('created_at', { ascending: false })

    if (error) throw error

    // Agrupar grabaciones por mes
    const grouped = data.reduce((acc, item) => {
      const date = new Date(item.created_at)
      const year = date.getFullYear()
      const month = date.getMonth() // 0-indexed
      const monthKey = `${year}-${month}`

      if (!acc[monthKey]) {
        acc[monthKey] = { 
          month: formatMonth(date), 
          recordings: [] 
        }
      }

      // Obtener la URL pública del audio
      let audioUrl;
      if (item.audio_url.startsWith('data:')) {
        // Si es una URL de datos, usarla directamente
        audioUrl = item.audio_url;
      } else {
        // Si es una ruta de Supabase, construir la URL correctamente
        const path = item.audio_url.replace(/^https?:\/\/[^\/]+\/storage\/v1\/object\/public\/gritos\//, '');
        const { data: publicUrlData } = supabase.storage
          .from('gritos')
          .getPublicUrl(path);
        audioUrl = publicUrlData.publicUrl;
      }

      console.log('URL de audio procesada:', {
        original: item.audio_url,
        processed: audioUrl,
        path: item.audio_url.replace(/^https?:\/\/[^\/]+\/storage\/v1\/object\/public\/gritos\//, '')
      });

      acc[monthKey].recordings.push({
        id: item.id,
        title: item.name,
        createdAt: item.created_at,
        filePath: audioUrl
      })
      return acc
    }, {})

    // Convertir el objeto agrupado a un array ordenado
    groupedRecordings.value = Object.keys(grouped)
      .sort((a, b) => new Date(b) - new Date(a)) // Ordenar por mes descendente
      .map(key => grouped[key])

  } catch (error) {
    console.error('Error cargando grabaciones:', error.message)
  } finally {
    loading.value = false
  }
}

const formatTime = (timestamp) => {
  const date = new Date(timestamp)
  const now = new Date()
  const diff = now - date

  const minutes = Math.floor(diff / 60000)
  const hours = Math.floor(minutes / 60)
  const days = Math.floor(hours / 24)

  if (days > 0) return `Hace ${days} días`
  if (hours > 0) return `Hace ${hours} horas`
  if (minutes > 0) return `Hace ${minutes} minutos`
  return 'Ahora mismo'
}

const formatMonth = (date) => {
  const options = { year: 'numeric', month: 'long' }
  return date.toLocaleDateString('es-ES', options)
}

onMounted(() => {
  loadRecordings()
})

const transitionStyle = computed(() => {
  const x = transitionOrigin.value.x
  const y = transitionOrigin.value.y
  const size = circleSize.value

  return {
    viewTransitionName: 'circular-reveal',
    '--circle-x': `${x}px`,
    '--circle-y': `${y}px`,
    '--circle-size': `${size}px`,
    '--circle-color': '#fefefe' // Color de fondo de HomeView
  }
})

const startBackTransition = () => {
  if (!backButton.value) return
  
  const rect = backButton.value.getBoundingClientRect()
  transitionOrigin.value = {
    x: rect.left + rect.width / 2,
    y: rect.top + rect.height / 2
  }
  
  isTransitioning.value = true
  router.push('/'); // Mover la navegación aquí al inicio de la animación

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
      // router.push('/'); // Eliminamos la navegación de aquí
    }
  }
  
  requestAnimationFrame(animate)
}

const goToProfile = () => {
  router.push('/profile')
}

const deleteRecording = (recording) => {
  recordingToDelete.value = recording
  showConfirmModal.value = true
}

const confirmDelete = async (reasonData) => {
  if (!recordingToDelete.value) return
  
  try {
    isDeleting.value = true
    
    // Guardar el motivo de eliminación en una colección de logs en Supabase
    const { error: logError } = await supabase
      .from('deletion_logs')
      .insert({
        recording_id: recordingToDelete.value.id,
        reason_selected: reasonData.selected,
        other_reason: reasonData.otherText || null,
        created_at: new Date().toISOString()
      })

    if (logError) throw logError

    // Extraer el path del archivo de la URL (si es una URL de Supabase)
    const path = recordingToDelete.value.filePath.replace(/^https?:\/\/[^\/]+\/storage\/v1\/object\/public\/gritos\//, '');

    // Eliminar el archivo de storage de Supabase
    const { error: storageError } = await supabase.storage
      .from(SUPABASE_BUCKET_NAME)
      .remove([path])

    if (storageError) throw storageError
    
    // Eliminar el registro de la base de datos de Supabase
    const { error: dbError } = await supabase
      .from('gritos')
      .delete()
      .eq('id', recordingToDelete.value.id)
    
    if (dbError) throw dbError
    
    // Recargar la lista
    await loadRecordings()
  } catch (error) {
    console.error('Error al eliminar y registrar el motivo en Supabase:', error.message)
  } finally {
    isDeleting.value = false
    showConfirmModal.value = false
    recordingToDelete.value = null
  }
}

const handleConfirmDelete = (reasonData) => {
  console.log('Motivo de eliminación (Supabase):', reasonData.selected)
  if (reasonData.selected === 'Otros' && reasonData.otherText) {
    console.log('Detalle del motivo (Supabase - Otros):', reasonData.otherText)
  }
  confirmDelete(reasonData)
}
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

@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in-up {
  animation: fade-in-up 1s ease-out forwards;
  opacity: 0;
}
</style> 