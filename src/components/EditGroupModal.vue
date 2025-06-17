<template>
  <div v-if="modelValue" class="fixed inset-0 bg-[var(--color-dark-background)]/80 flex justify-center items-center z-50 p-4 modal-fade-in">
    <div 
      class="bg-[var(--color-dark-background-lighter)] rounded-2xl p-6 w-full max-w-md flex flex-col space-y-6 text-[#fefefe]"
    >
      <h2 class="text-xl font-medium text-center">Editar grupo</h2>

      <div class="space-y-4">
        <div>
          <label for="groupName" class="block text-sm font-medium mb-2">Nombre del grupo</label>
          <input
            type="text"
            id="groupName"
            v-model="groupName"
            class="w-full px-4 py-2 rounded-lg bg-white/10 border border-white/10 focus:border-white focus:outline-none transition-colors text-[#fefefe]"
            :class="{ 'border-red-500': errorMessage }"
            placeholder="Mi grupo de gritos"
            @input="clearError"
          />
        </div>
        <div>
          <label for="groupDescription" class="block text-sm font-medium mb-2">Descripción</label>
          <textarea
            id="groupDescription"
            v-model="groupDescription"
            rows="3"
            class="w-full px-4 py-2 rounded-lg bg-white/10 border border-white/10 focus:border-white focus:outline-none transition-colors resize-none text-[#fefefe]"
            placeholder="Un grupo para compartir gritos..."></textarea>
        </div>
        <p v-if="errorMessage" class="text-red-500 text-sm">{{ errorMessage }}</p>
      </div>

      <div class="flex space-x-4">
        <button
          @click="closeModal"
          class="flex-1 bg-[var(--color-dark-background-light)] border border-white/10 text-[#fefefe] rounded-lg py-3 font-medium hover:bg-[var(--color-dark-background-lighter)] transition-colors duration-200"
        >
          Cancelar
        </button>
        <button
          @click="handleUpdate"
          :disabled="!groupName.trim() || isUpdating"
          class="flex-1 bg-red-500 text-white rounded-lg py-3 font-medium hover:bg-red-600 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {{ isUpdating ? 'Guardando...' : 'Guardar cambios' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { supabase } from '../supabase'

const props = defineProps({
  modelValue: Boolean,
  group: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['update:modelValue', 'groupUpdated'])

const groupName = ref('')
const groupDescription = ref('')
const isUpdating = ref(false)
const errorMessage = ref('')

const closeModal = () => {
  emit('update:modelValue', false)
  errorMessage.value = ''
}

const clearError = () => {
  errorMessage.value = ''
}

const handleUpdate = async () => {
  if (!groupName.value.trim()) return

  isUpdating.value = true
  errorMessage.value = ''

  try {
    const { error } = await supabase
      .from('groups')
      .update({
        name: groupName.value.trim(),
        description: groupDescription.value.trim() || null,
      })
      .eq('id', props.group.id)

    if (error) throw error

    emit('groupUpdated', {
      ...props.group,
      name: groupName.value.trim(),
      description: groupDescription.value.trim() || null
    })
    closeModal()
  } catch (error) {
    console.error('Error al actualizar el grupo:', error)
    errorMessage.value = error.message
  } finally {
    isUpdating.value = false
  }
}

// Actualizar los valores cuando cambia el grupo
watch(() => props.group, (newGroup) => {
  if (newGroup) {
    groupName.value = newGroup.name
    groupDescription.value = newGroup.description || ''
  }
}, { immediate: true })
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