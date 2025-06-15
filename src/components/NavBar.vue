
<template>
  <nav class="navbar">
    <div class="container">
      <div class="nav-content">
        <!-- Logo -->
        <router-link to="/" class="nav-brand">
          <div class="logo">
            <span class="logo-icon">🛍️</span>
            <span class="logo-text">TuTienda</span>
          </div>
        </router-link>

        <!-- Menu Principal -->
        <div class="nav-menu" :class="{ 'nav-menu-open': isMobileMenuOpen }">
          <router-link to="/" class="nav-link" @click="closeMobileMenu">
            Inicio
          </router-link>
          <router-link to="/products" class="nav-link" @click="closeMobileMenu">
            Productos
          </router-link>
          
          <!-- Dropdown de Categorías -->
          <div class="nav-dropdown" @mouseenter="showCategories = true" @mouseleave="showCategories = false">
            <button class="nav-link dropdown-toggle">
              Categorías
              <span class="dropdown-arrow">▼</span>
            </button>
            <div class="dropdown-menu" v-show="showCategories">
              <router-link to="/products?category=electronics" class="dropdown-item">
                📱 Electrónicos
              </router-link>
              <router-link to="/products?category=clothing" class="dropdown-item">
                👕 Ropa
              </router-link>
              <router-link to="/products?category=home" class="dropdown-item">
                🏠 Hogar
              </router-link>
              <router-link to="/products?category=sports" class="dropdown-item">
                ⚽ Deportes
              </router-link>
            </div>
          </div>
        </div>

        <!-- Acciones de Usuario -->
        <div class="nav-actions">
          <!-- Buscador -->
          <div class="search-container" :class="{ 'search-open': isSearchOpen }">
            <input 
              v-model="searchQuery"
              type="text" 
              placeholder="Buscar productos..."
              class="search-input"
              @keyup.enter="performSearch"
            />
            <button @click="toggleSearch" class="search-btn">
              🔍
            </button>
          </div>

          <!-- Carrito -->
          <router-link to="/cart" class="nav-action cart-btn">
            <span class="cart-icon">🛒</span>
            <span v-if="cartItemCount > 0" class="cart-badge">{{ cartItemCount }}</span>
          </router-link>

          <!-- Usuario -->
          <div class="user-menu" v-if="user">
            <div class="user-dropdown" @click="toggleUserMenu">
              <img :src="user.avatar || defaultAvatar" :alt="user.name" class="user-avatar" />
              <span class="user-name">{{ user.name }}</span>
              <span class="dropdown-arrow">▼</span>
            </div>
            <div class="dropdown-menu user-dropdown-menu" v-show="showUserMenu">
              <router-link to="/profile" class="dropdown-item">
                👤 Mi Perfil
              </router-link>
              <router-link to="/orders" class="dropdown-item">
                📦 Mis Pedidos
              </router-link>
              <div class="dropdown-divider"></div>
              <button @click="handleLogout" class="dropdown-item logout-btn">
                🚪 Cerrar Sesión
              </button>
            </div>
          </div>

          <!-- Botones de Login/Registro -->
          <div v-else class="auth-buttons">
            <router-link to="/login" class="btn btn-outline btn-sm">
              Iniciar Sesión
            </router-link>
            <router-link to="/register" class="btn btn-primary btn-sm">
              Registrarse
            </router-link>
          </div>

          <!-- Toggle Mobile Menu -->
          <button @click="toggleMobileMenu" class="mobile-menu-btn">
            <span></span>
            <span></span>
            <span></span>
          </button>
        </div>
      </div>
    </div>
  </nav>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from '../composables/useStore'

const router = useRouter()
const store = useStore()

// Estados locales
const isMobileMenuOpen = ref(false)
const isSearchOpen = ref(false)
const showCategories = ref(false)
const showUserMenu = ref(false)
const searchQuery = ref('')

// Computed properties
const user = computed(() => store.user.value)
const cartItemCount = computed(() => store.cartItemCount.value)
const defaultAvatar = 'https://ui-avatars.com/api/?name=Usuario&background=3498db&color=fff'

// Métodos
const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value
}

const closeMobileMenu = () => {
  isMobileMenuOpen.value = false
}

const toggleSearch = () => {
  isSearchOpen.value = !isSearchOpen.value
  if (isSearchOpen.value) {
    performSearch()
  }
}

const performSearch = () => {
  if (searchQuery.value.trim()) {
    router.push(`/products?search=${encodeURIComponent(searchQuery.value)}`)
    searchQuery.value = ''
    isSearchOpen.value = false
  }
}

const toggleUserMenu = () => {
  showUserMenu.value = !showUserMenu.value
}

