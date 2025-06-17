<template>
  <div v-if="modelValue" class="fixed inset-0 bg-[var(--color-dark-background)]/80 flex justify-center items-center z-50 p-4 modal-fade-in">
    <div 
      class="bg-[var(--color-dark-background-lighter)] rounded-2xl p-6 w-full max-w-sm flex flex-col space-y-6 text-[#fefefe]"
    >
      <h2 class="text-xl font-medium text-center">Crear nuevo grupo</h2>

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
          <p v-if="errorMessage" class="text-red-500 text-sm mt-1">{{ errorMessage }}</p>
        </div>
        <div>
          <label for="groupDescription" class="block text-sm font-medium mb-2">Descripción (opcional)</label>
          <textarea
            id="groupDescription"
            v-model="groupDescription"
            rows="3"
            class="w-full px-4 py-2 rounded-lg bg-white/10 border border-white/10 focus:border-white focus:outline-none transition-colors resize-none text-[#fefefe]"
            placeholder="Un grupo para compartir gritos..."></textarea>
        </div>
      </div>

      <div class="flex space-x-4">
        <button
          @click="closeModal"
          class="flex-1 bg-[var(--color-dark-background-light)] border border-white/10 text-[#fefefe] rounded-lg py-3 font-medium hover:bg-[var(--color-dark-background-lighter)] transition-colors duration-200"
        >
          Cancelar
        </button>
        <button
          @click="handleCreateGroup"
          :disabled="!groupName.trim() || isCreating"
          class="flex-1 bg-red-500 text-white rounded-lg py-3 font-medium hover:bg-red-600 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {{ isCreating ? 'Creando...' : 'Crear' }}
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
})

const emit = defineEmits(['update:modelValue', 'groupCreated'])

const groupName = ref('')
const groupDescription = ref('')
const isCreating = ref(false)
const errorMessage = ref('')

const closeModal = () => {
  emit('update:modelValue', false)
  groupName.value = ''
  groupDescription.value = ''
  isCreating.value = false
  errorMessage.value = ''
}

const clearError = () => {
  errorMessage.value = ''
}

const handleCreateGroup = async () => {
  if (!groupName.value.trim()) return

  isCreating.value = true
  errorMessage.value = ''

  try {
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) {
      throw new Error('Usuario no autenticado')
    }

    const { data, error } = await supabase
      .from('groups')
      .insert({
        name: groupName.value.trim(),
        description: groupDescription.value.trim() || null,
        created_by: user.id,
      })
      .select()

    if (error) {
      console.error('Error al crear el grupo:', error)
      throw error
    }
    
    const newGroup = data[0]

    // Añadir al creador como miembro del grupo con rol 'admin'
    const { error: memberError } = await supabase
      .from('group_members')
      .insert({
        group_id: newGroup.id,
        user_id: user.id,
        role: 'admin'
      })

    if (memberError) {
      console.error('Error al añadir creador como miembro:', memberError)
      throw memberError
    }

    emit('groupCreated', newGroup)
    closeModal()
  } catch (error) {
    console.error('Error en el proceso de creación de grupo:', error.message)
    errorMessage.value = `Error al crear el grupo: ${error.message}`
  } finally {
    isCreating.value = false
  }
}

watch(() => props.modelValue, (newValue) => {
  if (!newValue) {
    groupName.value = ''
    groupDescription.value = ''
    isCreating.value = false
    errorMessage.value = ''
  }
})
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