<template>
  <div class="min-h-screen bg-[var(--color-dark-background)] text-[#fefefe] pb-20">
    <div class="max-w-2xl mx-auto px-4 py-6">
      <h1 class="text-2xl font-bold mb-6">Amigos</h1>
      
      <!-- Lista de amigos -->
      <div class="space-y-4">
        <div v-for="friend in friends" :key="friend.id" 
          class="bg-[var(--color-dark-background-lighter)] p-4 rounded-lg flex items-center justify-between">
          <div class="flex items-center space-x-3">
            <div class="w-10 h-10 rounded-full bg-red-500 flex items-center justify-center">
              <span class="text-lg font-bold">{{ friend.email.charAt(0).toUpperCase() }}</span>
            </div>
            <div>
              <p class="font-medium">{{ friend.email }}</p>
              <p class="text-sm text-gray-400">Amigo desde {{ formatDate(friend.created_at) }}</p>
            </div>
          </div>
          <button class="text-red-500 hover:text-red-400 transition-colors">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { supabase } from '../supabase'

const friends = ref([])

const formatDate = (date) => {
  return new Date(date).toLocaleDateString('es-ES', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

const loadFriends = async () => {
  try {
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) return

    const { data, error } = await supabase
      .from('friends')
      .select('*')
      .eq('user_id', user.id)

    if (error) throw error
    friends.value = data
  } catch (error) {
    console.error('Error cargando amigos:', error)
  }
}

onMounted(() => {
  loadFriends()
})
</script> 