<template>
  <div class="h-[100dvh] bg-[#fefefe] text-[#111] flex flex-col">
    <div class="safe-area-top">
      <div class="w-full p-4">
        <h1 class="text-2xl font-medium text-center">
          Crear cuenta
        </h1>
      </div>
    </div>

    <div class="flex-1 p-4 flex flex-col justify-center">
      <form @submit.prevent="handleRegister" class="space-y-6">
        <div class="space-y-2">
          <label for="name" class="block text-sm font-medium">Nombre</label>
          <input
            id="name"
            v-model="name"
            type="text"
            required
            class="w-full px-4 py-2 rounded-lg bg-white/10 border border-[#111]/10 focus:border-[#111] focus:outline-none transition-colors"
            placeholder="Tu nombre"
          />
        </div>

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
          type="submit"
          class="w-full bg-[#111] text-white rounded-lg py-3 font-medium hover:bg-[#333] transition-colors duration-200"
          :disabled="loading"
        >
          {{ loading ? 'Creando cuenta...' : 'Crear cuenta' }}
        </button>

        <p v-if="error" class="text-red-500 text-sm text-center">
          {{ error }}
        </p>
      </form>

      <div class="mt-8 text-center">
        <p class="text-sm text-[#111]/70">
          ¿Ya tienes una cuenta?
          <button
            @click="goToLogin"
            class="text-[#111] font-medium hover:opacity-80 transition-opacity"
          >
            Inicia sesión
          </button>
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../supabase'

const router = useRouter()
const name = ref('')
const email = ref('')
const password = ref('')
const loading = ref(false)
const error = ref('')

const handleRegister = async () => {
  try {
    loading.value = true
    error.value = ''

    const { data, error: signUpError } = await supabase.auth.signUp({
      email: email.value,
      password: password.value,
      options: {
        data: {
          name: name.value
        }
      }
    })

    if (signUpError) throw signUpError

    router.push('/')
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

const goToLogin = () => {
  router.push('/login')
}
</script>

<style scoped>
.safe-area-top {
  padding-top: env(safe-area-inset-top);
  background-color: #fefefe;
}
</style> 