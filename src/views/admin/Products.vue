
<template>
  <div class="admin-products">
    <div class="products-header">
      <h1>Gestión de Productos</h1>
      <button @click="showCreateModal = true" class="btn btn-primary">
        + Agregar Producto
      </button>
    </div>

    <!-- Filtros y búsqueda -->
    <div class="products-filters">
      <div class="search-box">
        <input 
          v-model="searchTerm" 
          type="text" 
          placeholder="Buscar productos..."
          class="form-control"
        >
      </div>
      <select v-model="selectedCategory" class="form-control">
        <option value="">Todas las categorías</option>
        <option value="electronics">Electrónicos</option>
        <option value="clothing">Ropa</option>
        <option value="books">Libros</option>
      </select>
    </div>

    <!-- Tabla de productos -->
    <div class="products-table-container">
      <table class="admin-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Producto</th>
            <th>Categoría</th>
            <th>Precio</th>
            <th>Stock</th>
            <th>Estado</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="product in filteredProducts" :key="product.id">
            <td>#{{ product.id }}</td>
            <td>
              <div class="product-cell">
                <img :src="product.image" :alt="product.name" class="product-thumb">
                <div>
                  <h4>{{ product.name }}</h4>
                  <p>{{ product.description.substring(0, 50) }}...</p>
                </div>
              </div>
            </td>
            <td>{{ product.category }}</td>
            <td>${{ product.price }}</td>
            <td>
              <span :class="{'low-stock': product.stock < 10}">
                {{ product.stock }}
              </span>
            </td>
            <td>
              <span class="status-badge" :class="product.status">
                {{ product.status }}
              </span>
            </td>
            <td>
              <div class="action-buttons">
                <button @click="editProduct(product)" class="btn btn-sm btn-secondary">
                  ✏️
                </button>
                <button @click="deleteProduct(product.id)" class="btn btn-sm btn-danger">
                  🗑️
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal para crear/editar producto -->
    <div v-if="showCreateModal || showEditModal" class="modal-overlay" @click="closeModals">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h2>{{ showCreateModal ? 'Agregar Producto' : 'Editar Producto' }}</h2>
          <button @click="closeModals" class="close-btn">&times;</button>
        </div>
        
        <form @submit.prevent="submitProduct" class="product-form">
          <div class="form-grid">
            <div class="form-group">
              <label class="form-label">Nombre del producto</label>
              <input v-model="productForm.name" type="text" class="form-control" required>
            </div>
            
            <div class="form-group">
              <label class="form-label">Categoría</label>
              <select v-model="productForm.category" class="form-control" required>
                <option value="">Seleccionar categoría</option>
                <option value="electronics">Electrónicos</option>
                <option value="clothing">Ropa</option>
                <option value="books">Libros</option>
              </select>
            </div>
            
            <div class="form-group">
              <label class="form-label">Precio</label>
              <input v-model="productForm.price" type="number" step="0.01" class="form-control" required>
            </div>
            
            <div class="form-group">
              <label class="form-label">Stock</label>
              <input v-model="productForm.stock" type="number" class="form-control" required>
            </div>
          </div>
          
          <div class="form-group">
            <label class="form-label">Descripción</label>
            <textarea v-model="productForm.description" class="form-control" rows="4" required></textarea>
          </div>
          
          <div class="form-group">
            <label class="form-label">URL de la imagen</label>
            <input v-model="productForm.image" type="url" class="form-control" required>
          </div>
          
          <div class="form-group">
            <label class="form-label">Estado</label>
            <select v-model="productForm.status" class="form-control">
              <option value="active">Activo</option>
              <option value="inactive">Inactivo</option>
            </select>
          </div>
          
          <div class="modal-actions">
            <button type="button" @click="closeModals" class="btn btn-secondary">
              Cancelar
            </button>
            <button type="submit" class="btn btn-primary">
              {{ showCreateModal ? 'Crear Producto' : 'Actualizar Producto' }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { useStore } from '../../composables/useStore'

export default {
  name: 'AdminProducts',
  setup() {
    const store = useStore()
    
    // Estados reactivos
    const products = ref([])
    const searchTerm = ref('')
    const selectedCategory = ref('')
    const showCreateModal = ref(false)
    const showEditModal = ref(false)
    const editingProduct = ref(null)
    
    // Formulario de producto
    const productForm = ref({
      name: '',
      description: '',
      price: '',
      stock: '',
      category: '',
      image: '',
      status: 'active'
    })

    // Productos filtrados
    const filteredProducts = computed(() => {
      let filtered = products.value

      if (searchTerm.value) {
        filtered = filtered.filter(product => 
          product.name.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
          product.description.toLowerCase().includes(searchTerm.value.toLowerCase())
        )
      }

      if (selectedCategory.value) {
        filtered = filtered.filter(product => product.category === selectedCategory.value)
      }

      return filtered
    })

    // Cargar productos
    const loadProducts = async () => {
      try {
        // Aquí harías la llamada a tu API Laravel
        // const response = await store.fetchProducts()
        
        // Datos de ejemplo mientras conectas con Laravel
        products.value = [
          {
            id: 1,
            name: 'Smartphone Pro Max',
            description: 'Último modelo con pantalla OLED y cámara de 108MP',
            price: 899.99,
            stock: 25,
            category: 'electronics',
            image: 'https://via.placeholder.com/80x80',
            status: 'active'
          },
          {
            id: 2,
            name: 'Laptop Gaming RGB',
            description: 'Laptop para gaming con RTX 4070 y 32GB RAM',
            price: 1599.99,
            stock: 8,
            category: 'electronics',
            image: 'https://via.placeholder.com/80x80',
            status: 'active'
          },
          {
            id: 3,
            name: 'Camiseta Premium',
            description: 'Camiseta de algodón 100% orgánico',
            price: 29.99,
            stock: 150,
            category: 'clothing',
            image: 'https://via.placeholder.com/80x80',
            status: 'active'
          }
        ]
      } catch (error) {
        console.error('Error cargando productos:', error)
      }
    }

    // Función para editar producto
    const editProduct = (product) => {
      editingProduct.value = product
      productForm.value = { ...product }
      showEditModal.value = true
    }

    // Función para eliminar producto
    const deleteProduct = async (productId) => {
      if (confirm('¿Estás seguro de que quieres eliminar este producto?')) {
        try {
          // await store.deleteProduct(productId)
          products.value = products.value.filter(p => p.id !== productId)
          alert('Producto eliminado correctamente')
        } catch (error) {
          console.error('Error eliminando producto:', error)
          alert('Error al eliminar el producto')
        }
      }
    }

    // Función para enviar formulario
    const submitProduct = async () => {
      try {
        if (showCreateModal.value) {
          // Crear nuevo producto
          const newProduct = {
            ...productForm.value,
            id: Date.now(), // ID temporal
            price: parseFloat(productForm.value.price),
            stock: parseInt(productForm.value.stock)
          }
          products.value.push(newProduct)
          alert('Producto creado correctamente')
        } else {
          // Actualizar producto existente
          const index = products.value.findIndex(p => p.id === editingProduct.value.id)
          if (index !== -1) {
            products.value[index] = {
              ...productForm.value,
              id: editingProduct.value.id,
              price: parseFloat(productForm.value.price),
              stock: parseInt(productForm.value.stock)
            }
          }
          alert('Producto actualizado correctamente')
        }
        
        closeModals()
      } catch (error) {
        console.error('Error guardando producto:', error)
        alert('Error al guardar el producto')
      }
    }

    // Cerrar modales
    const closeModals = () => {
      showCreateModal.value = false
      showEditModal.value = false
      editingProduct.value = null
      productForm.value = {
        name: '',
        description: '',
        price: '',
        stock: '',
        category: '',
        image: '',
        status: 'active'
      }
    }

    onMounted(() => {
      loadProducts()
    })

    return {
      products,
      searchTerm,
      selectedCategory,
      showCreateModal,
      showEditModal,
      productForm,
      filteredProducts,
      editProduct,
      deleteProduct,
      submitProduct,
      closeModals
    }
  }
}
</script>

<style scoped>
.admin-products {
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.products-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.products-filters {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.search-box {
  flex: 1;
}

.products-table-container {
  background: white;
  border-radius: 12px;
  box-shadow: var(--shadow-sm);
  overflow: hidden;
}

.product-cell {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.product-thumb {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
}

.product-cell h4 {
  margin: 0 0 0.25rem 0;
  font-size: 0.95rem;
}

.product-cell p {
  margin: 0;
  color: var(--text-secondary);
  font-size: 0.8rem;
}

.low-stock {
  color: var(--danger-color);
  font-weight: bold;
}

.action-buttons {
  display: flex;
  gap: 0.5rem;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 12px;
  width: 90%;
  max-width: 600px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid var(--border-color);
}

.close-btn {
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: var(--text-secondary);
}

.product-form {
  padding: 1.5rem;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}

.modal-actions {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 2rem;
}

@media (max-width: 768px) {
  .admin-products {
    padding: 1rem;
  }
  
  .products-header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .products-filters {
    flex-direction: column;
  }
  
  .form-grid {
    grid-template-columns: 1fr;
  }
  
  .modal-actions {
    flex-direction: column;
  }
}
</style>
