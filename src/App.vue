<template>
  <div id="app">
    <NavBar />

    <main class="main-content">
      <router-view />
    </main>

    <Footer />
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import NavBar from './components/NavBar.vue'
import Footer from './components/Footer.vue'
import { useStore } from './composables/useStore'

const store = useStore()

onMounted(() => {
  // Inicializar autenticación al cargar la app
  store.initializeAuth()

  // Cargar productos si no están cargados
  if (store.products.value.length === 0) {
    store.fetchProducts()
  }
})
</script>

<style>
#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  padding-top: 2rem;
}

/* Animaciones globales */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter-from {
  transform: translateX(-100%);
}

.slide-leave-to {
  transform: translateX(100%);
}
</style>