
<template>
  <div class="admin-settings">
    <div class="settings-header">
      <h1>Configuración del Sistema</h1>
      <p>Gestiona la configuración general de tu tienda</p>
    </div>

    <div class="settings-content">
      <!-- Configuración General -->
      <div class="settings-section">
        <h2>🏪 Configuración General</h2>
        <form @submit.prevent="saveGeneralSettings" class="settings-form">
          <div class="form-grid">
            <div class="form-group">
              <label class="form-label">Nombre de la tienda</label>
              <input v-model="generalSettings.store_name" type="text" class="form-control" required>
            </div>
            
            <div class="form-group">
              <label class="form-label">Email de contacto</label>
              <input v-model="generalSettings.contact_email" type="email" class="form-control" required>
            </div>
            
            <div class="form-group">
              <label class="form-label">Teléfono</label>
              <input v-model="generalSettings.phone" type="tel" class="form-control">
            </div>
            
            <div class="form-group">
              <label class="form-label">Moneda</label>
              <select v-model="generalSettings.currency" class="form-control">
                <option value="EUR">Euro (€)</option>
                <option value="USD">Dólar ($)</option>
                <option value="GBP">Libra (£)</option>
              </select>
            </div>
          </div>
          
          <div class="form-group">
            <label class="form-label">Dirección</label>
            <textarea v-model="generalSettings.address" class="form-control" rows="3"></textarea>
          </div>
          
          <div class="form-group">
            <label class="form-label">Descripción de la tienda</label>
            <textarea v-model="generalSettings.description" class="form-control" rows="4"></textarea>
          </div>
          
          <button type="submit" class="btn btn-primary">Guardar Configuración General</button>
        </form>
      </div>

      <!-- Configuración de Envío -->
      <div class="settings-section">
        <h2>🚚 Configuración de Envío</h2>
        <form @submit.prevent="saveShippingSettings" class="settings-form">
          <div class="form-grid">
            <div class="form-group">
              <label class="form-label">Costo de envío estándar</label>
              <input v-model="shippingSettings.standard_cost" type="number" step="0.01" class="form-control">
            </div>
            
            <div class="form-group">
              <label class="form-label">Envío gratis desde</label>
              <input v-model="shippingSettings.free_shipping_threshold" type="number" step="0.01" class="form-control">
            </div>
            
            <div class="form-group">
              <label class="form-label">Tiempo de entrega (días)</label>
              <input v-model="shippingSettings.delivery_time" type="number" class="form-control">
            </div>
            
            <div class="form-group">
              <label class="form-label">Costo de envío express</label>
              <input v-model="shippingSettings.express_cost" type="number" step="0.01" class="form-control">
            </div>
          </div>
          
          <div class="form-group">
            <label class="form-label">Países de envío (separados por comas)</label>
            <textarea v-model="shippingSettings.shipping_countries" class="form-control" rows="3"></textarea>
          </div>
          
          <button type="submit" class="btn btn-primary">Guardar Configuración de Envío</button>
        </form>
      </div>

      <!-- Configuración de Pagos -->
      <div class="settings-section">
        <h2>💳 Configuración de Pagos</h2>
        <form @submit.prevent="savePaymentSettings" class="settings-form">
          <div class="payment-methods">
            <h3>Métodos de pago activos</h3>
            
            <div class="payment-method">
              <div class="method-header">
                <label class="checkbox-label">
                  <input v-model="paymentSettings.stripe_enabled" type="checkbox">
                  <span>Stripe</span>
                </label>
              </div>
              <div v-if="paymentSettings.stripe_enabled" class="method-config">
                <div class="form-grid">
                  <div class="form-group">
                    <label class="form-label">Stripe Public Key</label>
                    <input v-model="paymentSettings.stripe_public_key" type="text" class="form-control">
                  </div>
                  <div class="form-group">
                    <label class="form-label">Stripe Secret Key</label>
                    <input v-model="paymentSettings.stripe_secret_key" type="password" class="form-control">
                  </div>
                </div>
              </div>
            </div>
            
            <div class="payment-method">
              <div class="method-header">
                <label class="checkbox-label">
                  <input v-model="paymentSettings.paypal_enabled" type="checkbox">
                  <span>PayPal</span>
                </label>
              </div>
              <div v-if="paymentSettings.paypal_enabled" class="method-config">
                <div class="form-grid">
                  <div class="form-group">
                    <label class="form-label">PayPal Client ID</label>
                    <input v-model="paymentSettings.paypal_client_id" type="text" class="form-control">
                  </div>
                  <div class="form-group">
                    <label class="form-label">PayPal Secret</label>
                    <input v-model="paymentSettings.paypal_secret" type="password" class="form-control">
                  </div>
                </div>
              </div>
            </div>
            
            <div class="payment-method">
              <label class="checkbox-label">
                <input v-model="paymentSettings.bank_transfer_enabled" type="checkbox">
                <span>Transferencia Bancaria</span>
              </label>
              <div v-if="paymentSettings.bank_transfer_enabled" class="method-config">
                <div class="form-group">
                  <label class="form-label">Información bancaria</label>
                  <textarea v-model="paymentSettings.bank_info" class="form-control" rows="4"></textarea>
                </div>
              </div>
            </div>
          </div>
          
          <button type="submit" class="btn btn-primary">Guardar Configuración de Pagos</button>
        </form>
      </div>

      <!-- Configuración de Email -->
      <div class="settings-section">
        <h2>📧 Configuración de Email</h2>
        <form @submit.prevent="saveEmailSettings" class="settings-form">
          <div class="form-grid">
            <div class="form-group">
              <label class="form-label">Servidor SMTP</label>
              <input v-model="emailSettings.smtp_host" type="text" class="form-control">
            </div>
            
            <div class="form-group">
              <label class="form-label">Puerto SMTP</label>
              <input v-model="emailSettings.smtp_port" type="number" class="form-control">
            </div>
            
            <div class="form-group">
              <label class="form-label">Usuario SMTP</label>
              <input v-model="emailSettings.smtp_username" type="text" class="form-control">
            </div>
            
            <div class="form-group">
              <label class="form-label">Contraseña SMTP</label>
              <input v-model="emailSettings.smtp_password" type="password" class="form-control">
            </div>
          </div>
          
          <div class="form-group">
            <label class="checkbox-label">
              <input v-model="emailSettings.smtp_secure" type="checkbox">
              <span>Usar SSL/TLS</span>
            </label>
          </div>
          
          <div class="form-group">
            <label class="form-label">Email desde</label>
            <input v-model="emailSettings.from_email" type="email" class="form-control">
          </div>
          
          <div class="form-group">
            <label class="form-label">Nombre desde</label>
            <input v-model="emailSettings.from_name" type="text" class="form-control">
          </div>
          
          <div class="form-actions">
            <button type="button" @click="testEmail" class="btn btn-secondary">
              Enviar Email de Prueba
            </button>
            <button type="submit" class="btn btn-primary">
              Guardar Configuración de Email
            </button>
          </div>
        </form>
      </div>

      <!-- Configuración de SEO -->
      <div class="settings-section">
        <h2>🔍 Configuración SEO</h2>
        <form @submit.prevent="saveSeoSettings" class="settings-form">
          <div class="form-group">
            <label class="form-label">Título del sitio</label>
            <input v-model="seoSettings.site_title" type="text" class="form-control">
          </div>
          
          <div class="form-group">
            <label class="form-label">Meta descripción</label>
            <textarea v-model="seoSettings.meta_description" class="form-control" rows="3"></textarea>
          </div>
          
          <div class="form-group">
            <label class="form-label">Palabras clave (separadas por comas)</label>
            <textarea v-model="seoSettings.meta_keywords" class="form-control" rows="2"></textarea>
          </div>
          
          <div class="form-group">
            <label class="form-label">Google Analytics ID</label>
            <input v-model="seoSettings.google_analytics_id" type="text" class="form-control">
          </div>
          
          <div class="form-group">
            <label class="form-label">Facebook Pixel ID</label>
            <input v-model="seoSettings.facebook_pixel_id" type="text" class="form-control">
          </div>
          
          <button type="submit" class="btn btn-primary">Guardar Configuración SEO</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'

