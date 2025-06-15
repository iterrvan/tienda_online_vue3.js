
<template>
  <div class="profile-page">
    <div class="container">
      <div class="profile-header">
        <div class="profile-avatar">
          <img :src="userAvatar" :alt="user?.name" />
          <button @click="changeAvatar" class="avatar-edit-btn">📷</button>
        </div>
        <div class="profile-info">
          <h1>{{ user?.name || 'Usuario' }}</h1>
          <p>{{ user?.email }}</p>
          <span class="user-badge">{{ userRole }}</span>
        </div>
      </div>

      <div class="profile-content">
        <div class="profile-sidebar">
          <nav class="profile-nav">
            <button 
              v-for="tab in profileTabs" 
              :key="tab.id"
              @click="activeTab = tab.id"
              :class="{ active: activeTab === tab.id }"
              class="profile-nav-item"
            >
              <span class="nav-icon">{{ tab.icon }}</span>
              {{ tab.name }}
            </button>
          </nav>
        </div>

        <div class="profile-main">
          <!-- Información Personal -->
          <div v-show="activeTab === 'personal'" class="profile-section">
            <h2>Información Personal</h2>
            <form @submit.prevent="updateProfile" class="profile-form">
              <div class="form-grid">
                <div class="form-group">
                  <label class="form-label">Nombre</label>
                  <input v-model="profileData.name" type="text" class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Email</label>
                  <input v-model="profileData.email" type="email" class="form-control" />
                </div>
              </div>
              
              <div class="form-grid">
                <div class="form-group">
                  <label class="form-label">Teléfono</label>
                  <input v-model="profileData.phone" type="tel" class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Fecha de Nacimiento</label>
                  <input v-model="profileData.birthDate" type="date" class="form-control" />
                </div>
              </div>

              <div class="form-actions">
                <button type="submit" class="btn btn-primary">Guardar Cambios</button>
              </div>
            </form>
          </div>

          <!-- Mis Pedidos -->
          <div v-show="activeTab === 'orders'" class="profile-section">
            <h2>Mis Pedidos</h2>
            <div v-if="orders.length > 0" class="orders-list">
              <div v-for="order in orders" :key="order.id" class="order-card">
                <div class="order-header">
                  <div class="order-info">
                    <strong>Pedido #{{ order.id }}</strong>
                    <span class="order-date">{{ formatDate(order.date) }}</span>
                  </div>
                  <span class="order-status" :class="order.status">
                    {{ getStatusText(order.status) }}
                  </span>
                </div>
                <div class="order-items">
                  <div v-for="item in order.items" :key="item.id" class="order-item">
                    <img :src="item.image" :alt="item.name" />
                    <div class="item-details">
                      <div class="item-name">{{ item.name }}</div>
                      <div class="item-quantity">Cantidad: {{ item.quantity }}</div>
                    </div>
                    <div class="item-price">${{ item.price }}</div>
                  </div>
                </div>
                <div class="order-total">
                  Total: ${{ order.total }}
                </div>
              </div>
            </div>
            <div v-else class="empty-state">
              <p>No tienes pedidos aún</p>
              <router-link to="/products" class="btn btn-primary">Comenzar a Comprar</router-link>
            </div>
          </div>

          <!-- Direcciones -->
          <div v-show="activeTab === 'addresses'" class="profile-section">
            <div class="section-header">
              <h2>Mis Direcciones</h2>
              <button @click="addNewAddress" class="btn btn-primary">+ Nueva Dirección</button>
            </div>
            
            <div class="addresses-grid">
              <div v-for="address in addresses" :key="address.id" class="address-card">
                <div class="address-header">
                  <strong>{{ address.name }}</strong>
                  <div class="address-actions">
                    <button @click="editAddress(address)" class="btn-icon">✏️</button>
                    <button @click="deleteAddress(address.id)" class="btn-icon">🗑️</button>
                  </div>
                </div>
                <div class="address-details">
                  <p>{{ address.street }}</p>
                  <p>{{ address.city }}, {{ address.zipCode }}</p>
                  <p>{{ address.country }}</p>
                </div>
                <div v-if="address.isDefault" class="default-badge">
                  Por defecto
                </div>
              </div>
            </div>
          </div>

          <!-- Seguridad -->
          <div v-show="activeTab === 'security'" class="profile-section">
            <h2>Seguridad</h2>
            
            <div class="security-section">
              <h3>Cambiar Contraseña</h3>
              <form @submit.prevent="changePassword" class="password-form">
                <div class="form-group">
                  <label class="form-label">Contraseña Actual</label>
                  <input v-model="passwordData.current" type="password" class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Nueva Contraseña</label>
                  <input v-model="passwordData.new" type="password" class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Confirmar Nueva Contraseña</label>
                  <input v-model="passwordData.confirm" type="password" class="form-control" />
                </div>
                <button type="submit" class="btn btn-primary">Cambiar Contraseña</button>
              </form>
            </div>

            <div class="security-section">
              <h3>Autenticación de Dos Factores</h3>
              <div class="two-factor">
                <p>Añade una capa extra de seguridad a tu cuenta</p>
                <button class="btn btn-outline">Configurar 2FA</button>
              </div>
            </div>
          </div>

          <!-- Configuración -->
          <div v-show="activeTab === 'settings'" class="profile-section">
            <h2>Configuración</h2>
            
            <div class="settings-group">
              <h3>Notificaciones</h3>
              <div class="setting-item">
                <label class="setting-label">
                  <input type="checkbox" v-model="settings.emailNotifications" />
                  <span>Recibir notificaciones por email</span>
                </label>
              </div>
              <div class="setting-item">
                <label class="setting-label">
                  <input type="checkbox" v-model="settings.smsNotifications" />
                  <span>Recibir notificaciones por SMS</span>
                </label>
              </div>
            </div>

            <div class="settings-group">
              <h3>Privacidad</h3>
              <div class="setting-item">
                <label class="setting-label">
                  <input type="checkbox" v-model="settings.profilePublic" />
                  <span>Perfil público</span>
                </label>
              </div>
            </div>

            <div class="danger-zone">
              <h3>Zona de Peligro</h3>
              <p>Una vez que elimines tu cuenta, no hay vuelta atrás.</p>
              <button @click="deleteAccount" class="btn btn-danger">Eliminar Cuenta</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useStore } from '../composables/useStore'

