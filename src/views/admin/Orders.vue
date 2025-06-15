
<template>
  <div class="admin-orders">
    <div class="orders-header">
      <h1>Gestión de Pedidos</h1>
      <div class="orders-stats">
        <div class="stat-item">
          <span class="stat-number">{{ totalOrders }}</span>
          <span class="stat-label">Total Pedidos</span>
        </div>
        <div class="stat-item">
          <span class="stat-number">${{ totalRevenue }}</span>
          <span class="stat-label">Ingresos</span>
        </div>
      </div>
    </div>

    <!-- Filtros -->
    <div class="orders-filters">
      <select v-model="selectedStatus" class="form-control">
        <option value="">Todos los estados</option>
        <option value="pending">Pendiente</option>
        <option value="processing">Procesando</option>
        <option value="shipped">Enviado</option>
        <option value="delivered">Entregado</option>
        <option value="cancelled">Cancelado</option>
      </select>
      
      <input 
        v-model="searchTerm" 
        type="text" 
        placeholder="Buscar por cliente o ID..."
        class="form-control"
      >
      
      <input 
        v-model="dateFilter" 
        type="date" 
        class="form-control"
      >
    </div>

    <!-- Tabla de pedidos -->
    <div class="orders-table-container">
      <table class="admin-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Cliente</th>
            <th>Productos</th>
            <th>Total</th>
            <th>Estado</th>
            <th>Fecha</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="order in filteredOrders" :key="order.id">
            <td>#{{ order.id }}</td>
            <td>
              <div class="customer-info">
                <strong>{{ order.customer_name }}</strong>
                <br>
                <small>{{ order.customer_email }}</small>
              </div>
            </td>
            <td>
              <div class="order-products">
                <span>{{ order.items_count }} producto(s)</span>
                <br>
                <small>{{ order.products_summary }}</small>
              </div>
            </td>
            <td class="order-total">${{ order.total }}</td>
            <td>
              <select 
                v-model="order.status" 
                @change="updateOrderStatus(order.id, order.status)"
                class="status-select"
                :class="order.status"
              >
                <option value="pending">Pendiente</option>
                <option value="processing">Procesando</option>
                <option value="shipped">Enviado</option>
                <option value="delivered">Entregado</option>
                <option value="cancelled">Cancelado</option>
              </select>
            </td>
            <td>{{ formatDate(order.created_at) }}</td>
            <td>
              <div class="action-buttons">
                <button @click="viewOrderDetails(order)" class="btn btn-sm btn-primary">
                  👁️
                </button>
                <button @click="printInvoice(order)" class="btn btn-sm btn-secondary">
                  🖨️
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal de detalles del pedido -->
    <div v-if="showOrderModal" class="modal-overlay" @click="closeOrderModal">
      <div class="modal-content order-modal" @click.stop>
        <div class="modal-header">
          <h2>Detalles del Pedido #{{ selectedOrder.id }}</h2>
          <button @click="closeOrderModal" class="close-btn">&times;</button>
        </div>
        
        <div class="order-details">
          <!-- Información del cliente -->
          <div class="detail-section">
            <h3>Información del Cliente</h3>
            <div class="detail-grid">
              <div>
                <strong>Nombre:</strong> {{ selectedOrder.customer_name }}
              </div>
              <div>
                <strong>Email:</strong> {{ selectedOrder.customer_email }}
              </div>
              <div>
                <strong>Teléfono:</strong> {{ selectedOrder.customer_phone }}
              </div>
            </div>
          </div>

          <!-- Dirección de envío -->
          <div class="detail-section">
            <h3>Dirección de Envío</h3>
            <p>{{ selectedOrder.shipping_address }}</p>
          </div>

          <!-- Productos del pedido -->
          <div class="detail-section">
            <h3>Productos</h3>
            <div class="order-items">
              <div v-for="item in selectedOrder.items" :key="item.id" class="order-item">
                <img :src="item.image" :alt="item.name" class="item-image">
                <div class="item-info">
                  <h4>{{ item.name }}</h4>
                  <p>Cantidad: {{ item.quantity }}</p>
                  <p>Precio: ${{ item.price }}</p>
                </div>
                <div class="item-total">
                  ${{ (item.price * item.quantity).toFixed(2) }}
                </div>
              </div>
            </div>
          </div>

          <!-- Resumen del pedido -->
          <div class="detail-section">
            <h3>Resumen</h3>
            <div class="order-summary">
              <div class="summary-row">
                <span>Subtotal:</span>
                <span>${{ selectedOrder.subtotal }}</span>
              </div>
              <div class="summary-row">
                <span>Envío:</span>
                <span>${{ selectedOrder.shipping_cost }}</span>
              </div>
              <div class="summary-row">
                <span>Impuestos:</span>
                <span>${{ selectedOrder.tax }}</span>
              </div>
              <div class="summary-row total-row">
                <span>Total:</span>
                <span>${{ selectedOrder.total }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'AdminOrders',
  setup() {
    // Estados reactivos
    const orders = ref([])
    const selectedStatus = ref('')
    const searchTerm = ref('')
    const dateFilter = ref('')
    const showOrderModal = ref(false)
    const selectedOrder = ref(null)

    // Estadísticas computadas
    const totalOrders = computed(() => orders.value.length)
    const totalRevenue = computed(() => 
      orders.value.reduce((sum, order) => sum + parseFloat(order.total), 0).toFixed(2)
    )

    // Pedidos filtrados
    const filteredOrders = computed(() => {
      let filtered = orders.value

      if (selectedStatus.value) {
        filtered = filtered.filter(order => order.status === selectedStatus.value)
      }

      if (searchTerm.value) {
        const term = searchTerm.value.toLowerCase()
        filtered = filtered.filter(order => 
          order.customer_name.toLowerCase().includes(term) ||
          order.customer_email.toLowerCase().includes(term) ||
          order.id.toString().includes(term)
        )
      }

      if (dateFilter.value) {
        filtered = filtered.filter(order => 
          order.created_at.startsWith(dateFilter.value)
        )
      }

      return filtered
    })

    // Formatear fecha
    const formatDate = (date) => {
      return new Date(date).toLocaleDateString('es-ES', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })
    }

    // Cargar pedidos
    const loadOrders = async () => {
      try {
        // Aquí harías la llamada a tu API Laravel
        // const response = await fetch('/api/admin/orders')
        
        // Datos de ejemplo
        orders.value = [
          {
            id: 1001,
            customer_name: 'Juan Pérez',
            customer_email: 'juan@ejemplo.com',
            customer_phone: '+34 666 777 888',
            items_count: 2,
            products_summary: 'Smartphone, Auriculares',
            total: 899.99,
            subtotal: 829.99,
            shipping_cost: 15.00,
            tax: 55.00,
            status: 'processing',
            created_at: '2024-01-15T10:30:00Z',
            shipping_address: 'Calle Principal 123, Madrid, España',
            items: [
              {
                id: 1,
                name: 'Smartphone Pro',
                price: 699.99,
                quantity: 1,
                image: 'https://via.placeholder.com/60x60'
              },
              {
                id: 2,
                name: 'Auriculares Bluetooth',
                price: 129.99,
                quantity: 1,
                image: 'https://via.placeholder.com/60x60'
              }
            ]
          },
          {
            id: 1002,
            customer_name: 'María García',
            customer_email: 'maria@ejemplo.com',
            customer_phone: '+34 555 666 777',
            items_count: 1,
            products_summary: 'Laptop Gaming',
            total: 1599.99,
            subtotal: 1499.99,
            shipping_cost: 0.00,
            tax: 100.00,
            status: 'shipped',
            created_at: '2024-01-14T16:45:00Z',
            shipping_address: 'Avenida Central 456, Barcelona, España',
            items: [
              {
                id: 3,
                name: 'Laptop Gaming RGB',
                price: 1499.99,
                quantity: 1,
                image: 'https://via.placeholder.com/60x60'
              }
            ]
          }
        ]
      } catch (error) {
        console.error('Error cargando pedidos:', error)
      }
    }

    // Actualizar estado del pedido
    const updateOrderStatus = async (orderId, newStatus) => {
      try {
        // Aquí harías la llamada a tu API Laravel
        // await fetch(`/api/admin/orders/${orderId}`, {
        //   method: 'PATCH',
        //   body: JSON.stringify({ status: newStatus })
        // })
        
        console.log(`Pedido ${orderId} actualizado a ${newStatus}`)
        // Mostrar notificación de éxito
      } catch (error) {
        console.error('Error actualizando pedido:', error)
      }
    }

    // Ver detalles del pedido
    const viewOrderDetails = (order) => {
      selectedOrder.value = order
      showOrderModal.value = true
    }

    // Cerrar modal
    const closeOrderModal = () => {
      showOrderModal.value = false
      selectedOrder.value = null
    }

    // Imprimir factura
    const printInvoice = (order) => {
      // Implementar lógica de impresión
      console.log(`Imprimiendo factura del pedido ${order.id}`)
      window.print()
    }

    onMounted(() => {
      loadOrders()
    })

    return {
      orders,
      selectedStatus,
      searchTerm,
      dateFilter,
      showOrderModal,
      selectedOrder,
      totalOrders,
      totalRevenue,
      filteredOrders,
      formatDate,
      updateOrderStatus,
      viewOrderDetails,
      closeOrderModal,
      printInvoice
    }
  }
}
</script>

