<template>
  <div class="perfil-view">
    <div class="perfil-container">
      <h2 class="section-title">👤 Perfil</h2>

      <div class="perfil-grid">

        <!-- Tarjeta de perfil -->
        <div class="card perfil-card">
          <div class="avatar">
            <span class="avatar-icon">💧</span>
          </div>
          <h3 class="perfil-name">{{ currentUser.name || 'Usuario DrinkPal' }}</h3>
          <p class="perfil-id">ID: {{ currentUser.id_producto || '—' }}</p>

          <div class="perfil-info-list">
            <div class="info-row">
              <span class="info-label">Meta diaria</span>
              <!-- Muestra la meta GUARDADA, no la que está siendo editada en el input -->
              <span class="info-val">{{ metaDiariaGuardada.toLocaleString('es-CO') }} ml</span>
            </div>
            <div class="info-row">
              <span class="info-label">Racha actual</span>
              <span class="info-val flame">🔥 {{ racha }} {{ racha === 1 ? 'día' : 'días' }}</span>
            </div>
            <div class="info-row">
              <span class="info-label">Logros desbloqueados</span>
              <span class="info-val">{{ logrosDesbloqueados }} / 6</span>
            </div>
            <div class="info-row">
              <span class="info-label">Fecha de registro</span>
              <span class="info-val">{{ fechaRegistroFormateada }}</span>
            </div>
          </div>

          <button class="btn-danger" @click="$emit('logout')">Cerrar Sesión</button>
        </div>

        <!-- Configuración -->
        <div class="card config-card">
          <h3>⚙️ Configuración</h3>

          <div class="config-group">
            <label>Meta diaria de agua (ml)</label>
            <!-- metaDiariaInput es independiente hasta que el usuario presione "Guardar Cambios" -->
            <input type="number" v-model.number="metaDiariaInput" class="config-input" min="500" max="5000" step="100" />
          </div>

          <div class="config-group">
            <label>Recordatorios</label>
            <div class="toggle-row">
              <span>Notificaciones de hidratación</span>
              <div class="toggle" :class="{ on: notificaciones }" @click="notificaciones = !notificaciones">
                <div class="toggle-thumb"></div>
              </div>
            </div>
          </div>

          <div class="config-group">
            <label>Frecuencia de recordatorio</label>
            <select v-model="frecuencia" class="config-input">
              <option value="30">Cada 30 minutos</option>
              <option value="60">Cada hora</option>
              <option value="90">Cada 1.5 horas</option>
              <option value="120">Cada 2 horas</option>
            </select>
          </div>

          <button class="btn-primary" @click="guardar" :disabled="guardando">
            {{ guardando ? 'Guardando...' : 'Guardar Cambios' }}
          </button>
          <p v-if="saved" class="saved-msg">✅ Cambios guardados correctamente</p>
          <p v-if="errorGuardar" class="error-msg">❌ {{ errorGuardar }}</p>
        </div>

        <!-- Calculadora -->
        <div class="card stats-quick-card">
          <h3>💧 Calcula tu consumo ideal</h3>

          <div class="config-group">
            <label>Edad</label>
            <input type="number" v-model.number="edadCalc" class="config-input" min="1" />
          </div>

          <div class="config-group">
            <label>Peso (kg)</label>
            <input type="number" v-model.number="pesoCalc" class="config-input" min="1" />
          </div>

          <div class="check-group">
            <label class="check-item">
              <input type="checkbox" v-model="haceEjercicio">
              <span>🏃 Ejercicio (+500ml)</span>
            </label>

            <label class="check-item">
              <input type="checkbox" v-model="haceCalor">
              <span>☀️ Calor (+500ml)</span>
            </label>
          </div>

          <p v-if="errorCalculo" class="error-msg">⚠️ {{ errorCalculo }}</p>
          <button class="btn-primary" @click="calcularAgua">Calcular</button>

          <div v-if="resultadoLitros" class="resultado-box">
            <p class="resultado-title">META DIARIA:</p>
            <h2>{{ resultadoLitros }} Litros</h2>
            <p class="resultado-msg">
              El resultado de esta prueba no exime de consultar a un médico o profesional especializado.
            </p>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'PerfilView',
  props: {
    currentUser: { type: Object, default: () => ({}) },
    dailyGoal:   { type: Number, default: 2000 },
    racha:       { type: Number, default: 0 },
    waterToday:  { type: Number, default: 0 }
  },
  emits: ['logout', 'meta-actualizada'],
  data() {
    return {
      // Valor CONFIRMADO (guardado en DB) — se muestra en la tarjeta izquierda
      metaDiariaGuardada: 2000,
      // Valor del INPUT — solo se aplica al pulsar "Guardar Cambios"
      metaDiariaInput: 2000,

      notificaciones: true,
      frecuencia: '60',
      saved: false,
      guardando: false,
      errorGuardar: '',

      edadCalc: '',
      pesoCalc: '',
      haceEjercicio: false,
      haceCalor: false,
      resultadoLitros: null,
      errorCalculo: ''
    }
  },
  computed: {
    // Misma lógica de logros que HomeView
    logrosDesbloqueados() {
      const p = this.dailyGoal > 0
        ? Math.round((this.waterToday / this.dailyGoal) * 100)
        : 0
      let count = 0
      if (this.waterToday > 0) count++
      if (p >= 10)  count++
      if (p >= 50)  count++
      if (p >= 70)  count++
      if (p >= 100) count++
      if (p >= 120) count++
      return count
    },
    fechaRegistroFormateada() {
      if (!this.currentUser.fechaRegistro) return '—'
      const d = new Date(this.currentUser.fechaRegistro)
      return d.toLocaleDateString('es-CO', {
        day: '2-digit', month: 'short', year: 'numeric'
      })
    }
  },
  watch: {
    // Se dispara cuando App.vue termina de cargar el perfil desde /me
    'currentUser.metaConsumoMl': {
      immediate: true,
      handler(val) {
        const meta = (val && val > 0) ? val : this.dailyGoal
        this.metaDiariaGuardada = meta
        this.metaDiariaInput    = meta
      }
    },
    dailyGoal: {
      immediate: true,
      handler(val) {
        if (!this.currentUser.metaConsumoMl || this.currentUser.metaConsumoMl === 0) {
          this.metaDiariaGuardada = val
          this.metaDiariaInput    = val
        }
      }
    }
  },
  methods: {
    async guardar() {
      this.errorGuardar = ''
      this.guardando = true
      try {
        const token  = localStorage.getItem('token')
        const userId = localStorage.getItem('userId')

        // Ruta correcta del backend: PUT /usuarios/:id/meta
        await axios.put(
          `http://'+process.env.VITE_API_HOST+':3000/usuarios/${userId}/meta`,
          { metaConsumoMl: this.metaDiariaInput },
          { headers: { 'x-auth-token': token } }
        )

        // Solo aquí se actualiza el valor del recuadro izquierdo
        this.metaDiariaGuardada = this.metaDiariaInput

        // Propaga el nuevo dailyGoal a Home, Semana, Estadísticas y Consultar
        this.$emit('meta-actualizada', Number(this.metaDiariaInput))

        this.saved = true
        setTimeout(() => { this.saved = false }, 3000)
      } catch (error) {
        console.error('Error al guardar meta:', error)
        this.errorGuardar = 'No se pudo guardar. Intenta de nuevo.'
        setTimeout(() => { this.errorGuardar = '' }, 4000)
      } finally {
        this.guardando = false
      }
    },

    calcularAgua() {
      this.errorCalculo = ''

      // Ambos campos son obligatorios y deben ser positivos
      if (!this.edadCalc || !this.pesoCalc || Number(this.edadCalc) <= 0 || Number(this.pesoCalc) <= 0) {
        this.errorCalculo = 'Debes ingresar tu edad y peso para calcular.'
        this.resultadoLitros = null
        return
      }

      let factor = 35
      if (this.edadCalc <= 30)      factor = 40
      else if (this.edadCalc <= 55) factor = 35
      else if (this.edadCalc <= 65) factor = 30
      else                          factor = 25

      let total = this.pesoCalc * factor
      if (this.haceEjercicio) total += 500
      if (this.haceCalor)     total += 500

      this.resultadoLitros = (total / 1000).toFixed(2)

      // Copia el resultado al campo de configuración (en ml) listo para que el usuario guarde
      this.metaDiariaInput = Math.round(total)
    }
  }
}
</script>