const handleLogout = () => {
  store.logout()
  showUserMenu.value = false
  router.push('/')
}
</script>

<style scoped>
.navbar {
  background: white;
  box-shadow: var(--shadow-sm);
  position: sticky;
  top: 0;
  z-index: 1000;
  border-bottom: 1px solid var(--border-color);
}

.nav-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 0;
}

/* === LOGO === */
.nav-brand {
  text-decoration: none;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.logo-icon {
  font-size: 2rem;
}

.logo-text {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--primary-color);
}

/* === MENU PRINCIPAL === */
.nav-menu {
  display: flex;
  align-items: center;
  gap: 2rem;
}

.nav-link {
  color: var(--text-primary);
  font-weight: 500;
  padding: 0.5rem 0;
  position: relative;
  transition: all 0.3s ease;
}

.nav-link:hover {
  color: var(--primary-color);
  text-decoration: none;
}

.nav-link.router-link-active {
  color: var(--primary-color);
}

.nav-link.router-link-active::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 0;
  right: 0;
  height: 2px;
  background: var(--primary-color);
}

/* === DROPDOWN === */
.nav-dropdown {
  position: relative;
}

.dropdown-toggle {
  background: none;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.dropdown-arrow {
  font-size: 0.8rem;
  transition: transform 0.3s ease;
}

.nav-dropdown:hover .dropdown-arrow {
  transform: rotate(180deg);
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  left: 0;
  background: white;
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow-lg);
  min-width: 200px;
  z-index: 1001;
  padding: 0.5rem 0;
}

.dropdown-item {
  display: block;
  padding: 0.75rem 1rem;
  color: var(--text-primary);
  text-decoration: none;
  transition: background-color 0.3s ease;
}

.dropdown-item:hover {
  background: var(--bg-secondary);
  color: var(--primary-color);
  text-decoration: none;
}

.dropdown-divider {
  height: 1px;
  background: var(--border-color);
  margin: 0.5rem 0;
}

/* === ACCIONES === */
.nav-actions {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.search-container {
  position: relative;
  display: flex;
  align-items: center;
}

.search-input {
  padding: 0.5rem 1rem;
  border: 2px solid var(--border-color);
  border-radius: 25px;
  width: 0;
  opacity: 0;
  transition: all 0.3s ease;
}

.search-container.search-open .search-input {
  width: 250px;
  opacity: 1;
}

.search-btn {
  background: none;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 50%;
  transition: background-color 0.3s ease;
}

.search-btn:hover {
  background: var(--bg-secondary);
}

/* === CARRITO === */
.nav-action {
  position: relative;
  padding: 0.5rem;
  border-radius: var(--border-radius);
  transition: background-color 0.3s ease;
}

.nav-action:hover {
  background: var(--bg-secondary);
  text-decoration: none;
}

.cart-icon {
  font-size: 1.5rem;
}

.cart-badge {
  position: absolute;
  top: 0;
  right: 0;
  background: var(--accent-color);
  color: white;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  font-size: 0.8rem;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
}

/* === USUARIO === */
.user-dropdown {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: var(--border-radius);
  transition: background-color 0.3s ease;
}

.user-dropdown:hover {
  background: var(--bg-secondary);
}

.user-avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  object-fit: cover;
}

.user-name {
  font-weight: 500;
  color: var(--text-primary);
}

.user-dropdown-menu {
  right: 0;
  left: auto;
}

.logout-btn {
  width: 100%;
  text-align: left;
  background: none;
  border: none;
  cursor: pointer;
}

/* === AUTH BUTTONS === */
.auth-buttons {
  display: flex;
  gap: 0.5rem;
}

.btn-sm {
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
}

/* === MOBILE MENU === */
.mobile-menu-btn {
  display: none;
  flex-direction: column;
  gap: 4px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.5rem;
}

.mobile-menu-btn span {
  width: 25px;
  height: 3px;
  background: var(--text-primary);
  transition: all 0.3s ease;
}

/* === RESPONSIVE === */
@media (max-width: 768px) {
  .nav-menu {
    position: fixed;
    top: 70px;
    left: 0;
    right: 0;
    background: white;
    flex-direction: column;
    padding: 2rem;
    box-shadow: var(--shadow-lg);
    transform: translateX(-100%);
    transition: transform 0.3s ease;
  }

  .nav-menu-open {
    transform: translateX(0);
  }

  .mobile-menu-btn {
    display: flex;
  }

  .search-container.search-open .search-input {
    width: 200px;
  }

  .auth-buttons {
    flex-direction: column;
  }

  .user-name {
    display: none;
  }
}
</style>
