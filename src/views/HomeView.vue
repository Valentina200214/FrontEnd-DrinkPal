<template>
  <div class="home-view">
    <div class="home-body">

      <!-- LADO IZQUIERDO: Gota grande -->
      <div class="left-panel">
        <div class="gota-wrapper">
          <img :src="gotaImage" :alt="gotaEstado.label" class="gota-img" />
          <p class="gota-estado-label">{{ gotaEstado.label }}</p>
        </div>

        <!-- Barra de progreso de agua -->
        <div class="progress-container">
          <div class="progress-label">
            <span>Hidratación diaria</span>
            <span class="progress-percent">{{ percent }}%</span>
          </div>
          <div class="progress-bar-bg">
            <div class="progress-bar-fill" :style="{ width: Math.min(percent, 100) + '%' }"></div>
          </div>
        </div>

        <!-- Botones agregar agua -->
        <div class="water-buttons">
          <button class="btn-water" @click="registrarAgua(150)">+ 150 ml</button>
          <button class="btn-water" @click="registrarAgua(250)">+ 250 ml</button>
          <button class="btn-water" @click="registrarAgua(500)">+ 500 ml</button>
          <button class="btn-water" @click="registrarAgua(750)">+ 750 ml</button>
        </div>
      </div>

      <!-- LADO DERECHO: Info -->
      <div class="right-panel">

        <!-- Consumo en ml -->
        <div class="card consumo-card">
          <h3>💧 Consumo de Hoy</h3>
          <div class="consumo-display">
            <span class="consumo-value">{{ waterToday }}</span>
            <span class="consumo-unit">ml</span>
          </div>
          <p class="consumo-meta">Meta: <strong>{{ dailyGoal }} ml</strong> ({{ percent }}% completado)</p>
          <p class="consumo-restante">
            <span v-if="waterToday < dailyGoal">Faltan <strong>{{ dailyGoal - waterToday }} ml</strong> para tu meta</span>
            <span v-else style="color: #00c853;">✅ ¡Meta del día alcanzada!</span>
          </p>
        </div>

        <!-- Mensaje motivador -->
        <div class="card motivacion-card">
          <h3>✨ Motivación</h3>
          <p class="motivacion-msg">{{ gotaEstado.mensaje }}</p>
        </div>

        <!-- Logros y metas -->
        <div class="card logros-card">
          <h3>🏆 Logros y Metas</h3>
          <div class="logros-list">
            <div
              v-for="logro in logros"
              :key="logro.id"
              :class="['logro-item', logro.completado ? 'completado' : 'pendiente']"
            >
              <span class="logro-icon">{{ logro.completado ? '✅' : '🔒' }}</span>
              <div class="logro-info">
                <span class="logro-nombre">{{ logro.nombre }}</span>
                <span class="logro-desc">{{ logro.desc }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Racha -->
        <div class="card racha-card">
          <h3>🔥 Racha Actual</h3>
          <p class="racha-valor">{{ racha }} {{ racha === 1 ? 'día seguido' : 'días seguidos' }}</p>
          <p class="racha-sub">{{ rachaMsg }}</p>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'HomeView',
  props: {
    waterToday: { type: Number, default: 0 },
    dailyGoal: { type: Number, default: 2000 },
    currentUser: { type: Object, default: () => ({}) },
    racha: { type: Number, default: 0 }
  },
  emits: ['agua-registrada'],
  computed: {
    percent() {
      return Math.round((this.waterToday / this.dailyGoal) * 100)
    },
    gotaEstado() {
      const p = this.percent
      if (p === 0) return {
        label: 'Pensativa',
        mensaje: '¡Aún no has bebido agua hoy! Recuerda que tu cuerpo necesita hidratación desde el primer momento del día. ¡Empieza con un vaso!'
      }
      if (p >= 100) return {
        label: 'Enamorada',
        mensaje: '¡Increíble! Superaste tu meta de hidratación. Tu cuerpo te lo agradece. ¡Eres un campeón de la hidratación!'
      }
      if (p >= 70) return {
        label: 'Feliz',
        mensaje: '¡Casi lo logras! Estás muy cerca de completar tu meta diaria. Un par de vasos más y habrás triunfado.'
      }
      if (p >= 50) return {
        label: 'Enérgica',
        mensaje: '¡Vas a mitad de camino! Tu energía y concentración mejoran con cada sorbo. ¡Sigue bebiendo!'
      }
      if (p >= 30) return {
        label: 'Entusiasmada',
        mensaje: '¡Buen inicio! Tu cuerpo está empezando a hidratarse. No te detengas, cada mililitro cuenta.'
      }
      if (p >= 10) return {
        label: 'Asustada',
        mensaje: '¡Apenas empezamos! El agua es esencial para tu cerebro y músculos. ¡Bebe más pronto!'
      }
      return {
        label: 'Pensativa',
        mensaje: '¡Vamos! Solo unos pocos ml bebidos. Tu cuerpo necesita hidratación para funcionar bien. ¡Toma el primer vaso ahora!'
      }
    },
    gotaImage() {
      const p = this.percent
      if (p === 0) return require('@/assets/Gota_pensativa.png')
      if (p >= 100) return require('@/assets/Gota_enamorada.png')
      if (p >= 70) return require('@/assets/Gota_feliz.png')
      if (p >= 50) return require('@/assets/Gota_energica.png')
      if (p >= 30) return require('@/assets/Gota_entusiasmada.png')
      if (p >= 10) return require('@/assets/Gota_asustada.png')
      return require('@/assets/Gota_pensativa.png')
    },
    logros() {
      const p = this.percent
      return [
        { id: 1, nombre: 'Primera gota', desc: 'Bebe al menos 1 ml hoy', completado: this.waterToday > 0 },
        { id: 2, nombre: 'Inicio hidratado', desc: 'Llega al 10% de tu meta', completado: p >= 10 },
        { id: 3, nombre: 'A medio camino', desc: 'Llega al 50% de tu meta', completado: p >= 50 },
        { id: 4, nombre: 'Casi campeón', desc: 'Llega al 70% de tu meta', completado: p >= 70 },
        { id: 5, nombre: '¡Meta del día!', desc: 'Completa el 100% de tu meta', completado: p >= 100 },
        { id: 6, nombre: 'Supersaturado', desc: 'Supera tu meta en un 20%', completado: p >= 120 },
      ]
    },
    rachaMsg() {
      if (this.racha === 0)  return '¡Cumple tu meta hoy para comenzar tu racha! 💪'
      if (this.racha < 3)   return '¡Buen inicio! Mantén el ritmo los próximos días.'
      if (this.racha < 7)   return '¡Vas muy bien! Aguanta hasta los 7 días para el logro "Constante".'
      if (this.racha === 7) return '🏆 ¡Lograste el logro "Constante"! ¡Una semana perfecta!'
      return `¡Increíble! Llevas ${this.racha} días consecutivos cumpliendo tu meta. ¡Imparable!`
    }
  },
  methods: {
  
    async registrarAgua(ml) {
      try {
        const token = localStorage.getItem('token')
        const userId = localStorage.getItem('userId')

        await axios.post(
          'http://localhost:3000/consumos',
          { usuarioFinal: userId, consumoInstante: ml },
          { headers: { 'x-auth-token': token } }
        )

        this.$emit('agua-registrada') // App.vue llama fetchWaterToday()
      } catch (error) {
        console.error('Error al registrar consumo:', error)
      }
    }
  }
}
</script>

