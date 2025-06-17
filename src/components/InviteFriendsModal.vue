<template>
  <div v-if="modelValue" class="fixed inset-0 bg-[var(--color-dark-background)]/80 flex justify-center items-center z-50 p-4 modal-fade-in">
    <div 
      class="bg-[var(--color-dark-background-lighter)] rounded-2xl p-6 w-full max-w-md flex flex-col space-y-6 text-[#fefefe]"
    >
      <div class="flex justify-between items-center">
        <h2 class="text-xl font-medium">Invitar amigos</h2>
        <button
          @click="shareLink"
          class="bg-red-500 text-white p-2 rounded-lg hover:bg-red-600 transition-colors duration-200 flex items-center space-x-2"
          :disabled="!canShare"
          title="Compartir grupo"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
            <path d="M15 8a3 3 0 10-2.977-2.63l-4.94 2.47a3 3 0 100 4.319l4.94 2.47a3 3 0 10.895-1.789l-4.94-2.47a3.027 3.027 0 000-.74l4.94-2.47C13.456 7.68 14.19 8 15 8z" />
          </svg>
        </button>
      </div>

      <!-- Pestañas de métodos de invitación -->
      <div class="flex space-x-2 border-b border-white/10">
        <button 
          v-for="tab in tabs" 
          :key="tab.id"
          @click="activeTab = tab.id"
          class="px-4 py-2 text-sm font-medium transition-colors duration-200"
          :class="activeTab === tab.id ? 'text-red-500 border-b-2 border-red-500' : 'text-gray-400 hover:text-[#fefefe]'"
        >
          {{ tab.label }}
        </button>
      </div>

      <!-- Contenido de las pestañas -->
      <div class="space-y-4">
        <!-- Invitación por Email -->
        <div v-if="activeTab === 'email'" class="space-y-4">
          <div>
            <label for="email" class="block text-sm font-medium mb-2">Email del amigo</label>
            <input
              type="email"
              id="email"
              v-model="email"
              class="w-full px-4 py-2 rounded-lg bg-white/10 border border-white/10 focus:border-white focus:outline-none transition-colors text-[#fefefe]"
              :class="{ 'border-red-500': errorMessage }"
              placeholder="amigo@ejemplo.com"
              @input="clearError"
            />
          </div>
          <button
            @click="handleInviteByEmail"
            :disabled="!email.trim() || isInviting"
            class="w-full bg-red-500 text-white rounded-lg py-3 font-medium hover:bg-red-600 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {{ isInviting ? 'Enviando...' : 'Enviar invitación' }}
          </button>
        </div>

        <!-- Invitación por WhatsApp -->
        <div v-if="activeTab === 'whatsapp'" class="space-y-4">
          <div>
            <label for="phone" class="block text-sm font-medium mb-2">Número de teléfono</label>
            <div class="flex space-x-2">
              <input
                type="tel"
                id="phone"
                v-model="phone"
                class="flex-1 px-4 py-2 rounded-lg bg-white/10 border border-white/10 focus:border-white focus:outline-none transition-colors text-[#fefefe]"
                placeholder="+34 600 000 000"
                @input="clearError"
              />
              <button
                @click="handleInviteByWhatsApp"
                :disabled="!phone.trim()"
                class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="currentColor" viewBox="0 0 24 24">
                  <path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413z"/>
                </svg>
              </button>
            </div>
          </div>
          <p class="text-sm text-gray-400">Se abrirá WhatsApp con un mensaje predefinido</p>
        </div>

        <!-- Invitación por Enlace -->
        <div v-if="activeTab === 'link'" class="space-y-4">
          <div>
            <label class="block text-sm font-medium mb-2">Enlace de invitación</label>
            <div class="flex space-x-2">
              <input
                type="text"
                :value="inviteLink"
                readonly
                class="flex-1 px-4 py-2 rounded-lg bg-white/10 border border-white/10 focus:border-white focus:outline-none transition-colors text-[#fefefe]"
              />
              <button
                @click="copyInviteLink"
                class="bg-[var(--color-dark-background-light)] text-[#fefefe] px-4 py-2 rounded-lg hover:bg-[var(--color-dark-background-lighter)] transition-colors duration-200"
                title="Copiar enlace"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3" />
                </svg>
              </button>
            </div>
          </div>
          <button
            @click="shareToWhatsApp"
            class="w-full bg-green-500 text-white rounded-lg py-3 font-medium hover:bg-green-600 transition-colors duration-200 flex items-center justify-center space-x-2"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="currentColor" viewBox="0 0 24 24">
              <path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413z"/>
            </svg>
            <span>Compartir por WhatsApp</span>
          </button>
          <p class="text-sm text-gray-400">Comparte este enlace con tus amigos para que se unan al grupo</p>
        </div>
      </div>

      <p v-if="errorMessage" class="text-red-500 text-sm">{{ errorMessage }}</p>

      <button
        @click="closeModal"
        class="w-full bg-[var(--color-dark-background-light)] border border-white/10 text-[#fefefe] rounded-lg py-3 font-medium hover:bg-[var(--color-dark-background-lighter)] transition-colors duration-200"
      >
        Cerrar
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { supabase } from '../supabase'