export default {
  name: 'AdminSettings',
  setup() {
    // Estados reactivos para cada sección de configuración
    const generalSettings = ref({
      store_name: '',
      contact_email: '',
      phone: '',
      currency: 'EUR',
      address: '',
      description: ''
    })

    const shippingSettings = ref({
      standard_cost: 0,
      free_shipping_threshold: 0,
      delivery_time: 3,
      express_cost: 0,
      shipping_countries: 'España, Francia, Portugal, Italia'
    })

    const paymentSettings = ref({
      stripe_enabled: false,
      stripe_public_key: '',
      stripe_secret_key: '',
      paypal_enabled: false,
      paypal_client_id: '',
      paypal_secret: '',
      bank_transfer_enabled: false,
      bank_info: ''
    })

    const emailSettings = ref({
      smtp_host: '',
      smtp_port: 587,
      smtp_username: '',
      smtp_password: '',
      smtp_secure: true,
      from_email: '',
      from_name: ''
    })

    const seoSettings = ref({
      site_title: '',
      meta_description: '',
      meta_keywords: '',
      google_analytics_id: '',
      facebook_pixel_id: ''
    })

    // Cargar configuraciones
    const loadSettings = async () => {
      try {
        // Aquí harías las llamadas a tu API Laravel para cargar las configuraciones
        // const response = await fetch('/api/admin/settings')
        
        // Datos de ejemplo
        generalSettings.value = {
          store_name: 'TuTienda Online',
          contact_email: 'contacto@tutienda.com',
          phone: '+34 900 123 456',
          currency: 'EUR',
          address: 'Calle Principal 123, 28001 Madrid, España',
          description: 'Tu tienda online de confianza con los mejores productos al mejor precio.'
        }

        shippingSettings.value = {
          standard_cost: 4.99,
          free_shipping_threshold: 50.00,
          delivery_time: 3,
          express_cost: 9.99,
          shipping_countries: 'España, Francia, Portugal, Italia, Alemania'
        }

        // Configuraciones iniciales vacías para seguridad
        paymentSettings.value = {
          stripe_enabled: true,
          stripe_public_key: 'pk_test_...',
          stripe_secret_key: '',
          paypal_enabled: false,
          paypal_client_id: '',
          paypal_secret: '',
          bank_transfer_enabled: true,
          bank_info: 'Banco: TuBanco\nIBAN: ES12 3456 7890 1234 5678 9012\nTitular: TuTienda S.L.'
        }

        emailSettings.value = {
          smtp_host: 'smtp.gmail.com',
          smtp_port: 587,
          smtp_username: 'tu-email@gmail.com',
          smtp_password: '',
          smtp_secure: true,
          from_email: 'noreply@tutienda.com',
          from_name: 'TuTienda'
        }

        seoSettings.value = {
          site_title: 'TuTienda - Tienda Online',
          meta_description: 'Encuentra los mejores productos en TuTienda. Envío gratis, devoluciones fáciles y atención al cliente 24/7.',
          meta_keywords: 'tienda online, productos, ofertas, envío gratis',
          google_analytics_id: 'GA-XXXXXXXXX-X',
          facebook_pixel_id: ''
        }

      } catch (error) {
        console.error('Error cargando configuraciones:', error)
      }
    }

    // Funciones para guardar cada sección
    const saveGeneralSettings = async () => {
      try {
        // await fetch('/api/admin/settings/general', {
        //   method: 'POST',
        //   body: JSON.stringify(generalSettings.value)
        // })
        alert('Configuración general guardada correctamente')
      } catch (error) {
        console.error('Error guardando configuración general:', error)
        alert('Error al guardar la configuración general')
      }
    }

    const saveShippingSettings = async () => {
      try {
        // await fetch('/api/admin/settings/shipping', {
        //   method: 'POST',
        //   body: JSON.stringify(shippingSettings.value)
        // })
        alert('Configuración de envío guardada correctamente')
      } catch (error) {
        console.error('Error guardando configuración de envío:', error)
        alert('Error al guardar la configuración de envío')
      }
    }

    const savePaymentSettings = async () => {
      try {
        // await fetch('/api/admin/settings/payment', {
        //   method: 'POST',
        //   body: JSON.stringify(paymentSettings.value)
        // })
        alert('Configuración de pagos guardada correctamente')
      } catch (error) {
        console.error('Error guardando configuración de pagos:', error)
        alert('Error al guardar la configuración de pagos')
      }
    }

    const saveEmailSettings = async () => {
      try {
        // await fetch('/api/admin/settings/email', {
        //   method: 'POST',
        //   body: JSON.stringify(emailSettings.value)
        // })
        alert('Configuración de email guardada correctamente')
      } catch (error) {
        console.error('Error guardando configuración de email:', error)
        alert('Error al guardar la configuración de email')
      }
    }

    const saveSeoSettings = async () => {
      try {
        // await fetch('/api/admin/settings/seo', {
        //   method: 'POST',
        //   body: JSON.stringify(seoSettings.value)
        // })
        alert('Configuración SEO guardada correctamente')
      } catch (error) {
        console.error('Error guardando configuración SEO:', error)
        alert('Error al guardar la configuración SEO')
      }
    }

    // Enviar email de prueba
    const testEmail = async () => {
      try {
        // await fetch('/api/admin/settings/test-email', {
        //   method: 'POST',
        //   body: JSON.stringify({ to: emailSettings.value.from_email })
        // })
        alert('Email de prueba enviado correctamente')
      } catch (error) {
        console.error('Error enviando email de prueba:', error)
        alert('Error al enviar el email de prueba')
      }
    }

    onMounted(() => {
      loadSettings()
    })

    return {
      generalSettings,
      shippingSettings,
      paymentSettings,
      emailSettings,
      seoSettings,
      saveGeneralSettings,
      saveShippingSettings,
      savePaymentSettings,
      saveEmailSettings,
      saveSeoSettings,
      testEmail
    }
  }
}
</script>

