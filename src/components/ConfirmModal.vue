<template>
  <div v-show="modelValue" :class="['fixed bottom-0 left-0 right-0 h-full bg-[var(--color-dark-background)]/80 flex justify-center items-center z-50', { 'modal-slide-up': !closing && modelValue, 'modal-slide-down': closing }]">
    <div
      class="bg-[var(--color-dark-background)] w-full flex flex-col text-[#fefefe] h-full max-h-[90%]"
    >
      <div class="text-center px-6 pt-10 pb-4">
        <h2 class="text-xl font-medium mb-2">{{ title }}</h2>
        <p class="text-[#ccc] mb-4">{{ message }}</p>

        <h3 v-if="reasons && reasons.length > 0" class="text-lg font-medium mb-3">{{ reasonsTitle || '¿Por qué?' }}</h3>
      </div>
      <div class="flex-1 overflow-y-auto px-6 pb-4">
        <div v-if="reasons && reasons.length > 0" class="space-y-4 text-left">
          <label v-for="reasonOption in reasons" :key="reasonOption.value" class="flex items-center space-x-2 cursor-pointer">
            <input type="radio" name="reason" :value="reasonOption.value" v-model="selectedReason" class="hidden"/>
            <div :class="['w-6 h-6 rounded-full border-2 border-white flex items-center justify-center transition-colors duration-200', { 'bg-red-500 border-red-500': selectedReason === reasonOption.value }]">
              <Check v-if="selectedReason === reasonOption.value" class="w-4 h-4 text-white" />
            </div>
            <span>{{ reasonOption.label }}</span>
          </label>
        </div>

        <div v-if="selectedReason === 'Otros'" class="mt-4">
          <textarea
            v-model="otherReasonText"
            rows="3"
            class="w-full px-4 py-2 rounded-lg bg-white/5 border border-[#fefefe]/10 focus:border-[#fefefe] focus:outline-none transition-colors resize-none text-[#fefefe]"
            placeholder="Escribe aquí el motivo..."
          ></textarea>
        </div>
      </div>

      <div class="px-6 py-4 border-t border-[var(--color-dark-background-light)]">
        <div class="flex flex-col space-y-4">
          <button
            @click="handleCancel"
            class="flex-1 bg-[var(--color-dark-background-light)] border border-[#fefefe]/10 text-[#fefefe] rounded-lg py-3 font-medium hover:bg-[var(--color-dark-background-lighter)] transition-colors duration-200"
          >
            {{ cancelText }}
          </button>
          <button
            @click="handleConfirm"
            :disabled="!isConfirmEnabled"
            class="flex-1 bg-red-500 text-white rounded-lg py-3 font-medium hover:bg-red-600 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {{ confirmText }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'
import { Check } from 'lucide-vue-next'

const props = defineProps({
  modelValue: Boolean,
  title: {
    type: String,
    default: 'Confirmar acción'
  },
  message: {
    type: String,
    required: true
  },
  confirmText: {
    type: String,
    default: 'Confirmar'
  },
  cancelText: {
    type: String,
    default: 'Cancelar'
  },
  reasonsTitle: {
    type: String,
    default: null
  },
  reasons: {
    type: Array,
    default: () => [] // Array de objetos { value: string, label: string }
  }
})

const emit = defineEmits(['update:modelValue', 'confirm', 'cancel'])

const selectedReason = ref(null)
const otherReasonText = ref('')
const closing = ref(false)

const ANIMATION_DURATION = 300;

watch(() => props.modelValue, (newValue) => {
  if (!newValue) {
    selectedReason.value = null
    otherReasonText.value = ''
    closing.value = false;
  }
})

const isConfirmEnabled = computed(() => {
  if (props.reasons.length > 0 && !selectedReason.value) {
    return false
  }
  if (selectedReason.value === 'Otros') {
    return otherReasonText.value.trim() !== ''
  }
  return true
})

const handleConfirm = () => {
  closing.value = true;
  setTimeout(() => {
    const reasonData = props.reasons.length > 0 ? {
      selected: selectedReason.value,
      otherText: selectedReason.value === 'Otros' ? otherReasonText.value : null
    } : null;
    emit('confirm', reasonData)
    emit('update:modelValue', false)
  }, ANIMATION_DURATION);
}

const handleCancel = () => {
  closing.value = true;
  setTimeout(() => {
    emit('cancel')
    emit('update:modelValue', false)
  }, ANIMATION_DURATION); 
}
</script>

<style scoped>
.modal-slide-up {
  animation: slideUp 0.3s ease-out forwards;
}

@keyframes slideUp {
  from {
    transform: translateY(100%);
  }
  to {
    transform: translateY(0%);
  }
}

.modal-slide-down {
  animation: slideDown 0.3s ease-in forwards;
}

@keyframes slideDown {
  from {
    transform: translateY(0%);
  }
  to {
    transform: translateY(100%);
  }
}
</style> 