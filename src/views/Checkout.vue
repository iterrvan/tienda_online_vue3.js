
<template>
  <div class="checkout-page">
    <div class="container">
      <div class="checkout-header">
        <h1>Finalizar Compra</h1>
        <div class="checkout-steps">
          <div class="step" :class="{ active: currentStep >= 1 }">
            <span class="step-number">1</span>
            <span class="step-title">Información</span>
          </div>
          <div class="step" :class="{ active: currentStep >= 2 }">
            <span class="step-number">2</span>
            <span class="step-title">Envío</span>
          </div>
          <div class="step" :class="{ active: currentStep >= 3 }">
            <span class="step-number">3</span>
            <span class="step-title">Pago</span>
          </div>
        </div>
      </div>

      <div class="checkout-content">
        <div class="checkout-form">
          <!-- Paso 1: Información Personal -->
          <div v-show="currentStep === 1" class="checkout-step">
            <h2>Información de Contacto</h2>
            <form @submit.prevent="nextStep">
              <div class="form-grid">
                <div class="form-group">
                  <label class="form-label">Nombre *</label>
                  <input v-model="formData.firstName" type="text" required class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Apellido *</label>
                  <input v-model="formData.lastName" type="text" required class="form-control" />
                </div>
              </div>
              
              <div class="form-group">
                <label class="form-label">Email *</label>
                <input v-model="formData.email" type="email" required class="form-control" />
              </div>
              
              <div class="form-group">
                <label class="form-label">Teléfono *</label>
                <input v-model="formData.phone" type="tel" required class="form-control" />
              </div>
              
              <div class="checkout-actions">
                <router-link to="/cart" class="btn btn-outline">Volver al Carrito</router-link>
                <button type="submit" class="btn btn-primary">Continuar</button>
              </div>
            </form>
          </div>

          <!-- Paso 2: Información de Envío -->
          <div v-show="currentStep === 2" class="checkout-step">
            <h2>Dirección de Envío</h2>
            <form @submit.prevent="nextStep">
              <div class="form-group">
                <label class="form-label">Dirección *</label>
                <input v-model="formData.address" type="text" required class="form-control" />
              </div>
              
              <div class="form-grid">
                <div class="form-group">
                  <label class="form-label">Ciudad *</label>
                  <input v-model="formData.city" type="text" required class="form-control" />
                </div>
                <div class="form-group">
                  <label class="form-label">Código Postal *</label>
                  <input v-model="formData.zipCode" type="text" required class="form-control" />
                </div>
              </div>
              
              <div class="form-group">
                <label class="form-label">País *</label>
                <select v-model="formData.country" required class="form-control">
                  <option value="">Seleccionar país</option>
                  <option value="CO">Colombia</option>
                  <option value="MX">México</option>
                  <option value="AR">Argentina</option>
                  <option value="ES">España</option>
                </select>
              </div>

              <div class="shipping-options">
                <h3>Método de Envío</h3>
                <div class="shipping-option" v-for="option in shippingOptions" :key="option.id">
                  <label class="radio-label">
                    <input 
                      type="radio" 
                      :value="option.id" 
                      v-model="formData.shippingMethod"
                      name="shipping"
                    />
                    <div class="radio-content">
                      <div class="option-info">
                        <strong>{{ option.name }}</strong>
                        <span class="option-time">{{ option.time }}</span>
                      </div>
                      <div class="option-price">${{ option.price }}</div>
                    </div>
                  </label>
                </div>
              </div>
              
              <div class="checkout-actions">
                <button type="button" @click="prevStep" class="btn btn-outline">Atrás</button>
                <button type="submit" class="btn btn-primary">Continuar</button>
              </div>
            </form>
          </div>

          <!-- Paso 3: Pago -->
          <div v-show="currentStep === 3" class="checkout-step">
            <h2>Información de Pago</h2>
            <form @submit.prevent="processPayment">
              <div class="payment-methods">
                <div class="payment-method" v-for="method in paymentMethods" :key="method.id">
                  <label class="radio-label">
                    <input 
                      type="radio" 
                      :value="method.id" 
                      v-model="formData.paymentMethod"
                      name="payment"
                    />
                    <div class="radio-content">
                      <span class="method-icon">{{ method.icon }}</span>
                      <span class="method-name">{{ method.name }}</span>
                    </div>
                  </label>
                </div>
              </div>

              <div v-if="formData.paymentMethod === 'card'" class="card-form">
                <div class="form-group">
                  <label class="form-label">Número de Tarjeta *</label>
                  <input v-model="formData.cardNumber" type="text" required class="form-control" 
                         placeholder="1234 5678 9012 3456" />
                </div>
                
                <div class="form-grid">
                  <div class="form-group">
                    <label class="form-label">Vencimiento *</label>
                    <input v-model="formData.cardExpiry" type="text" required class="form-control" 
                           placeholder="MM/AA" />
                  </div>
                  <div class="form-group">
                    <label class="form-label">CVV *</label>
                    <input v-model="formData.cardCvv" type="text" required class="form-control" 
                           placeholder="123" />
                  </div>
                </div>
                
                <div class="form-group">
                  <label class="form-label">Nombre en la Tarjeta *</label>
                  <input v-model="formData.cardName" type="text" required class="form-control" />
                </div>
              </div>
              
              <div class="checkout-actions">
                <button type="button" @click="prevStep" class="btn btn-outline">Atrás</button>
                <button type="submit" class="btn btn-primary btn-lg" :disabled="isProcessing">
                  {{ isProcessing ? 'Procesando...' : `Pagar $${finalTotal.toFixed(2)}` }}
                </button>
              </div>
            </form>
          </div>
        </div>

        <!-- Resumen del Pedido -->
        <div class="order-summary">
          <div class="summary-card">
            <h3>Resumen del Pedido</h3>
            
            <div class="summary-items">
              <div v-for="item in cartItems" :key="item.id" class="summary-item">
                <img :src="item.image" :alt="item.name" class="item-image" />
                <div class="item-details">
                  <div class="item-name">{{ item.name }}</div>
                  <div class="item-quantity">Cantidad: {{ item.quantity }}</div>
                </div>
                <div class="item-price">${{ (item.price * item.quantity).toFixed(2) }}</div>
              </div>
            </div>

            <div class="summary-totals">
              <div class="total-line">
                <span>Subtotal:</span>
                <span>${{ subtotal.toFixed(2) }}</span>
              </div>
              <div class="total-line">
                <span>Envío:</span>
                <span>${{ shippingCost.toFixed(2) }}</span>
              </div>
              <div class="total-line">
                <span>Impuestos:</span>
                <span>${{ taxes.toFixed(2) }}</span>
              </div>
              <div class="total-line total">
                <span>Total:</span>
                <span>${{ finalTotal.toFixed(2) }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from '../composables/useStore'

const router = useRouter()
const store = useStore()

// Estados
const currentStep = ref(1)
const isProcessing = ref(false)

const formData = reactive({
  firstName: '',
  lastName: '',
  email: store.user.value?.email || '',
  phone: '',
  address: '',
  city: '',
  zipCode: '',
  country: '',
  shippingMethod: 'standard',
  paymentMethod: 'card',
  cardNumber: '',
  cardExpiry: '',
  cardCvv: '',
  cardName: ''
})

// Opciones
const shippingOptions = [
  { id: 'standard', name: 'Envío Estándar', time: '5-7 días', price: 10 },
  { id: 'express', name: 'Envío Express', time: '2-3 días', price: 20 },
  { id: 'overnight', name: 'Entrega Inmediata', time: '24 horas', price: 35 }
]

const paymentMethods = [
  { id: 'card', name: 'Tarjeta de Crédito/Débito', icon: '💳' },
  { id: 'paypal', name: 'PayPal', icon: '🟦' },
  { id: 'bank', name: 'Transferencia Bancaria', icon: '🏦' }
]

// Computed
const cartItems = computed(() => store.cartItems.value)
const subtotal = computed(() => store.cartTotal.value)
const shippingCost = computed(() => {
  const option = shippingOptions.find(o => o.id === formData.shippingMethod)
  return option ? option.price : 0
})
const taxes = computed(() => subtotal.value * 0.1)
const finalTotal = computed(() => subtotal.value + shippingCost.value + taxes.value)

// Métodos
const nextStep = () => {
  if (currentStep.value < 3) {
    currentStep.value++
  }
}

const prevStep = () => {
  if (currentStep.value > 1) {
    currentStep.value--
  }
}

const processPayment = async () => {
  try {
    isProcessing.value = true
    
    // Simular procesamiento de pago
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    // Crear orden
    const orderData = {
      ...formData,
      items: cartItems.value,
      subtotal: subtotal.value,
      shippingCost: shippingCost.value,
      taxes: taxes.value,
      total: finalTotal.value
    }
    
    await store.createOrder(orderData)
    
    // Redireccionar a confirmación
    router.push('/order-confirmation')
    
  } catch (error) {
    alert('Error al procesar el pago. Inténtalo de nuevo.')
    console.error('Payment error:', error)
  } finally {
    isProcessing.value = false
  }
}
</script>

<style scoped>
.checkout-page {
  min-height: 80vh;
  padding: 2rem 0;
}

.checkout-header {
  text-align: center;
  margin-bottom: 3rem;
}

.checkout-steps {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-top: 2rem;
}

.step {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  opacity: 0.5;
  transition: opacity 0.3s ease;
}

.step.active {
  opacity: 1;
}

.step-number {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: var(--border-color);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
}

.step.active .step-number {
  background: var(--primary-color);
  color: white;
}

.checkout-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 3rem;
}

