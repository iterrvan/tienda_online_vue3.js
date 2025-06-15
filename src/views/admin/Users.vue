
<template>
  <div class="admin-users">
    <div class="users-header">
      <h1>Gestión de Usuarios</h1>
      <button @click="showCreateModal = true" class="btn btn-primary">
        + Agregar Usuario
      </button>
    </div>

    <!-- Estadísticas -->
    <div class="users-stats">
      <div class="stat-card">
        <div class="stat-icon">👥</div>
        <div class="stat-info">
          <h3>{{ totalUsers }}</h3>
          <p>Total Usuarios</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon">👨‍💼</div>
        <div class="stat-info">
          <h3>{{ adminUsers }}</h3>
          <p>Administradores</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon">🛒</div>
        <div class="stat-info">
          <h3>{{ customerUsers }}</h3>
          <p>Clientes</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon">🟢</div>
        <div class="stat-info">
          <h3>{{ activeUsers }}</h3>
          <p>Activos</p>
        </div>
      </div>
    </div>

    <!-- Filtros -->
    <div class="users-filters">
      <input 
        v-model="searchTerm" 
        type="text" 
        placeholder="Buscar usuarios..."
        class="form-control"
      >
      <select v-model="roleFilter" class="form-control">
        <option value="">Todos los roles</option>
        <option value="admin">Administrador</option>
        <option value="customer">Cliente</option>
      </select>
      <select v-model="statusFilter" class="form-control">
        <option value="">Todos los estados</option>
        <option value="active">Activo</option>
        <option value="inactive">Inactivo</option>
        <option value="banned">Baneado</option>
      </select>
    </div>

    <!-- Tabla de usuarios -->
    <div class="users-table-container">
      <table class="admin-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Usuario</th>
            <th>Email</th>
            <th>Rol</th>
            <th>Estado</th>
            <th>Registro</th>
            <th>Último acceso</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in filteredUsers" :key="user.id">
            <td>#{{ user.id }}</td>
            <td>
              <div class="user-cell">
                <div class="user-avatar">
                  {{ user.name.charAt(0).toUpperCase() }}
                </div>
                <div>
                  <h4>{{ user.name }}</h4>
                  <p>{{ user.username }}</p>
                </div>
              </div>
            </td>
            <td>{{ user.email }}</td>
            <td>
              <span class="role-badge" :class="user.role">
                {{ user.role === 'admin' ? 'Administrador' : 'Cliente' }}
              </span>
            </td>
            <td>
              <span class="status-badge" :class="user.status">
                {{ getStatusLabel(user.status) }}
              </span>
            </td>
            <td>{{ formatDate(user.created_at) }}</td>
            <td>{{ formatDate(user.last_login) }}</td>
            <td>
              <div class="action-buttons">
                <button @click="editUser(user)" class="btn btn-sm btn-secondary">
                  ✏️
                </button>
                <button 
                  @click="toggleUserStatus(user)" 
                  class="btn btn-sm"
                  :class="user.status === 'active' ? 'btn-warning' : 'btn-success'"
                >
                  {{ user.status === 'active' ? '🚫' : '✅' }}
                </button>
                <button @click="deleteUser(user.id)" class="btn btn-sm btn-danger">
                  🗑️
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal para crear/editar usuario -->
    <div v-if="showCreateModal || showEditModal" class="modal-overlay" @click="closeModals">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h2>{{ showCreateModal ? 'Agregar Usuario' : 'Editar Usuario' }}</h2>
          <button @click="closeModals" class="close-btn">&times;</button>
        </div>
        
        <form @submit.prevent="submitUser" class="user-form">
          <div class="form-grid">
            <div class="form-group">
              <label class="form-label">Nombre completo</label>
              <input v-model="userForm.name" type="text" class="form-control" required>
            </div>
            
            <div class="form-group">
              <label class="form-label">Nombre de usuario</label>
              <input v-model="userForm.username" type="text" class="form-control" required>
            </div>
            
            <div class="form-group">
              <label class="form-label">Email</label>
              <input v-model="userForm.email" type="email" class="form-control" required>
            </div>
            
            <div class="form-group">
              <label class="form-label">Teléfono</label>
              <input v-model="userForm.phone" type="tel" class="form-control">
            </div>
          </div>
          
          <div class="form-grid">
            <div class="form-group">
              <label class="form-label">Rol</label>
              <select v-model="userForm.role" class="form-control" required>
                <option value="">Seleccionar rol</option>
                <option value="admin">Administrador</option>
                <option value="customer">Cliente</option>
              </select>
            </div>
            
            <div class="form-group">
              <label class="form-label">Estado</label>
              <select v-model="userForm.status" class="form-control">
                <option value="active">Activo</option>
                <option value="inactive">Inactivo</option>
                <option value="banned">Baneado</option>
              </select>
            </div>
          </div>
          
          <div class="form-group" v-if="showCreateModal">
            <label class="form-label">Contraseña</label>
            <input v-model="userForm.password" type="password" class="form-control" required>
          </div>
          
          <div class="form-group">
            <label class="form-label">Dirección</label>
            <textarea v-model="userForm.address" class="form-control" rows="3"></textarea>
          </div>
          
          <div class="modal-actions">
            <button type="button" @click="closeModals" class="btn btn-secondary">
              Cancelar
            </button>
            <button type="submit" class="btn btn-primary">
              {{ showCreateModal ? 'Crear Usuario' : 'Actualizar Usuario' }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'AdminUsers',
  setup() {
    // Estados reactivos
    const users = ref([])
    const searchTerm = ref('')
    const roleFilter = ref('')
    const statusFilter = ref('')
    const showCreateModal = ref(false)
    const showEditModal = ref(false)
    const editingUser = ref(null)
    
    // Formulario de usuario
    const userForm = ref({
      name: '',
      username: '',
      email: '',
      phone: '',
      role: '',
      status: 'active',
      password: '',
      address: ''
    })

    // Estadísticas computadas
    const totalUsers = computed(() => users.value.length)
    const adminUsers = computed(() => users.value.filter(u => u.role === 'admin').length)
    const customerUsers = computed(() => users.value.filter(u => u.role === 'customer').length)
    const activeUsers = computed(() => users.value.filter(u => u.status === 'active').length)

    // Usuarios filtrados
    const filteredUsers = computed(() => {
      let filtered = users.value

      if (searchTerm.value) {
        const term = searchTerm.value.toLowerCase()
        filtered = filtered.filter(user => 
          user.name.toLowerCase().includes(term) ||
          user.email.toLowerCase().includes(term) ||
          user.username.toLowerCase().includes(term)
        )
      }

      if (roleFilter.value) {
        filtered = filtered.filter(user => user.role === roleFilter.value)
      }

      if (statusFilter.value) {
        filtered = filtered.filter(user => user.status === statusFilter.value)
      }

      return filtered
    })

    // Formatear fecha
    const formatDate = (date) => {
      if (!date) return 'Nunca'
      return new Date(date).toLocaleDateString('es-ES', {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }

    // Obtener etiqueta de estado
    const getStatusLabel = (status) => {
      const labels = {
        active: 'Activo',
        inactive: 'Inactivo',
        banned: 'Baneado'
      }
      return labels[status] || status
    }

    // Cargar usuarios
    const loadUsers = async () => {
      try {
        // Aquí harías la llamada a tu API Laravel
        // const response = await fetch('/api/admin/users')
        
        // Datos de ejemplo
        users.value = [
          {
            id: 1,
            name: 'Admin Principal',
            username: 'admin',
            email: 'admin@tutienda.com',
            phone: '+34 600 000 000',
            role: 'admin',
            status: 'active',
            created_at: '2023-01-01T00:00:00Z',
            last_login: '2024-01-15T10:30:00Z',
            address: 'Oficina Principal, Madrid'
          },
          {
            id: 2,
            name: 'Juan Pérez',
            username: 'juanperez',
            email: 'juan@ejemplo.com',
            phone: '+34 666 777 888',
            role: 'customer',
            status: 'active',
            created_at: '2024-01-10T15:20:00Z',
            last_login: '2024-01-15T09:45:00Z',
            address: 'Calle Principal 123, Madrid'
          },
          {
            id: 3,
            name: 'María García',
            username: 'mariagarcia',
            email: 'maria@ejemplo.com',
            phone: '+34 555 666 777',
            role: 'customer',
            status: 'inactive',
            created_at: '2024-01-05T12:10:00Z',
            last_login: '2024-01-12T18:30:00Z',
            address: 'Avenida Central 456, Barcelona'
          },
          {
            id: 4,
            name: 'Carlos López',
            username: 'carloslopez',
            email: 'carlos@ejemplo.com',
            phone: '+34 777 888 999',
            role: 'customer',
            status: 'banned',
            created_at: '2023-12-20T08:00:00Z',
            last_login: '2024-01-08T14:15:00Z',
            address: 'Plaza Mayor 789, Valencia'
          }
        ]
      } catch (error) {
        console.error('Error cargando usuarios:', error)
      }
    }

    // Editar usuario
    const editUser = (user) => {
      editingUser.value = user
      userForm.value = { ...user }
      delete userForm.value.password // No mostrar contraseña
      showEditModal.value = true
    }

    // Cambiar estado del usuario
    const toggleUserStatus = async (user) => {
      try {
        const newStatus = user.status === 'active' ? 'inactive' : 'active'
        user.status = newStatus
        
        // Aquí harías la llamada a tu API Laravel
        // await fetch(`/api/admin/users/${user.id}`, {
        //   method: 'PATCH',
        //   body: JSON.stringify({ status: newStatus })
        // })
        
        alert(`Usuario ${newStatus === 'active' ? 'activado' : 'desactivado'} correctamente`)
      } catch (error) {
        console.error('Error cambiando estado del usuario:', error)
      }
    }

    // Eliminar usuario
    const deleteUser = async (userId) => {
      if (confirm('¿Estás seguro de que quieres eliminar este usuario?')) {
        try {
          // await fetch(`/api/admin/users/${userId}`, { method: 'DELETE' })
          users.value = users.value.filter(u => u.id !== userId)
          alert('Usuario eliminado correctamente')
        } catch (error) {
          console.error('Error eliminando usuario:', error)
          alert('Error al eliminar el usuario')
        }
      }
    }

    // Enviar formulario
    const submitUser = async () => {
      try {
        if (showCreateModal.value) {
          // Crear nuevo usuario
          const newUser = {
            ...userForm.value,
            id: Date.now(), // ID temporal
            created_at: new Date().toISOString(),
            last_login: null
          }
          users.value.push(newUser)
          alert('Usuario creado correctamente')
        } else {
          // Actualizar usuario existente
          const index = users.value.findIndex(u => u.id === editingUser.value.id)
          if (index !== -1) {
            users.value[index] = {
              ...users.value[index],
              ...userForm.value
            }
          }
          alert('Usuario actualizado correctamente')
        }
        
        closeModals()
      } catch (error) {
        console.error('Error guardando usuario:', error)
        alert('Error al guardar el usuario')
      }
    }

    // Cerrar modales
    const closeModals = () => {
      showCreateModal.value = false
      showEditModal.value = false
      editingUser.value = null
      userForm.value = {
        name: '',
        username: '',
        email: '',
        phone: '',
        role: '',
        status: 'active',
        password: '',
        address: ''
      }
    }

    onMounted(() => {
      loadUsers()
    })

    return {
      users,
      searchTerm,
      roleFilter,
      statusFilter,
      showCreateModal,
      showEditModal,
      userForm,
      totalUsers,
      adminUsers,
      customerUsers,
      activeUsers,
      filteredUsers,
      formatDate,
      getStatusLabel,
      editUser,
      toggleUserStatus,
      deleteUser,
      submitUser,
      closeModals
    }
  }
}
</script>

<style scoped>
.admin-users {
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.users-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.users-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.users-filters {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.users-table-container {
  background: white;
  border-radius: 12px;
  box-shadow: var(--shadow-sm);
  overflow: hidden;
}

.user-cell {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: var(--primary-color);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 1.2rem;
}

.user-cell h4 {
  margin: 0 0 0.25rem 0;
  font-size: 0.95rem;
}

.user-cell p {
  margin: 0;
  color: var(--text-secondary);
  font-size: 0.8rem;
}

.role-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
}

.role-badge.admin {
  background: #e2e3e5;
  color: #383d41;
}

.role-badge.customer {
  background: #cce5ff;
  color: #004085;
}

.user-form {
  padding: 1.5rem;
}

@media (max-width: 768px) {
  .admin-users {
    padding: 1rem;
  }
  
  .users-header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .users-filters {
    flex-direction: column;
  }
  
  .users-stats {
    grid-template-columns: 1fr;
  }
}
</style>