const store = useStore()

// Estados
const activeTab = ref('personal')
const orders = ref([])
const addresses = ref([])

const profileTabs = [
  { id: 'personal', name: 'Información Personal', icon: '👤' },
  { id: 'orders', name: 'Mis Pedidos', icon: '📦' },
  { id: 'addresses', name: 'Direcciones', icon: '📍' },
  { id: 'security', name: 'Seguridad', icon: '🔒' },
  { id: 'settings', name: 'Configuración', icon: '⚙️' }
]

// Datos reactivos
const profileData = reactive({
  name: '',
  email: '',
  phone: '',
  birthDate: ''
})

const passwordData = reactive({
  current: '',
  new: '',
  confirm: ''
})

const settings = reactive({
  emailNotifications: true,
  smsNotifications: false,
  profilePublic: false
})

// Computed
const user = computed(() => store.user.value)
const userAvatar = computed(() => 
  user.value?.avatar || `https://ui-avatars.com/api/?name=${user.value?.name}&background=3498db&color=fff`
)
const userRole = computed(() => {
  const role = user.value?.role || 'customer'
  return role === 'admin' ? 'Administrador' : 'Cliente'
})

// Métodos
const updateProfile = () => {
  // Aquí iría la lógica para actualizar el perfil
  alert('Perfil actualizado exitosamente')
}

const changePassword = () => {
  if (passwordData.new !== passwordData.confirm) {
    alert('Las contraseñas no coinciden')
    return
  }
  // Aquí iría la lógica para cambiar la contraseña
  alert('Contraseña cambiada exitosamente')
  Object.assign(passwordData, { current: '', new: '', confirm: '' })
}

const changeAvatar = () => {
  // Aquí iría la lógica para cambiar el avatar
  alert('Función de cambiar avatar próximamente...')
}

const addNewAddress = () => {
  // Aquí iría la lógica para agregar nueva dirección
  alert('Agregar nueva dirección próximamente...')
}

const editAddress = (address) => {
  alert(`Editar dirección: ${address.name}`)
}

const deleteAddress = (id) => {
  if (confirm('¿Estás seguro de que quieres eliminar esta dirección?')) {
    addresses.value = addresses.value.filter(addr => addr.id !== id)
  }
}

const deleteAccount = () => {
  if (confirm('¿Estás seguro de que quieres eliminar tu cuenta? Esta acción no se puede deshacer.')) {
    // Aquí iría la lógica para eliminar la cuenta
    alert('Eliminación de cuenta próximamente...')
  }
}

const formatDate = (date) => {
  return new Date(date).toLocaleDateString('es-ES')
}

const getStatusText = (status) => {
  const statuses = {
    pending: 'Pendiente',
    processing: 'Procesando',
    shipped: 'Enviado',
    delivered: 'Entregado',
    cancelled: 'Cancelado'
  }
  return statuses[status] || status
}

// Cargar datos al montar
onMounted(() => {
  // Cargar datos del usuario
  if (user.value) {
    Object.assign(profileData, {
      name: user.value.name || '',
      email: user.value.email || '',
      phone: user.value.phone || '',
      birthDate: user.value.birthDate || ''
    })
  }

  // Datos de ejemplo - reemplazar con llamadas a API real
  orders.value = [
    {
      id: '12345',
      date: '2024-01-15',
      status: 'delivered',
      total: 89.99,
      items: [
        { id: 1, name: 'Producto de ejemplo', quantity: 2, price: 44.99, image: 'https://picsum.photos/60/60?random=1' }
      ]
    }
  ]

  addresses.value = [
    {
      id: 1,
      name: 'Casa',
      street: 'Calle Principal 123',
      city: 'Ciudad',
      zipCode: '12345',
      country: 'País',
      isDefault: true
    }
  ]
})
</script>