.checkout-form {
  background: white;
  padding: 2rem;
  border-radius: var(--border-radius-lg);
  box-shadow: var(--shadow-sm);
  height: fit-content;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.shipping-options,
.payment-methods {
  margin: 2rem 0;
}

.shipping-option,
.payment-method {
  margin-bottom: 1rem;
}

.radio-label {
  display: block;
  cursor: pointer;
  border: 2px solid var(--border-color);
  border-radius: var(--border-radius);
  padding: 1rem;
  transition: all 0.3s ease;
}

.radio-label:hover {
  border-color: var(--primary-color);
}

.radio-label:has(input:checked) {
  border-color: var(--primary-color);
  background: rgba(52, 152, 219, 0.05);
}

.radio-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.option-info {
  display: flex;
  flex-direction: column;
}

.option-time {
  font-size: 0.9rem;
  color: var(--text-secondary);
}

.option-price {
  font-weight: 600;
  color: var(--primary-color);
}

.method-icon {
  font-size: 1.5rem;
  margin-right: 0.5rem;
}

.card-form {
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid var(--border-color);
}

.checkout-actions {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  margin-top: 2rem;
}

.order-summary {
  position: sticky;
  top: 2rem;
  height: fit-content;
}

.summary-card {
  background: white;
  border-radius: var(--border-radius-lg);
  box-shadow: var(--shadow-sm);
  padding: 2rem;
}

.summary-items {
  margin: 1.5rem 0;
}

.summary-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem 0;
  border-bottom: 1px solid var(--border-color);
}

.item-image {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: var(--border-radius);
}

.item-details {
  flex: 1;
}

.item-name {
  font-weight: 500;
  margin-bottom: 0.25rem;
}

.item-quantity {
  font-size: 0.9rem;
  color: var(--text-secondary);
}

.item-price {
  font-weight: 600;
  color: var(--primary-color);
}

.summary-totals {
  border-top: 1px solid var(--border-color);
  padding-top: 1rem;
}

.total-line {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
}

.total-line.total {
  font-size: 1.2rem;
  font-weight: 600;
  color: var(--primary-color);
  border-top: 1px solid var(--border-color);
  padding-top: 1rem;
  margin-top: 1rem;
}

@media (max-width: 768px) {
  .checkout-content {
    grid-template-columns: 1fr;
  }
  
  .checkout-steps {
    gap: 1rem;
  }
  
  .step-title {
    display: none;
  }
  
  .form-grid {
    grid-template-columns: 1fr;
  }
  
  .checkout-actions {
    flex-direction: column;
  }
}
</style>