<style scoped>
.admin-orders {
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.orders-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.orders-stats {
  display: flex;
  gap: 2rem;
}

.stat-item {
  text-align: center;
}

.stat-number {
  display: block;
  font-size: 2rem;
  font-weight: bold;
  color: var(--primary-color);
}

.stat-label {
  font-size: 0.9rem;
  color: var(--text-secondary);
}

.orders-filters {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.orders-table-container {
  background: white;
  border-radius: 12px;
  box-shadow: var(--shadow-sm);
  overflow: hidden;
}

.customer-info strong {
  color: var(--text-primary);
}

.order-products {
  font-size: 0.9rem;
}

.order-total {
  font-weight: bold;
  color: var(--primary-color);
  font-size: 1.1rem;
}

.status-select {
  padding: 0.25rem 0.5rem;
  border: none;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
}

.status-select.pending {
  background: #fff3cd;
  color: #856404;
}

.status-select.processing {
  background: #cce5ff;
  color: #004085;
}

.status-select.shipped {
  background: #e2e3e5;
  color: #383d41;
}

.status-select.delivered {
  background: #d4edda;
  color: #155724;
}

.status-select.cancelled {
  background: #f8d7da;
  color: #721c24;
}

.order-modal {
  max-width: 800px;
}

.order-details {
  padding: 1.5rem;
}

.detail-section {
  margin-bottom: 2rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border-color);
}

.detail-section:last-child {
  border-bottom: none;
}

.detail-section h3 {
  margin-top: 0;
  margin-bottom: 1rem;
  color: var(--primary-color);
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.order-items {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.order-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: var(--background-light);
  border-radius: 8px;
}

.item-image {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
}

.item-info {
  flex: 1;
}

.item-info h4 {
  margin: 0 0 0.5rem 0;
}

.item-info p {
  margin: 0;
  font-size: 0.9rem;
  color: var(--text-secondary);
}

.item-total {
  font-weight: bold;
  color: var(--primary-color);
  font-size: 1.1rem;
}

.order-summary {
  background: var(--background-light);
  padding: 1rem;
  border-radius: 8px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
}

.total-row {
  font-weight: bold;
  font-size: 1.1rem;
  color: var(--primary-color);
  border-top: 1px solid var(--border-color);
  padding-top: 0.5rem;
  margin-top: 0.5rem;
}

@media (max-width: 768px) {
  .admin-orders {
    padding: 1rem;
  }
  
  .orders-header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .orders-filters {
    flex-direction: column;
  }
  
  .detail-grid {
    grid-template-columns: 1fr;
  }
}
</style>