<style scoped>
.profile-page {
  padding: 2rem 0;
}

.profile-header {
  display: flex;
  align-items: center;
  gap: 2rem;
  background: white;
  padding: 2rem;
  border-radius: var(--border-radius-lg);
  box-shadow: var(--shadow-sm);
  margin-bottom: 2rem;
}

.profile-avatar {
  position: relative;
}

.profile-avatar img {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  object-fit: cover;
  border: 4px solid var(--primary-color);
}

.avatar-edit-btn {
  position: absolute;
  bottom: 0;
  right: 0;
  background: var(--primary-color);
  border: none;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  cursor: pointer;
  font-size: 1rem;
  color: white;
}

.profile-info h1 {
  margin-bottom: 0.5rem;
  color: var(--text-primary);
}

.profile-info p {
  color: var(--text-secondary);
  margin-bottom: 1rem;
}

.user-badge {
  background: var(--primary-color);
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 15px;
  font-size: 0.9rem;
  font-weight: 500;
}

.profile-content {
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: 2rem;
}

.profile-sidebar {
  background: white;
  border-radius: var(--border-radius-lg);
  box-shadow: var(--shadow-sm);
  padding: 1.5rem;
  height: fit-content;
  position: sticky;
  top: 2rem;
}

.profile-nav {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.profile-nav-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  border: none;
  background: none;
  border-radius: var(--border-radius);
  cursor: pointer;
  transition: all 0.3s ease;
  text-align: left;
  font-size: 1rem;
}

.profile-nav-item:hover {
  background: var(--bg-secondary);
}

.profile-nav-item.active {
  background: var(--primary-color);
  color: white;
}

.nav-icon {
  font-size: 1.2rem;
}

.profile-main {
  background: white;
  border-radius: var(--border-radius-lg);
  box-shadow: var(--shadow-sm);
  padding: 2rem;
}

.profile-section h2 {
  margin-bottom: 2rem;
  color: var(--text-primary);
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
}

.form-actions {
  margin-top: 2rem;
}

.orders-list {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.order-card {
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  padding: 1.5rem;
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.order-date {
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.order-status {
  padding: 0.25rem 0.75rem;
  border-radius: 15px;
  font-size: 0.9rem;
  font-weight: 500;
}

.order-status.pending { background: #fff3cd; color: #856404; }
.order-status.processing { background: #d1ecf1; color: #0c5460; }
.order-status.shipped { background: #d4edda; color: #155724; }
.order-status.delivered { background: #d4edda; color: #155724; }
.order-status.cancelled { background: #f8d7da; color: #721c24; }

.order-items {
  margin-bottom: 1rem;
}

.order-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 0.5rem 0;
}

.order-item img {
  width: 50px;
  height: 50px;
  border-radius: var(--border-radius);
  object-fit: cover;
}

.item-details {
  flex: 1;
}

.item-name {
  font-weight: 500;
}

.item-quantity {
  font-size: 0.9rem;
  color: var(--text-secondary);
}

.item-price {
  font-weight: 600;
  color: var(--primary-color);
}

.order-total {
  text-align: right;
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--primary-color);
  border-top: 1px solid var(--border-color);
  padding-top: 1rem;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.addresses-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}

.address-card {
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius);
  padding: 1.5rem;
  position: relative;
}

.address-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.address-actions {
  display: flex;
  gap: 0.5rem;
}

.btn-icon {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 3px;
}

.btn-icon:hover {
  background: var(--bg-secondary);
}

.default-badge {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: var(--success-color);
  color: white;
  padding: 0.25rem 0.5rem;
  border-radius: 10px;
  font-size: 0.8rem;
}

.security-section {
  margin-bottom: 3rem;
  padding-bottom: 2rem;
  border-bottom: 1px solid var(--border-color);
}

.password-form {
  max-width: 400px;
}

.two-factor p {
  margin-bottom: 1rem;
  color: var(--text-secondary);
}

.settings-group {
  margin-bottom: 2rem;
}

.setting-item {
  margin-bottom: 1rem;
}

.setting-label {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  cursor: pointer;
}

.danger-zone {
  background: #fff5f5;
  border: 1px solid #fed7d7;
  border-radius: var(--border-radius);
  padding: 2rem;
  margin-top: 3rem;
}

.danger-zone h3 {
  color: var(--accent-color);
  margin-bottom: 1rem;
}

.btn-danger {
  background: var(--accent-color);
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: var(--border-radius);
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.btn-danger:hover {
  background: #c0392b;
}

.empty-state {
  text-align: center;
  padding: 3rem;
}

.empty-state p {
  margin-bottom: 2rem;
  color: var(--text-secondary);
}

@media (max-width: 768px) {
  .profile-content {
    grid-template-columns: 1fr;
  }
  
  .profile-header {
    flex-direction: column;
    text-align: center;
  }
  
  .form-grid {
    grid-template-columns: 1fr;
  }
  
  .addresses-grid {
    grid-template-columns: 1fr;
  }
  
  .section-header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
}
</style>
