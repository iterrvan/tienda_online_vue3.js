
<template>
  <div class="admin-dashboard">
    <!-- Header del Dashboard -->
    <div class="dashboard-header">
      <h1>Panel de Administración</h1>
      <p>Bienvenido al dashboard administrativo</p>
    </div>

    <!-- Estadísticas generales -->
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-icon">📦</div>
        <div class="stat-info">
          <h3>{{ totalProducts }}</h3>
          <p>Total Productos</p>
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-icon">🛒</div>
        <div class="stat-info">
          <h3>{{ totalOrders }}</h3>
          <p>Pedidos Totales</p>
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-icon">👥</div>
        <div class="stat-info">
          <h3>{{ totalUsers }}</h3>
          <p>Usuarios Registrados</p>
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-icon">💰</div>
        <div class="stat-info">
          <h3>${{ totalRevenue }}</h3>
          <p>Ingresos Totales</p>
        </div>
      </div>
    </div>

    <!-- Gráficos y tablas -->
    <div class="dashboard-content">
      <div class="dashboard-section">
        <h2>Ventas Recientes</h2>
        <div class="table-container">
          <table class="admin-table">
            <thead>
              <tr>
                <th>ID</th>
                <th>Cliente</th>
                <th>Total</th>
                <th>Estado</th>
                <th>Fecha</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="order in recentOrders" :key="order.id">
                <td>#{{ order.id }}</td>
                <td>{{ order.customer_name }}</td>
                <td>${{ order.total }}</td>
                <td>
                  <span class="status-badge" :class="order.status">
                    {{ order.status }}
                  </span>
                </td>
                <td>{{ formatDate(order.created_at) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div class="dashboard-section">
        <h2>Productos Más Vendidos</h2>
        <div class="product-list">
          <div v-for="product in topProducts" :key="product.id" class="product-item">
            <img :src="product.image" :alt="product.name" class="product-thumb">
            <div class="product-info">
              <h4>{{ product.name }}</h4>
              <p>{{ product.sales_count }} vendidos</p>
            </div>
            <div class="product-price">${{ product.price }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import { useStore } from '../../composables/useStore'

export default {
  name: 'AdminDashboard',
  setup() {
    const store = useStore()
    
    // Estados reactivos para las estadísticas
    const totalProducts = ref(0)
    const totalOrders = ref(0)
    const totalUsers = ref(0)
    const totalRevenue = ref(0)
    const recentOrders = ref([])
    const topProducts = ref([])

    // Función para formatear fechas
    const formatDate = (date) => {
      return new Date(date).toLocaleDateString('es-ES', {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      })
    }

    // Cargar datos del dashboard
    const loadDashboardData = async () => {
      try {
        // Aquí harías las llamadas a tu API Laravel
        // Por ahora usamos datos de ejemplo
        
        totalProducts.value = 156
        totalOrders.value = 1247
        totalUsers.value = 892
        totalRevenue.value = 45678.90

        // Pedidos recientes (datos de ejemplo)
        recentOrders.value = [
          {
            id: 1001,
            customer_name: 'Juan Pérez',
            total: 89.99,
            status: 'completado',
            created_at: '2024-01-15T10:30:00Z'
          },
          {
            id: 1002,
            customer_name: 'María García',
            total: 156.50,
            status: 'pendiente',
            created_at: '2024-01-15T09:15:00Z'
          },
          {
            id: 1003,
            customer_name: 'Carlos López',
            total: 234.75,
            status: 'enviado',
            created_at: '2024-01-14T16:45:00Z'
          }
        ]

        // Productos más vendidos (datos de ejemplo)
        topProducts.value = [
          {
            id: 1,
            name: 'Smartphone Pro',
            price: 699.99,
            sales_count: 45,
            image: 'https://via.placeholder.com/60x60'
          },
          {
            id: 2,
            name: 'Laptop Gaming',
            price: 1299.99,
            sales_count: 32,
            image: 'https://via.placeholder.com/60x60'
          },
          {
            id: 3,
            name: 'Auriculares Bluetooth',
            price: 199.99,
            sales_count: 78,
            image: 'https://via.placeholder.com/60x60'
          }
        ]

      } catch (error) {
        console.error('Error cargando datos del dashboard:', error)
      }
    }

    onMounted(() => {
      loadDashboardData()
    })

    return {
      totalProducts,
      totalOrders,
      totalUsers,
      totalRevenue,
      recentOrders,
      topProducts,
      formatDate
    }
  }
}
</script>

<style scoped>
.admin-dashboard {
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

.dashboard-header {
  margin-bottom: 2rem;
}

.dashboard-header h1 {
  color: var(--primary-color);
  margin-bottom: 0.5rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.stat-card {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: var(--shadow-sm);
  display: flex;
  align-items: center;
  gap: 1rem;
  transition: transform 0.2s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

.stat-icon {
  font-size: 2.5rem;
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  border-radius: 50%;
}

.stat-info h3 {
  font-size: 2rem;
  font-weight: bold;
  margin: 0;
  color: var(--text-primary);
}

.stat-info p {
  margin: 0;
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.dashboard-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 2rem;
}

.dashboard-section {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: var(--shadow-sm);
}

.dashboard-section h2 {
  margin-top: 0;
  margin-bottom: 1.5rem;
  color: var(--text-primary);
}

.table-container {
  overflow-x: auto;
}

.admin-table {
  width: 100%;
  border-collapse: collapse;
}

.admin-table th,
.admin-table td {
  padding: 0.75rem;
  text-align: left;
  border-bottom: 1px solid var(--border-color);
}

.admin-table th {
  background: var(--background-light);
  font-weight: 600;
  color: var(--text-primary);
}

.status-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
  text-transform: capitalize;
}

.status-badge.completado {
  background: #d4edda;
  color: #155724;
}

.status-badge.pendiente {
  background: #fff3cd;
  color: #856404;
}

.status-badge.enviado {
  background: #cce5ff;
  color: #004085;
}

.product-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.product-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: var(--background-light);
  border-radius: 8px;
}

.product-thumb {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
}

.product-info {
  flex: 1;
}

.product-info h4 {
  margin: 0 0 0.25rem 0;
  color: var(--text-primary);
}

.product-info p {
  margin: 0;
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.product-price {
  font-weight: bold;
  color: var(--primary-color);
  font-size: 1.1rem;
}

@media (max-width: 768px) {
  .admin-dashboard {
    padding: 1rem;
  }
  
  .dashboard-content {
    grid-template-columns: 1fr;
  }
  
  .stats-grid {
    grid-template-columns: 1fr;
  }
}
</style>
