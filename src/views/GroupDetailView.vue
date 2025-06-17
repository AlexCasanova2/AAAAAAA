<template>
  <div class="h-[100dvh] bg-[var(--color-dark-background)] text-white flex flex-col">
    <AppHeader :title="group ? group.name : 'Grupo'" :dark-mode="true">
      <template #left-content>
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Volver a Grupos"
          @click="goToGroups"
        >
          <ArrowLeft class="w-6 h-6 stroke-white" />
        </button>
      </template>
      <!-- Aquí podríamos añadir un botón para opciones del grupo si el usuario es admin -->
       
    </AppHeader>

    <div class="flex-1 p-4" :style="{ paddingTop: `calc(4rem + env(safe-area-inset-top))` }">
      <div v-if="loadingGroup || loadingMembers" class="flex justify-center items-center h-full">
        <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-white"></div>
      </div>
      <div v-else-if="!group" class="flex justify-center items-center h-full">
        <p class="text-white/70">Grupo no encontrado.</p>
      </div>
      <div v-else class="space-y-4">
        <div class="flex justify-between items-center mb-6">
          
          <div class="flex items-center space-x-3">
            <button 
              @click="showInviteModal = true"
              class="bg-red-500 text-white px-4 py-2 rounded-lg hover:bg-red-600 transition-colors duration-200 flex items-center space-x-2"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path d="M8 9a3 3 0 100-6 3 3 0 000 6zM8 11a6 6 0 016 6H2a6 6 0 016-6zM16 7a1 1 0 10-2 0v1h-1a1 1 0 100 2h1v1a1 1 0 102 0v-1h1a1 1 0 100-2h-1V7z" />
              </svg>
              <span>Invitar</span>
            </button>
            
            <div class="relative" v-if="isAdmin">
              <button 
                @click="showAdminMenu = !showAdminMenu"
                class="bg-[var(--color-dark-background-light)] text-[#fefefe] p-2 rounded-lg hover:bg-[var(--color-dark-background-lighter)] transition-colors duration-200"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 5v.01M12 12v.01M12 19v.01M12 6a1 1 0 110-2 1 1 0 010 2zm0 7a1 1 0 110-2 1 1 0 010 2zm0 7a1 1 0 110-2 1 1 0 010 2z" />
                </svg>
              </button>
              
              <!-- Menú desplegable -->
              <div 
                v-if="showAdminMenu"
                class="absolute right-0 mt-2 w-48 bg-[var(--color-dark-background-lighter)] rounded-lg shadow-lg border border-white/10 py-1 z-10"
              >
                <button 
                  @click="showEditGroupModal = true; showAdminMenu = false"
                  class="w-full px-4 py-2 text-left text-[#fefefe] hover:bg-[var(--color-dark-background-light)] transition-colors duration-200 flex items-center space-x-2"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                    <path d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 5.793L3 14.172V17h2.828l8.38-8.379-2.83-2.828z" />
                  </svg>
                  <span>Editar grupo</span>
                </button>
                <button 
                  @click="showDeleteGroupModal = true; showAdminMenu = false"
                  class="w-full px-4 py-2 text-left text-red-500 hover:bg-[var(--color-dark-background-light)] transition-colors duration-200 flex items-center space-x-2"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
                  </svg>
                  <span>Eliminar grupo</span>
                </button>
              </div>
            </div>
          </div>
        </div>
        <h1 class="text-2xl font-bold text-[#fefefe]">{{ group?.name }}</h1>
        <p class="text-gray-300 mb-4">{{ group.description || 'Sin descripción.' }}</p>
        
        <h3 class="text-xl font-semibold mb-3">Miembros del Grupo</h3>
        <div v-if="groupMembers.length > 0">
          <ul class="space-y-2">
            <li v-for="member in groupMembers" :key="member.id" class="bg-gray-800/50 rounded-lg p-3 flex items-center justify-between">
              <span class="font-medium">{{ member.user_email }}</span>
              <span class="text-gray-400 text-sm">{{ member.role }}</span>
            </li>
          </ul>
        </div>
        <p v-else class="text-gray-400">No hay miembros en este grupo (aparte de ti).</p>

        <!-- Opciones del grupo -->
        <div class="mt-8 space-y-4">
          <h3 class="text-xl font-semibold mb-3">Opciones del Grupo</h3>
          <button 
            class="w-full bg-red-500 text-white py-3 rounded-lg font-medium hover:bg-red-600 transition-colors duration-200"
            @click="showLeaveGroupConfirm = true"
          >
            Abandonar Grupo
          </button>
          <!-- Botón para invitar amigos, visible solo si el usuario es admin -->
          <button 
            v-if="isCurrentUserAdmin"
            class="w-full bg-blue-500 text-white py-3 rounded-lg font-medium hover:bg-blue-600 transition-colors duration-200"
            @click="openInviteModal"
          >
            Invitar Amigos
          </button>
        </div>

      </div>
    </div>

    <!-- Modal de confirmación para abandonar grupo -->
    <ConfirmModal
      v-model="showLeaveGroupConfirm"
      title="Confirmar Abandono de Grupo"
      message="¿Estás seguro de que quieres abandonar este grupo? Esta acción no se puede deshacer y perderás el acceso a su contenido a menos que te inviten de nuevo."
      confirm-text="Sí, abandonar"
      cancel-text="No, quedarme"
      reasons-title="¿Por qué abandonas este grupo?"
      :reasons="[
        { value: 'Ya no es relevante para mí', label: 'Ya no es relevante para mí' },
        { value: 'Demasiadas notificaciones', label: 'Demasiadas notificaciones' },
        { value: 'No hay actividad', label: 'No hay actividad' },
        { value: 'Contenido inapropiado', label: 'Contenido inapropiado' },
        { value: 'Otros', label: 'Otros' }
      ]"
      @confirm="leaveGroup"
    />

    <!-- Modales -->
    <InviteFriendsModal
      v-model="showInviteModal"
      :groupId="group?.id"
      @inviteSent="loadGroupMembers"
    />

    <EditGroupModal
      v-model="showEditGroupModal"
      :group="group"
      @groupUpdated="handleGroupUpdated"
    />

    <ConfirmModal
      v-model="showDeleteGroupModal"
      title="Eliminar grupo"
      message="¿Estás seguro de que quieres eliminar este grupo? Esta acción no se puede deshacer."
      confirmText="Sí, eliminar"
      cancelText="Cancelar"
      @confirm="handleDeleteGroup"
    />
  </div>
