<template>
  <div v-if="modelValue" class="fixed inset-0 bg-[var(--color-dark-background)]/80 flex justify-center items-center z-50 p-4 modal-fade-in">
    <div 
      ref="modalRef"
      class="bg-[var(--color-dark-background-lighter)] rounded-2xl p-6 w-full max-w-sm flex flex-col space-y-6 text-[#fefefe]"
      :class="{ 'is-transforming': isTransforming }"
    >
      <div v-show="showContent && !isTransforming">
        <h2 class="text-xl font-medium text-center">Guardar grito</h2>

        <div class="space-y-2">
          <div class="bg-[var(--color-dark-background)] p-4 rounded-lg">
            <AudioPlayer 
              :src="audioUrl" 
              :dark-mode="true"
              class="w-full"
            />
          </div>
        </div>

        <div class="space-y-2">
          <label for="audioName" class="block text-sm font-medium">Nombre del grito</label>
          <input
            type="text"
            id="audioName"
            v-model="title"
            class="w-full px-4 py-2 rounded-lg bg-white/10 border border-white/10 focus:border-white focus:outline-none transition-colors text-[#fefefe]"
            placeholder="Mi grito épico"
            @keyup.enter="handleSave"
          />
        </div>

        <div class="flex space-x-4">
          <button
            @click="closeModal"
            class="flex-1 bg-[var(--color-dark-background-light)] border border-white/10 text-[#fefefe] rounded-lg py-3 font-medium hover:bg-[var(--color-dark-background-lighter)] transition-colors duration-200"
          >
            Cancelar
          </button>
          <button
            @click="handleSave"
            :disabled="!title.trim() || isSaving"
            class="flex-1 bg-[var(--color-dark-background)] text-white rounded-lg py-3 font-medium hover:bg-[var(--color-dark-background-light)] transition-colors duration-200"
          >
            {{ isSaving ? 'Guardando...' : 'Guardar' }}
          </button>
        </div>
      </div>
      <div v-show="!showContent && !isTransforming" class="flex flex-col items-center justify-center h-full text-center p-8">
        <p class="text-xl font-semibold text-[#fefefe]">{{ statusMessage }}</p>
        <p class="text-sm text-[#ccc] mt-2">Espera un momento, esto toma su tiempo...</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../supabase'
import AudioPlayer from './AudioPlayer.vue'
import gsap from 'gsap'

const props = defineProps({
  modelValue: Boolean,
  audioBlob: Blob,
  audioUrl: {
    type: String,
    required: true
  },
  duration: Number
})

const emit = defineEmits(['update:modelValue', 'saved'])

const router = useRouter()
const title = ref('')
const isSaving = ref(false)
const modalRef = ref(null)
const isTransforming = ref(false)
const showSuccessMessage = ref(false)
const statusMessage = ref('')
const showContent = ref(true)

const closeModal = () => {
  emit('update:modelValue', false)
  setTimeout(() => {
    title.value = ''
    isSaving.value = false
    isTransforming.value = false
    statusMessage.value = ''
    showContent.value = true
  }, 300)
}

const handleSave = async () => {
  if (!title.value.trim()) return
  if (!props.audioBlob) return

  try {
    console.log('1. Iniciando guardado de audio')
    isSaving.value = true
    statusMessage.value = '¡Qué voz tan particular! Analizando...'
    showContent.value = false
    await new Promise(resolve => setTimeout(resolve, 2000))

    const { data: userData, error: userError } = await supabase.auth.getUser()
    if (userError || !userData.user) {
      console.error('Error al obtener el usuario:', userError)
      throw new Error('No se pudo obtener el usuario actual')
    }

    const user = userData.user
    console.log('Usuario actual:', user.id)

    const timestamp = new Date().getTime()
    const fileName = `${user.id}/${timestamp}.m4a`
    console.log('Nombre del archivo:', fileName)

    statusMessage.value = '¡Almacenando tu grito en el éter!'
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    const { data: uploadData, error: uploadError } = await supabase.storage
      .from('gritos')
      .upload(fileName, props.audioBlob, {
        contentType: props.audioBlob.type,
        cacheControl: '3600'
      })

    if (uploadError) {
      console.error('Error al subir el archivo:', uploadError)
      throw uploadError
    }

    console.log('Archivo subido exitosamente:', uploadData)

    const { data: { publicUrl } } = supabase.storage
      .from('gritos')
      .getPublicUrl(fileName)

    console.log('URL pública del archivo:', publicUrl)

    const { data: dbData, error: dbError } = await supabase
      .from('gritos')
      .insert([
        {
          user_id: user.id,
          name: title.value,
          audio_url: publicUrl,
          duration: props.duration,
          created_at: new Date().toISOString()
        }
      ])
      .select()

    if (dbError) {
      console.error('Error al guardar en la base de datos:', dbError)
      throw dbError
    }

    console.log('Datos guardados en la base de datos:', dbData)
    
    statusMessage.value = '¡Grito sellado y listo para la eternidad!'
    showSuccessMessage.value = true
    
    // Esperar 1.5 segundos y luego hacer fade out
    await new Promise(resolve => setTimeout(resolve, 1500))
    
    // Fade out suave
    gsap.to(modalRef.value, {
      opacity: 0,
      duration: 0.5,
      ease: 'power2.out',
      onComplete: () => {
        closeModal()
        emit('saved', dbData[0])
      }
    })
  } catch (error) {
    console.error('Error en el proceso de guardado:', error)
    statusMessage.value = '¡Error al guardar! Inténtalo de nuevo.'
    showContent.value = true
  } finally {
    isSaving.value = false
  }
}

watch(() => props.modelValue, (newValue) => {
  if (!newValue) {
    title.value = ''
    isTransforming.value = false
  }
})
</script>

<style scoped>
@keyframes modalFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.modal-fade-in {
  animation: modalFadeIn 0.5s ease-out forwards;
}

.is-transforming {
  transition: all 0.3s ease;
  transform-origin: center center;
}
</style> 