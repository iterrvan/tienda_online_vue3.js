
<template>
  <footer class="footer">
    <div class="container">
      <!-- Newsletter -->
      <div class="newsletter-section">
        <div class="newsletter-content">
          <h3>¡Mantente al día!</h3>
          <p>Suscríbete y recibe ofertas exclusivas y novedades</p>
        </div>
        <form @submit.prevent="subscribeNewsletter" class="newsletter-form">
          <input 
            v-model="email" 
            type="email" 
            placeholder="Tu email aquí..." 
            required
            class="newsletter-input"
          />
          <button type="submit" class="newsletter-btn">
            Suscribirse
          </button>
        </form>
      </div>

      <!-- Footer Links -->
      <div class="footer-content">
        <div class="footer-section">
          <h4>TuTienda</h4>
          <p>La mejor experiencia de compra online con productos de calidad y servicio excepcional.</p>
          <div class="social-links">
            <a href="#" class="social-link">📘</a>
            <a href="#" class="social-link">📷</a>
            <a href="#" class="social-link">🐦</a>
            <a href="#" class="social-link">📺</a>
          </div>
        </div>

        <div class="footer-section">
          <h4>Productos</h4>
          <ul class="footer-links">
            <li><router-link to="/products?category=electronics">Electrónicos</router-link></li>
            <li><router-link to="/products?category=clothing">Ropa</router-link></li>
            <li><router-link to="/products?category=home">Hogar</router-link></li>
            <li><router-link to="/products?category=sports">Deportes</router-link></li>
            <li><router-link to="/products?featured=true">Destacados</router-link></li>
          </ul>
        </div>

        <div class="footer-section">
          <h4>Cuenta</h4>
          <ul class="footer-links">
            <li><router-link to="/login">Iniciar Sesión</router-link></li>
            <li><router-link to="/register">Crear Cuenta</router-link></li>
            <li><router-link to="/profile">Mi Perfil</router-link></li>
            <li><router-link to="/orders">Mis Pedidos</router-link></li>
            <li><router-link to="/wishlist">Lista de Deseos</router-link></li>
          </ul>
        </div>

        <div class="footer-section">
          <h4>Ayuda</h4>
          <ul class="footer-links">
            <li><a href="#" @click="openChat">Centro de Ayuda</a></li>
            <li><a href="#" @click="openChat">Contacto</a></li>
            <li><a href="#" @click="openChat">Envíos</a></li>
            <li><a href="#" @click="openChat">Devoluciones</a></li>
            <li><a href="#" @click="openChat">Garantía</a></li>
          </ul>
        </div>

        <div class="footer-section">
          <h4>Empresa</h4>
          <ul class="footer-links">
            <li><a href="#" @click="openModal('about')">Sobre Nosotros</a></li>
            <li><a href="#" @click="openModal('careers')">Trabajar Aquí</a></li>
            <li><a href="#" @click="openModal('press')">Prensa</a></li>
            <li><a href="#" @click="openModal('sustainability')">Sostenibilidad</a></li>
          </ul>
        </div>

        <div class="footer-section">
          <h4>Contacto</h4>
          <div class="contact-info">
            <p>📞 +1 234 567 8900</p>
            <p>✉️ info@tutienda.com</p>
            <p>📍 Calle Principal 123, Ciudad</p>
            <p>🕒 Lun-Vie 9:00-18:00</p>
          </div>
        </div>
      </div>

      <!-- Footer Bottom -->
      <div class="footer-bottom">
        <div class="footer-bottom-content">
          <p>&copy; {{ currentYear }} TuTienda. Todos los derechos reservados.</p>
          <div class="footer-bottom-links">
            <a href="#" @click="openModal('privacy')">Privacidad</a>
            <a href="#" @click="openModal('terms')">Términos</a>
            <a href="#" @click="openModal('cookies')">Cookies</a>
          </div>
        </div>
        <div class="payment-methods">
          <span>Métodos de pago:</span>
          <div class="payment-icons">
            <span>💳</span>
            <span>🏧</span>
            <span>📱</span>
            <span>🎯</span>
          </div>
        </div>
      </div>
    </div>
  </footer>
</template>

<script setup>
import { ref, computed } from 'vue'

const email = ref('')
const currentYear = computed(() => new Date().getFullYear())

const subscribeNewsletter = () => {
  if (email.value) {
    alert(`¡Gracias por suscribirte con el email: ${email.value}!`)
    email.value = ''
    // Aquí iría la integración con tu API de newsletter
  }
}

const openChat = () => {
  alert('Función de chat próximamente...')
  // Aquí iría la integración con tu sistema de chat/soporte
}

const openModal = (type) => {
  alert(`Modal de ${type} próximamente...`)
  // Aquí iría la lógica para abrir modales con información legal
}
</script>

<style scoped>
.footer {
  background: var(--secondary-color);
  color: white;
  margin-top: auto;
}

.newsletter-section {
  background: var(--primary-color);
  padding: 3rem 0;
  text-align: center;
}

.newsletter-content h3 {
  font-size: 2rem;
  margin-bottom: 1rem;
  color: white;
}

.newsletter-content p {
  font-size: 1.1rem;
  margin-bottom: 2rem;
  opacity: 0.9;
}

.newsletter-form {
  display: flex;
  max-width: 500px;
  margin: 0 auto;
  gap: 1rem;
}

.newsletter-input {
  flex: 1;
  padding: 1rem;
  border: none;
  border-radius: var(--border-radius);
  font-size: 1rem;
}

.newsletter-btn {
  background: var(--accent-color);
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: var(--border-radius);
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.newsletter-btn:hover {
  background: #c0392b;
}

.footer-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 3rem;
  padding: 4rem 0;
}

.footer-section h4 {
  font-size: 1.2rem;
  margin-bottom: 1.5rem;
  color: var(--primary-color);
}

.footer-section p {
  line-height: 1.6;
  opacity: 0.9;
  margin-bottom: 1.5rem;
}

.social-links {
  display: flex;
  gap: 1rem;
}

.social-link {
  display: inline-block;
  width: 40px;
  height: 40px;
  background: var(--primary-color);
  border-radius: 50%;
  text-align: center;
  line-height: 40px;
  font-size: 1.2rem;
  transition: transform 0.3s ease;
}

.social-link:hover {
  transform: translateY(-3px);
  background: var(--accent-color);
}

.footer-links {
  list-style: none;
}

.footer-links li {
  margin-bottom: 0.75rem;
}

.footer-links a {
  color: rgba(255, 255, 255, 0.8);
  text-decoration: none;
  transition: color 0.3s ease;
}

.footer-links a:hover {
  color: var(--primary-color);
}

.contact-info p {
  margin-bottom: 0.75rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.footer-bottom {
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding: 2rem 0;
}

.footer-bottom-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.footer-bottom-links {
  display: flex;
  gap: 2rem;
}

.footer-bottom-links a {
  color: rgba(255, 255, 255, 0.7);
  text-decoration: none;
  transition: color 0.3s ease;
}

.footer-bottom-links a:hover {
  color: var(--primary-color);
}

.payment-methods {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  opacity: 0.8;
}

.payment-icons {
  display: flex;
  gap: 0.5rem;
}

.payment-icons span {
  font-size: 1.5rem;
}

@media (max-width: 768px) {
  .newsletter-form {
    flex-direction: column;
  }
  
  .footer-content {
    grid-template-columns: 1fr;
    gap: 2rem;
  }
  
  .footer-bottom-content {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
  
  .footer-bottom-links {
    justify-content: center;
  }
  
  .payment-methods {
    flex-direction: column;
    gap: 0.5rem;
  }
}
</style>
