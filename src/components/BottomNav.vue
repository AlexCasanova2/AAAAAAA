<template>
  <nav class="fixed bottom-0 left-0 right-0 bg-[var(--color-dark-background-lighter)] border-t border-white/10 px-4 py-2 flex justify-around items-center">
    <router-link 
      to="/" 
      class="flex flex-col items-center space-y-1 text-[#fefefe] hover:text-red-500 transition-colors duration-200"
      :class="{ 'text-red-500': $route.path === '/' }"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" />
      </svg>
      <span class="text-xs">Inicio</span>
    </router-link>

    <router-link 
      to="/friends" 
      class="flex flex-col items-center space-y-1 text-[#fefefe] hover:text-red-500 transition-colors duration-200"
      :class="{ 'text-red-500': $route.path === '/friends' }"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" />
      </svg>
      <span class="text-xs">Amigos</span>
    </router-link>

    <router-link 
      to="/groups" 
      class="flex flex-col items-center space-y-1 text-[#fefefe] hover:text-red-500 transition-colors duration-200"
      :class="{ 'text-red-500': $route.path.startsWith('/groups') }"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z" />
      </svg>
      <span class="text-xs">Grupos</span>
    </router-link>

    <router-link 
      to="/profile" 
      class="flex flex-col items-center space-y-1 text-[#fefefe] hover:text-red-500 transition-colors duration-200"
      :class="{ 'text-red-500': $route.path === '/profile' }"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
      </svg>
      <span class="text-xs">Perfil</span>
    </router-link>
  </nav>
</template>

<script setup>
// No necesitamos lógica adicional por ahora
</script>

<style scoped>
.router-link-active {
  color: #ef4444; /* text-red-500 */
}
</style> 