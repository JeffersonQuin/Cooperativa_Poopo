<template>
  <footer class="footer dashboard-footer">
    <div class="container">
      <div class="footer-content">
        
        <!-- Info Section -->
        <div class="footer-section">
          <div class="footer-info">
            
            <div class="info-item">
              <div class="logo-container">
                <img src="/logo.jfif" alt="Cooperativa Minera Poopó R.L." class="footer-logo">
                <div class="logo-shine"></div>
              </div>
              <div class="info-text">
                <p class="company-name">Cooperativa Minera Poopó R.L.</p>
                <p class="company-desc">Fundada el 26 de diciembre de 1953</p>
              </div>
            </div>
            
            <div class="info-item">
              <span class="info-icon">
                <i class="mdi mdi-application-cog"></i>
              </span>
              <div class="info-text">
                <p class="system-name">Sistema de Gestión v1.0.0</p>
                <p class="system-desc">Gestión Digital de Asistencia</p>
              </div>
            </div>
            
            <div class="info-item">
              <span class="info-icon">
                <i class="mdi mdi-account-group"></i>
              </span>
              <div class="info-text">
                <p class="directorio-name">Directorio 2025</p>
                <p class="directorio-desc">Al servicio de nuestros cooperativistas</p>
              </div>
            </div>
            
          </div>
        </div>
        
        <!-- Divider -->
        <div class="footer-divider-container">
          <div class="footer-divider"></div>
        </div>
        
        <!-- Copyright -->
        <div class="footer-copyright">
          <p class="copyright-text">
            <span class="copyright-icon">
              <i class="mdi mdi-copyright"></i>
            </span>
            <span>{{ currentYear }} Cooperativa Minera Poopó R.L. - Todos los derechos reservados</span>
          </p>
          <div class="footer-glow"></div>
        </div>
        
      </div>
    </div>
  </footer>
</template>

<script setup>
const currentYear = new Date().getFullYear()
</script>

<style scoped>
.dashboard-footer {
  background: linear-gradient(135deg, #1a2e1a 0%, #0f1f0f 50%, #1e461e 100%);
  padding: 2.5rem 1.5rem 1.5rem;
  margin-top: auto;
  border-top: 3px solid transparent;
  border-image: linear-gradient(135deg, #2e7d32, #9e9d24, #ffd700) 1;
  position: relative;
  overflow: hidden;
}

.dashboard-footer::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, #ffd700, transparent);
  animation: footerShimmer 4s infinite;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  position: relative;
  z-index: 2;
}

.footer-section {
  margin-bottom: 1.5rem;
}

.footer-info {
  display: flex;
  flex-wrap: wrap;
  gap: 2.5rem;
  justify-content: space-around;
  align-items: flex-start;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex: 1;
  min-width: 250px;
  padding: 1rem;
  border-radius: 12px;
  background: linear-gradient(135deg, rgba(26, 46, 26, 0.6), rgba(15, 31, 15, 0.4));
  border: 1px solid rgba(255, 215, 0, 0.2);
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  position: relative;
  overflow: hidden;
}

.info-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 215, 0, 0.1), transparent);
  transition: left 0.6s ease;
}

.info-item:hover::before {
  left: 100%;
}

.info-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(255, 215, 0, 0.2);
  border-color: rgba(255, 215, 0, 0.4);
}

/* Logo Styles */
.logo-container {
  position: relative;
  width: 50px;
  height: 50px;
  flex-shrink: 0;
}

.footer-logo {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  border: 2px solid #ffd700;
  box-shadow: 0 4px 15px rgba(255, 215, 0, 0.3);
  position: relative;
  z-index: 2;
}

.logo-shine {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 60px;
  height: 60px;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.2) 0%, transparent 70%);
  border-radius: 50%;
  animation: logoPulse 3s infinite;
}

/* Icon Styles */
.info-icon {
  font-size: 2rem;
  flex-shrink: 0;
  background: linear-gradient(135deg, #ffd700 0%, #ff9800 50%, #9e9d24 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  background-color: rgba(26, 46, 26, 0.8);
  border: 1px solid rgba(255, 215, 0, 0.3);
}

.info-text {
  flex: 1;
}

.company-name,
.system-name,
.directorio-name {
  font-weight: 700;
  font-size: 0.95rem;
  background: linear-gradient(135deg, #ffd700 0%, #ff9800 50%, #9e9d24 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.25rem;
  text-shadow: 0 2px 8px rgba(255, 215, 0, 0.2);
}

.company-desc,
.system-desc,
.directorio-desc {
  font-size: 0.8rem;
  color: #a5d6a7;
  font-weight: 500;
  line-height: 1.3;
  margin: 0;
}

/* Divider */
.footer-divider-container {
  margin: 2rem 0 1.5rem;
}

.footer-divider {
  height: 2px;
  background: linear-gradient(90deg, 
    transparent 0%, 
    #2e7d32 20%, 
    #9e9d24 50%, 
    #2e7d32 80%, 
    transparent 100%);
  border-radius: 2px;
  position: relative;
}

.footer-divider::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg, transparent, rgba(255, 215, 0, 0.3), transparent);
  animation: dividerShine 3s infinite;
}

/* Copyright */
.footer-copyright {
  text-align: center;
  padding: 1rem;
  border-radius: 10px;
  background: linear-gradient(135deg, rgba(26, 46, 26, 0.7), rgba(15, 31, 15, 0.5));
  border: 1px solid rgba(255, 215, 0, 0.2);
  position: relative;
  overflow: hidden;
}

.copyright-text {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-size: 0.85rem;
  color: #c8e6c9;
  font-weight: 600;
  margin: 0;
  position: relative;
  z-index: 2;
}

.copyright-icon {
  color: #ffd700;
  font-size: 1rem;
}

.footer-glow {
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 215, 0, 0.05), transparent);
  animation: copyrightGlow 6s infinite;
}

/* Animations */
@keyframes footerShimmer {
  0%, 100% { opacity: 0.3; }
  50% { opacity: 1; }
}

@keyframes logoPulse {
  0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.5; }
  50% { transform: translate(-50%, -50%) scale(1.1); opacity: 0.2; }
}

@keyframes dividerShine {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

@keyframes copyrightGlow {
  0% { left: -100%; }
  50% { left: 100%; }
  100% { left: 100%; }
}

/* Responsive */
@media screen and (max-width: 768px) {
  .footer-info {
    flex-direction: column;
    gap: 1.5rem;
  }
  
  .info-item {
    min-width: 100%;
    padding: 1rem 0.75rem;
  }
  
  .dashboard-footer {
    padding: 2rem 1rem 1rem;
  }
  
  .copyright-text {
    flex-direction: column;
    gap: 0.25rem;
    font-size: 0.8rem;
  }
  
  .logo-container,
  .info-icon {
    width: 40px;
    height: 40px;
  }
  
  .info-icon {
    font-size: 1.5rem;
  }
}

@media screen and (min-width: 769px) and (max-width: 1023px) {
  .footer-info {
    justify-content: space-between;
  }
  
  .info-item {
    flex: 0 1 calc(50% - 1.25rem);
  }
}

@media screen and (max-width: 480px) {
  .info-item {
    flex-direction: column;
    text-align: center;
    gap: 0.75rem;
  }
  
  .info-text {
    text-align: center;
  }
}
</style>