<style scoped>
.perfil-view {
  padding: 24px 32px;
  min-height: calc(100vh - 84px);
}
.perfil-container {
  max-width: 1000px;
  margin: 0 auto;
}
.section-title {
  font-family: 'Poppins', sans-serif;
  font-size: 1.5rem;
  font-weight: 700;
  color: #1a2a3a;
  margin-bottom: 24px;
}

.perfil-grid {
  display: grid;
  grid-template-columns: 280px 1fr;
  grid-template-rows: auto auto;
  gap: 18px;
}

.card {
  background: white;
  border-radius: 20px;
  padding: 24px;
  box-shadow: 0 6px 24px rgba(79,172,254,0.1);
}
.card h3 {
  font-family: 'Poppins', sans-serif;
  font-size: 1rem;
  font-weight: 600;
  color: #1a2a3a;
  margin-bottom: 18px;
}

.perfil-card {
  grid-row: span 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 12px;
}
.avatar {
  width: 88px;
  height: 88px;
  background: linear-gradient(135deg, #4facfe, #00c6ff);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2.6rem;
}
.perfil-name {
  font-family: 'Poppins', sans-serif;
  font-size: 1.2rem;
  font-weight: 700;
  color: #1a2a3a;
  margin: 0;
}
.perfil-id {
  font-size: 0.85rem;
  color: #6b8399;
  font-weight: 600;
}

.perfil-info-list {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 8px 0;
}
.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 14px;
  background: #f5fbff;
  border-radius: 12px;
  font-size: 0.88rem;
}
.info-label { color: #6b8399; font-weight: 600; }
.info-val { font-weight: 700; color: #1a2a3a; }
.info-val.flame { color: #ff6b35; }

.btn-danger {
  width: 100%;
  padding: 12px;
  background: #ff4757;
  color: white;
  border: none;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  cursor: pointer;
  margin-top: auto;
}

.config-group {
  margin-bottom: 18px;
}
.config-group label {
  display: block;
  font-size: 0.83rem;
  font-weight: 700;
  color: #6b8399;
  margin-bottom: 8px;
}
.config-input {
  width: 100%;
  padding: 11px 14px;
  border: 2px solid #e0eeff;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-size: 0.95rem;
}
.config-input:focus {
  border-color: #4facfe;
  outline: none;
}

.toggle-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.9rem;
  color: #1a2a3a;
  font-weight: 600;
}
.toggle {
  width: 48px;
  height: 26px;
  background: #d0e0ee;
  border-radius: 100px;
  cursor: pointer;
  position: relative;
}
.toggle.on { background: #4facfe; }
.toggle-thumb {
  position: absolute;
  top: 3px;
  left: 3px;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
}
.toggle.on .toggle-thumb { left: 25px; }

.btn-primary {
  width: 100%;
  padding: 12px;
  background: linear-gradient(135deg, #4facfe, #00c6ff);
  color: white;
  border: none;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  cursor: pointer;
}
.btn-primary:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}

.saved-msg {
  font-size: 0.85rem;
  color: #00c853;
  font-weight: 700;
  margin-top: 10px;
  text-align: center;
}
.error-msg {
  font-size: 0.85rem;
  color: #ff4757;
  font-weight: 700;
  margin-top: 0;
  margin-bottom: 10px;
  text-align: center;
}

.stats-quick-card {
  grid-column: 2;
}
.check-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 18px;
}
.check-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.92rem;
  color: #1a2a3a;
  font-weight: 600;
}
.resultado-box {
  margin-top: 18px;
  background: #eef7ff;
  border: 1px solid #cfe6ff;
  border-radius: 16px;
  padding: 20px;
  text-align: center;
}
.resultado-title {
  font-size: 0.85rem;
  font-weight: 800;
  color: #6b8399;
  margin-bottom: 8px;
}
.resultado-box h2 {
  color: #1e88e5;
  font-size: 2rem;
  margin: 0;
  font-family: 'Poppins', sans-serif;
}
.resultado-msg {
  margin-top: 14px;
  font-size: 0.82rem;
  color: #7b8ea3;
}
</style>