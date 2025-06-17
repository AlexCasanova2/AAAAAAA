<template>
  <div class="h-[100dvh] bg-[var(--color-dark-background)] text-white flex flex-col">
    <AppHeader title="Mis Grupos" :dark-mode="true">
      <template #left-content>
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Tornar"
          @click="goToHome"
        >
          <ArrowLeft class="w-6 h-6 stroke-white" />
        </button>
      </template>
      <template #right-content>
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Crear Grupo"
          @click="createGroup"
        >
          <Plus class="w-6 h-6 stroke-white" />
        </button>
      </template>
    </AppHeader>

    <div class="flex-1 p-4" :style="{ paddingTop: `calc(4rem + env(safe-area-inset-top))` }">
      <div v-if="loadingGroups" class="flex justify-center items-center h-full">
        <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-white"></div>
      </div>
      <div v-else-if="userGroups.length === 0" class="flex justify-center items-center h-full">
        <p class="text-white/70 text-center">No eres miembro de ningún grupo.<br/>¡Crea uno o únete a uno existente!</p>
      </div>
      <div v-else class="space-y-4">
        <div 
          v-for="group in userGroups" 
          :key="group.id"
          class="bg-gradient-to-br from-gray-800 to-gray-900 rounded-lg p-5 transition-all duration-300 ease-in-out shadow-lg border border-white/10 animate-fade-in-up hover:scale-[1.01] hover:shadow-xl cursor-pointer"
          @click="goToGroupDetail(group.id)"
        >
          <h4 class="font-semibold text-xl mb-1">{{ group.name }}</h4>
          <p class="text-gray-300 text-sm mb-3">{{ group.description || 'Sin descripción' }}</p>
          <p class="text-gray-400 text-xs">Miembro desde: {{ formatTime(group.joined_at) }}</p>
          <!-- Aquí puedes añadir más detalles o acciones del grupo -->
        </div>
      </div>
    </div>

    <CreateGroupModal 
      v-model="showCreateGroupModal"
      @group-created="handleGroupCreated"
    />
  </div>
</template>

<script setup>
import AppHeader from '../components/AppHeader.vue'
import CreateGroupModal from '../components/CreateGroupModal.vue'
import { ArrowLeft, Plus } from 'lucide-vue-next'
import { useRouter } from 'vue-router'
import { ref, onMounted } from 'vue'
import { supabase } from '../supabase'

const router = useRouter()

const showCreateGroupModal = ref(false)
const userGroups = ref([])
const loadingGroups = ref(true)

const goToHome = () => {
  router.push('/')
}

const createGroup = () => {
  showCreateGroupModal.value = true
}

const handleGroupCreated = (newGroup) => {
  console.log('Nuevo grupo creado:', newGroup)
  loadUserGroups()
}

const loadUserGroups = async () => {
  loadingGroups.value = true
  try {
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) {
      console.error('Usuario no autenticado, no se pueden cargar los grupos.')
      loadingGroups.value = false
      return
    }

    const { data: memberData, error: memberError } = await supabase
      .from('group_members')
      .select(`
        group_id,
        joined_at,
        groups (id, name, description, created_by)
      `)
      .eq('user_id', user.id)

    if (memberError) {
      console.error('Error al cargar miembros del grupo:', memberError)
      throw memberError
    }

    userGroups.value = memberData.map(member => ({
      id: member.groups.id,
      name: member.groups.name,
      description: member.groups.description,
      created_by: member.groups.created_by,
      joined_at: member.joined_at,
      role: member.role
    }))
    
  } catch (error) {
    console.error('Error cargando grupos del usuario:', error.message)
  } finally {
    loadingGroups.value = false
  }
}

const formatTime = (timestamp) => {
  const date = new Date(timestamp)
  const options = { year: 'numeric', month: 'long', day: 'numeric' }
  return date.toLocaleDateString('es-ES', options)
}

const goToGroupDetail = (groupId) => {
  router.push({ name: 'group-detail', params: { id: groupId } })
}

onMounted(() => {
  loadUserGroups()
})
</script>

<style scoped>
/* Estilos específicos si son necesarios */
</style> 