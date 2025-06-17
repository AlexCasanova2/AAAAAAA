<template>
  <div class="h-[100dvh] bg-[#fefefe] text-[#111] flex flex-col">
    <div class="safe-area-top">
      <div class="w-full p-4">
        <h1 class="text-2xl font-medium text-center">
          Iniciar sesión
        </h1>
      </div>
    </div>

    <div class="flex-1 p-4 flex flex-col justify-center">
      <form @submit.prevent="handleLogin" class="space-y-6">
        <div class="space-y-2">
          <label for="email" class="block text-sm font-medium">Email</label>
          <input
            id="email"
            v-model="email"
            type="email"
            required
            class="w-full px-4 py-2 rounded-lg bg-white/10 border border-[#111]/10 focus:border-[#111] focus:outline-none transition-colors"
            placeholder="tu@email.com"
          />
        </div>

        <div class="space-y-2">
          <label for="password" class="block text-sm font-medium">Contraseña</label>
          <input
            id="password"
            v-model="password"
            type="password"
            required
            class="w-full px-4 py-2 rounded-lg bg-white/10 border border-[#111]/10 focus:border-[#111] focus:outline-none transition-colors"
            placeholder="••••••••"
          />
        </div>

        <button
          ref="loginButton"
          type="submit"
          class="w-full bg-[#111] text-white rounded-lg py-3 font-medium hover:bg-[#333] transition-colors duration-200 relative"
          :disabled="loading"
        >
          {{ loading ? 'Iniciando sesión...' : 'Iniciar sesión' }}
        </button>

        <p v-if="error" class="text-red-500 text-sm text-center">
          {{ error }}
        </p>
      </form>

      <div class="mt-8 text-center">
        <p class="text-sm text-[#111]/70">
          ¿No tienes una cuenta?
          <button
            @click="goToRegister"
            class="text-[#111] font-medium hover:opacity-80 transition-opacity"
          >
            Regístrate
          </button>
        </p>
      </div>
    </div>

    <!-- Círculo de transición -->
    <TransitionCircle
      v-if="showTransition"
      :origin="transitionOrigin"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../supabase'
import TransitionCircle from '../components/TransitionCircle.vue'

const router = useRouter()
const email = ref('')
const password = ref('')
const loading = ref(false)
const error = ref('')
const loginButton = ref(null)
const showTransition = ref(false)
const transitionOrigin = ref({ x: 0, y: 0 })

const handleLogin = async () => {
  try {
    loading.value = true
    error.value = ''

    // Calcular el origen de la transición desde el botón
    const buttonRect = loginButton.value.getBoundingClientRect()
    transitionOrigin.value = {
      x: buttonRect.left + buttonRect.width / 2,
      y: buttonRect.top + buttonRect.height / 2
    }

    // Mostrar el círculo de transición
    showTransition.value = true

    // Esperar a que la animación esté casi completa antes de iniciar sesión
    await new Promise(resolve => setTimeout(resolve, 800))

    const { data, error: signInError } = await supabase.auth.signInWithPassword({
      email: email.value,
      password: password.value
    })

    if (signInError) throw signInError

    // Esperar a que la animación termine antes de navegar
    await new Promise(resolve => setTimeout(resolve, 400))
    router.push('/')
  } catch (err) {
    error.value = err.message
    showTransition.value = false
  } finally {
    loading.value = false
  }
}

const goToRegister = () => {
  router.push('/register')
}
</script>

<style scoped>
.safe-area-top {
  padding-top: env(safe-area-inset-top);
  background-color: #fefefe;
}
</style> 