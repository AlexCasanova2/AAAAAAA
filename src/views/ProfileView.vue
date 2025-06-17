<template>
  <div class="min-h-screen bg-[var(--color-dark-background)] text-[#fefefe] pb-20">
    <div class="max-w-2xl mx-auto px-4 py-6">
      <h1 class="text-2xl font-bold mb-6">Perfil</h1>
      
      <!-- Información del usuario -->
      <div class="bg-[var(--color-dark-background-lighter)] p-6 rounded-lg space-y-6">
        <div class="flex items-center space-x-4">
          <div class="w-20 h-20 rounded-full bg-red-500 flex items-center justify-center">
            <span class="text-3xl font-bold">{{ userEmail?.charAt(0).toUpperCase() }}</span>
          </div>
          <div>
            <h2 class="text-xl font-bold">{{ userEmail }}</h2>
            <p class="text-gray-400">Miembro desde {{ formatDate(userCreatedAt) }}</p>
          </div>
        </div>

        <!-- Estadísticas -->
        <div class="grid grid-cols-2 gap-4">
          <div class="bg-[var(--color-dark-background)] p-4 rounded-lg">
            <p class="text-gray-400 text-sm">Gritos grabados</p>
            <p class="text-2xl font-bold">{{ stats.recordings }}</p>
          </div>
          <div class="bg-[var(--color-dark-background)] p-4 rounded-lg">
            <p class="text-gray-400 text-sm">Grupos</p>
            <p class="text-2xl font-bold">{{ stats.groups }}</p>
          </div>
        </div>

        <!-- Acciones -->
        <div class="space-y-3">
          <button 
            @click="signOut"
            class="w-full bg-red-500 hover:bg-red-600 text-white font-medium py-2 px-4 rounded-lg transition-colors"
          >
            Cerrar sesión
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../supabase'

const router = useRouter()
const userEmail = ref('')
const userCreatedAt = ref('')
const stats = ref({
  recordings: 0,
  groups: 0
})

const formatDate = (date) => {
  if (!date) return ''
  return new Date(date).toLocaleDateString('es-ES', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

const loadUserData = async () => {
  try {
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) {
      router.push('/login')
      return
    }

    userEmail.value = user.email
    userCreatedAt.value = user.created_at

    // Cargar estadísticas
    const { count: recordingsCount } = await supabase
      .from('recordings')
      .select('*', { count: 'exact' })
      .eq('user_id', user.id)

    const { count: groupsCount } = await supabase
      .from('group_members')
      .select('*', { count: 'exact' })
      .eq('user_id', user.id)

    stats.value = {
      recordings: recordingsCount || 0,
      groups: groupsCount || 0
    }
  } catch (error) {
    console.error('Error cargando datos del usuario:', error)
  }
}

const signOut = async () => {
  try {
    await supabase.auth.signOut()
    router.push('/login')
  } catch (error) {
    console.error('Error cerrando sesión:', error)
  }
}

onMounted(() => {
  loadUserData()
})
</script> 