</template>

<script setup>
import AppHeader from '../components/AppHeader.vue'
import ConfirmModal from '../components/ConfirmModal.vue'
import { ArrowLeft } from 'lucide-vue-next'
import { useRouter, useRoute } from 'vue-router'
import { ref, onMounted } from 'vue'
import { supabase } from '../supabase'
import InviteFriendsModal from '../components/InviteFriendsModal.vue'
import EditGroupModal from '../components/EditGroupModal.vue'

const router = useRouter()
const route = useRoute()

const groupId = ref(route.params.id)
const group = ref(null)
const groupMembers = ref([])
const loadingGroup = ref(true)
const loadingMembers = ref(true)
const isCurrentUserAdmin = ref(false)
const showLeaveGroupConfirm = ref(false)
const showInviteModal = ref(false)
const showEditGroupModal = ref(false)
const showDeleteGroupModal = ref(false)

const goToGroups = () => {
  router.push('/groups')
}

const loadGroupDetails = async () => {
  loadingGroup.value = true
  try {
    const { data, error } = await supabase
      .from('groups')
      .select('*, created_by')
      .eq('id', groupId.value)
      .single()

    if (error) throw error
    group.value = data
  } catch (error) {
    console.error('Error cargando detalles del grupo:', error.message)
    group.value = null
  } finally {
    loadingGroup.value = false
  }
}

const loadGroupMembers = async () => {
  loadingMembers.value = true
  try {
    const { data: { user: currentUser } } = await supabase.auth.getUser()
    if (!currentUser) {
      throw new Error('Usuario no autenticado')
    }

    const { data, error } = await supabase
      .from('group_members')
      .select(`
        id,
        role,
        user_id(email) 
      `)
      .eq('group_id', groupId.value)

    if (error) throw error
    
    groupMembers.value = data.map(member => ({
      id: member.id,
      role: member.role,
      user_email: member.user_id.email
    }))

    // Verificar si el usuario actual es administrador
    isCurrentUserAdmin.value = groupMembers.value.some(
      (member) => member.user_id.email === currentUser.email && member.role === 'admin'
    )

  } catch (error) {
    console.error('Error cargando miembros del grupo:', error.message)
  } finally {
    loadingMembers.value = false
  }
}

const leaveGroup = async (reasonData) => {
  try {
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) {
      throw new Error('Usuario no autenticado')
    }

    // Guardar el motivo de abandono del grupo
    const { error: logError } = await supabase
      .from('group_abandonment_logs')
      .insert({
        user_id: user.id,
        group_id: groupId.value,
        reason_selected: reasonData.selected,
        other_reason: reasonData.otherText || null,
        abandoned_at: new Date().toISOString()
      })

    if (logError) {
      console.error('Error al registrar el abandono del grupo:', logError.message)
      // Aunque haya un error en el log, intentar seguir con la eliminación de la membresía
    }

    const { error } = await supabase
      .from('group_members')
      .delete()
      .eq('group_id', groupId.value)
      .eq('user_id', user.id)

    if (error) {
      console.error('Error al abandonar el grupo:', error.message)
      throw error
    }

    // Eliminar el alert de confirmación
    // alert('Has abandonado el grupo exitosamente.')
    router.push('/groups') // Redirigir a la vista de grupos
  } catch (error) {
    console.error('Error en el proceso de abandono de grupo:', error.message)
    alert(`No se pudo abandonar el grupo: ${error.message}`)
  } finally {
    showLeaveGroupConfirm.value = false // Cerrar el modal de confirmación
  }
}

const openInviteModal = () => {
  console.log('Abrir modal para invitar amigos')
  // Aquí iría la lógica para abrir un modal de invitación
}

const handleGroupUpdated = (updatedGroup) => {
  group.value = updatedGroup
}

const handleDeleteGroup = async () => {
  try {
    const { error } = await supabase
      .from('groups')
      .delete()
      .eq('id', group.value.id)

    if (error) throw error

    router.push('/groups')
  } catch (error) {
    console.error('Error al eliminar el grupo:', error)
    alert('Error al eliminar el grupo')
  }
}

onMounted(() => {
  if (groupId.value) {
    loadGroupDetails()
    loadGroupMembers()
  }
})
</script>

<style scoped>
/* Estilos específicos si son necesarios */
</style> 