const props = defineProps({
  modelValue: Boolean,
  groupId: {
    type: String,
    required: true
  }
})

const emit = defineEmits(['update:modelValue', 'inviteSent'])

const tabs = [
  { id: 'email', label: 'Email' },
  { id: 'whatsapp', label: 'WhatsApp' },
  { id: 'link', label: 'Enlace' }
]

const activeTab = ref('email')
const email = ref('')
const phone = ref('')
const isInviting = ref(false)
const errorMessage = ref('')
const inviteLink = computed(() => {
  return `${window.location.origin}/groups/${props.groupId}/join`
})

const closeModal = () => {
  emit('update:modelValue', false)
  email.value = ''
  phone.value = ''
  isInviting.value = false
  errorMessage.value = ''
  activeTab.value = 'email'
}

const clearError = () => {
  errorMessage.value = ''
}

const handleInviteByEmail = async () => {
  if (!email.value.trim()) return

  isInviting.value = true
  errorMessage.value = ''

  try {
    // Buscar el usuario por email
    const { data: userData, error: userError } = await supabase
      .from('users')
      .select('id')
      .eq('email', email.value.trim())
      .single()

    if (userError) {
      throw new Error('Usuario no encontrado')
    }

    // Verificar si el usuario ya es miembro del grupo
    const { data: memberData, error: memberError } = await supabase
      .from('group_members')
      .select('id')
      .eq('group_id', props.groupId)
      .eq('user_id', userData.id)
      .single()

    if (memberData) {
      throw new Error('Este usuario ya es miembro del grupo')
    }

    // Añadir al usuario como miembro del grupo
    const { error: inviteError } = await supabase
      .from('group_members')
      .insert({
        group_id: props.groupId,
        user_id: userData.id,
        role: 'member'
      })

    if (inviteError) {
      throw inviteError
    }

    emit('inviteSent')
    closeModal()
  } catch (error) {
    console.error('Error al invitar:', error)
    errorMessage.value = error.message
  } finally {
    isInviting.value = false
  }
}

const handleInviteByWhatsApp = () => {
  if (!phone.value.trim()) return

  const message = `¡Únete a mi grupo en GritApp! ${inviteLink.value}`
  const whatsappUrl = `https://wa.me/${phone.value.replace(/[^0-9]/g, '')}?text=${encodeURIComponent(message)}`
  window.open(whatsappUrl, '_blank')
}

const copyInviteLink = async () => {
  try {
    await navigator.clipboard.writeText(inviteLink.value)
    errorMessage.value = 'Enlace copiado al portapapeles'
    setTimeout(() => {
      errorMessage.value = ''
    }, 2000)
  } catch (err) {
    errorMessage.value = 'Error al copiar el enlace'
  }
}

const canShare = computed(() => {
  return navigator.share !== undefined
})

const shareLink = async () => {
  if (!navigator.share) {
    errorMessage.value = 'Tu navegador no soporta la función de compartir'
    return
  }

  try {
    await navigator.share({
      title: '¡Únete a mi grupo en GritApp!',
      text: 'Te invito a unirte a mi grupo en GritApp',
      url: inviteLink.value
    })
  } catch (error) {
    if (error.name !== 'AbortError') {
      errorMessage.value = 'Error al compartir'
    }
  }
}

const shareToWhatsApp = () => {
  const message = `¡Únete a mi grupo en GritApp! ${inviteLink.value}`
  window.open(`https://wa.me/?text=${encodeURIComponent(message)}`, '_blank')
}
</script>

<style scoped>
.modal-fade-in {
  animation: fadeIn 0.2s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style> 