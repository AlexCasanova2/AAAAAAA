<template>
  <div class="flex flex-col min-h-screen bg-[var(--color-dark-background)] text-[#fefefe]">
    <AppHeader v-if="showAppHeader" :title="headerTitle" :dark-mode="isHeaderDarkMode">
      <template #left-content>
        <component :is="headerLeftContent" />
      </template>
      <template #right-content>
        <component :is="headerRightContent" />
      </template>
    </AppHeader>

    <main class="flex-1 overflow-auto"
      :style="{
        paddingTop: showAppHeader ? `calc(72px + env(safe-area-inset-top))` : `0px`,
        paddingBottom: showBottomNav ? `40px` : `0px`
      }">
      <router-view v-slot="{ Component }">
        <component :is="Component" />
      </router-view>
    </main>

    <BottomNav v-if="showBottomNav" />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, shallowRef } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { supabase } from './supabase'
import { startTracking, stopTracking } from './utils/amplitude'
import BottomNav from './components/BottomNav.vue'
import AppHeader from './components/AppHeader.vue'

const router = useRouter()
const route = useRoute()

// Propiedades reactivas para el AppHeader global
const showAppHeader = ref(true)
const headerTitle = ref('')
const isHeaderDarkMode = ref(false)
const headerLeftContent = shallowRef(null)
const headerRightContent = shallowRef(null)

// Observar la ruta para controlar la visibilidad y props del header/bottom nav
router.beforeEach((to, from, next) => {
  // Controlar la visibilidad de la barra de navegación inferior
  showBottomNav.value = to.name !== 'login'

  // Reiniciar propiedades del header para cada ruta
  showAppHeader.value = true;
  headerTitle.value = '';
  isHeaderDarkMode.value = false;
  headerLeftContent.value = null;
  headerRightContent.value = null;

  // Lógica específica para cada ruta
  if (to.name === 'home') {
    // HomeView tiene un header blanco, y necesita los botones de Perfil y Grupos/Historial
    isHeaderDarkMode.value = false;
    headerLeftContent.value = {
      template: `
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Perfil"
          @click="$router.push('/profile')"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-[#111]" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" /></svg>
        </button>
      `,
      setup() { return {}} // Necesario para Vue 3 con template string
    };
    headerRightContent.value = {
      template: `
        <div class="flex items-center">
          <button 
            class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
            aria-label="Grupos"
            @click="$router.push('/groups')"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-[#111]" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z" /></svg>
          </button>
          <button 
            class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity ml-2" 
            aria-label="Historial"
            @click="$router.push('/history')"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-[#111]" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z" /></svg>
          </button>
        </div>
      `,
      setup() { return {$router: router} } // Pasar router para que sea accesible en el template
    };
  } else if (to.name === 'history') {
    // HistoryView tiene un header oscuro y un botón de "volver"
    headerTitle.value = 'Historial';
    isHeaderDarkMode.value = true;
    headerRightContent.value = {
      template: `
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Volver"
          @click="$router.push('/')"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" /></svg>
        </button>
      `,
      setup() { return {$router: router} }
    };
  } else if (to.name === 'profile') {
    headerTitle.value = 'Perfil';
    isHeaderDarkMode.value = true;
    headerRightContent.value = {
      template: `
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Volver"
          @click="$router.push('/')"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" /></svg>
        </button>
      `,
      setup() { return {$router: router} }
    };
  } else if (to.name === 'groups') {
    headerTitle.value = 'Grupos';
    isHeaderDarkMode.value = true;
    headerRightContent.value = {
      template: `
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Volver"
          @click="$router.push('/')"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" /></svg>
        </button>
      `,
      setup() { return {$router: router} }
    };
  } else if (to.name === 'group-detail') {
    headerTitle.value = 'Detalle del Grupo';
    isHeaderDarkMode.value = true;
    headerRightContent.value = {
      template: `
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Volver"
          @click="$router.push('/groups')"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" /></svg>
        </button>
      `,
      setup() { return {$router: router} }
    };
  } else if (to.name === 'friends') {
    headerTitle.value = 'Amigos';
    isHeaderDarkMode.value = true;
    headerRightContent.value = {
      template: `
        <button 
          class="p-1 flex items-center justify-center hover:opacity-80 transition-opacity" 
          aria-label="Volver"
          @click="$router.push('/')"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 stroke-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" /></svg>
        </button>
      `,
      setup() { return {$router: router} }
    };
  } else if (to.name === 'login') {
    showAppHeader.value = false;
  }
  next();
});

onMounted(() => {
  // Escuchar cambios en el estado de autenticación
  supabase.auth.onAuthStateChange((event, session) => {
    if (event === 'SIGNED_OUT') {
      // Redirigir al login si el usuario cierra sesión
      window.location.href = '/login'
    }
  })
  startTracking()
})

onUnmounted(() => {
  stopTracking()
})

// No mostrar la barra de navegación en la página de login
const showBottomNav = ref(true) // Inicialmente visible

</script>

<style>
/* Estilos generales para el layout de App.vue */
body {
  margin: 0;
  padding: 0;
  overflow: hidden; /* Controlado globalmente para el layout principal */
}

#app {
  min-height: 100vh;
  min-height: 100dvh;
}

/* Eliminar estilos de transición que puedan interferir */
/* Si hay transiciones push-left/right, quitarlas o adaptarlas */
</style> 