<style scoped>
.admin-settings {
  padding: 2rem;
  max-width: 1000px;
  margin: 0 auto;
}

.settings-header {
  margin-bottom: 2rem;
}

.settings-header h1 {
  color: var(--primary-color);
  margin-bottom: 0.5rem;
}

.settings-content {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.settings-section {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: var(--shadow-sm);
}

.settings-section h2 {
  margin-top: 0;
  margin-bottom: 1.5rem;
  color: var(--text-primary);
  font-size: 1.5rem;
}

.settings-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.payment-methods {
  margin-bottom: 1.5rem;
}

.payment-methods h3 {
  margin-bottom: 1rem;
  color: var(--text-primary);
}

.payment-method {
  border: 1px solid var(--border-color);
  border-radius: 8px;
  margin-bottom: 1rem;
  overflow: hidden;
}

.method-header {
  padding: 1rem;
  background: var(--background-light);
  border-bottom: 1px solid var(--border-color);
}

.method-config {
  padding: 1rem;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  font-weight: 500;
}

.checkbox-label input[type="checkbox"] {
  width: 18px;
  height: 18px;
  accent-color: var(--primary-color);
}

.form-actions {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
}

@media (max-width: 768px) {
  .admin-settings {
    padding: 1rem;
  }
  
  .settings-section {
    padding: 1rem;
  }
  
  .form-actions {
    flex-direction: column;
  }
}
</style>