<style scoped>
.home-view {
  padding: 24px 32px;
  min-height: calc(100vh - 84px);
  background: #f0f8ff;
}

.home-body {
  display: flex;
  gap: 32px;
  max-width: 1200px;
  margin: 0 auto;
  align-items: flex-start;
}

/* ===== LEFT PANEL ===== */
.left-panel {
  flex: 0 0 340px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.gota-wrapper {
  text-align: center;
}

.gota-img {
  width: 280px;
  height: 280px;
  object-fit: contain;
  filter: drop-shadow(0 8px 32px rgba(79,172,254,0.35));
  transition: all 0.5s ease;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-12px); }
}

.gota-estado-label {
  margin-top: 10px;
  font-weight: 700;
  font-size: 1.1rem;
  color: #4facfe;
}

.progress-container {
  width: 100%;
}
.progress-label {
  display: flex;
  justify-content: space-between;
  font-size: 0.88rem;
  font-weight: 600;
  color: #5a7a8a;
  margin-bottom: 8px;
}
.progress-percent { color: #4facfe; }

.progress-bar-bg {
  background: #d0e8ff;
  border-radius: 100px;
  height: 14px;
  overflow: hidden;
}
.progress-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #4facfe, #00c6ff);
  border-radius: 100px;
  transition: width 0.5s ease;
}

.water-buttons {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  width: 100%;
}
.btn-water {
  padding: 12px;
  background: white;
  border: 2px solid #b8deff;
  color: #0077cc;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-water:hover {
  background: linear-gradient(135deg, #4facfe, #00c6ff);
  color: white;
  border-color: transparent;
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(79,172,254,0.3);
}

/* ===== RIGHT PANEL ===== */
.right-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.card {
  background: white;
  border-radius: 20px;
  padding: 22px 24px;
  box-shadow: 0 6px 24px rgba(79,172,254,0.1);
}

.card h3 {
  font-family: 'Poppins', sans-serif;
  font-size: 1rem;
  font-weight: 600;
  color: #1a2a3a;
  margin-bottom: 14px;
}

/* Consumo */
.consumo-display {
  display: flex;
  align-items: baseline;
  gap: 6px;
  margin-bottom: 8px;
}
.consumo-value {
  font-size: 3rem;
  font-weight: 900;
  color: #4facfe;
  line-height: 1;
}
.consumo-unit { font-size: 1.2rem; font-weight: 700; color: #6b8399; }
.consumo-meta { font-size: 0.9rem; color: #6b8399; margin-bottom: 4px; }
.consumo-restante { font-size: 0.9rem; color: #1a2a3a; }

/* Motivacion */
.motivacion-msg {
  font-size: 0.97rem;
  color: #3a5a6a;
  line-height: 1.65;
  font-style: italic;
}

/* Logros */
.logros-list { display: flex; flex-direction: column; gap: 10px; }
.logro-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 14px;
  border-radius: 12px;
  transition: all 0.2s;
}
.logro-item.completado { background: #e8fdf0; border: 1.5px solid #b2f0cb; }
.logro-item.pendiente { background: #f5f8ff; border: 1.5px solid #dde8ff; opacity: 0.75; }
.logro-icon { font-size: 1.2rem; }
.logro-info { display: flex; flex-direction: column; }
.logro-nombre { font-weight: 700; font-size: 0.9rem; color: #1a2a3a; }
.logro-desc { font-size: 0.8rem; color: #6b8399; }

/* Racha */
.racha-valor { font-size: 2rem; font-weight: 800; color: #ff6b35; margin: 4px 0 6px; }
.racha-sub { font-size: 0.87rem; color: #6b8399; }
</style>
