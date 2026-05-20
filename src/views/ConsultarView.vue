<template>
  <div class="consultar-view">
    <div class="consultar-container">
      <h2 class="section-title">🔍 Consultar Consumo</h2>
 
      <div class="consultar-grid">
 
        <!-- Formulario de consulta -->
        <div class="card consulta-form-card">
          <h3>Selecciona un rango de fechas</h3>
          <div class="form-row">
            <div class="input-group">
              <label>Fecha inicio</label>
              <input type="date" v-model="fechaInicio" class="date-input" />
            </div>
            <div class="input-group">
              <label>Fecha fin</label>
              <input type="date" v-model="fechaFin" class="date-input" />
            </div>
          </div>
          <button class="btn-primary" @click="consultar" :disabled="cargando">
            {{ cargando ? 'Consultando...' : 'Consultar' }}
          </button>
        </div>
 
        <!-- Resultado -->
        <div v-if="resultado" class="card resultado-card">
          <h3>📋 Resultados del período</h3>
          <p class="periodo-label">{{ fechaInicio }} → {{ fechaFin }}</p>
 
          <div class="resultado-stats">
            <div class="resultado-item">
              <span class="resultado-icon">💧</span>
              <div>
                <span class="resultado-val">{{ resultado.total }}</span>
                <span class="resultado-lbl">Total consumido</span>
              </div>
            </div>
            <div class="resultado-item">
              <span class="resultado-icon">📅</span>
              <div>
                <span class="resultado-val">{{ resultado.promedio }}</span>
                <span class="resultado-lbl">Promedio diario</span>
              </div>
            </div>
            <div class="resultado-item">
              <span class="resultado-icon">🏆</span>
              <div>
                <span class="resultado-val">{{ resultado.diasMeta }}</span>
                <span class="resultado-lbl">Días con meta cumplida</span>
              </div>
            </div>
            <div class="resultado-item">
              <span class="resultado-icon">📈</span>
              <div>
                <span class="resultado-val">{{ resultado.porcentaje }}</span>
                <span class="resultado-lbl">Eficiencia general</span>
              </div>
            </div>
          </div>
 
          <!-- Barra visual -->
          <div class="resultado-bar-wrap">
            <div class="resultado-bar-bg">
              <div class="resultado-bar-fill" :style="{ width: Math.min(resultado.rawPct, 100) + '%' }"></div>
            </div>
            <span class="resultado-bar-label">{{ resultado.rawPct }}% de los días con meta cumplida</span>
          </div>
        </div>
 
        <!-- Consultas rápidas -->
        <div class="card quick-card">
          <h3>⚡ Consultas Rápidas</h3>
          <div class="quick-buttons">
            <button class="btn-quick" @click="quickQuery(7)">Últimos 7 días</button>
            <button class="btn-quick" @click="quickQuery(14)">Últimas 2 semanas</button>
            <button class="btn-quick" @click="quickQuery(30)">Último mes</button>
            <button class="btn-quick" @click="quickQuery(90)">Últimos 3 meses</button>
          </div>
        </div>
 
      </div>
      
    </div>
  </div>
</template>
 
<script>
import axios from 'axios'
 
export default {
  name: 'ConsultarView',
  data() {
    return {
      fechaInicio: '',
      fechaFin: '',
      resultado: null,
      cargando: false,
      dailyGoal: 2000   // se sobreescribe al montar con el valor real del usuario
    }
  },
  mounted() {
    this.fetchMetaUsuario()
  },
  methods: {
 
    // Trae la meta diaria real del usuario desde /me
    async fetchMetaUsuario() {
      try {
        const token = localStorage.getItem('token')
        if (!token) return
        const res = await axios.get('http://localhost:3000/me', {
          headers: { 'x-auth-token': token }
        })
        if (res.data && res.data.metaConsumoMl > 0) {
          this.dailyGoal = res.data.metaConsumoMl
        }
      } catch (error) {
        console.error('Error al obtener meta del usuario:', error)
      }
    },
 
    async consultar() {
      if (!this.fechaInicio || !this.fechaFin) {
        alert('Por favor selecciona ambas fechas')
        return
      }
      if (this.fechaInicio > this.fechaFin) {
        alert('La fecha inicio no puede ser mayor que la fecha fin')
        return
      }
 
      this.cargando = true
      this.resultado = null
 
      try {
        const token  = localStorage.getItem('token')
        const userId = localStorage.getItem('userId')
        if (!token || !userId) return
 
        const res = await axios.get(
          `http://localhost:3000/usuarios/${userId}/consumos`,
          { headers: { 'x-auth-token': token } }
        )
 
        const consumos = res.data.msg || []
        this.resultado = this.calcularResultados(consumos, this.fechaInicio, this.fechaFin)
 
      } catch (error) {
        console.error('Error al consultar consumos:', error)
        alert('Error al obtener datos. Intenta de nuevo.')
      } finally {
        this.cargando = false
      }
    },
 
    quickQuery(days) {
      const end   = new Date()
      const start = new Date()
      start.setDate(start.getDate() - (days - 1))
      this.fechaFin    = end.toISOString().split('T')[0]
      this.fechaInicio = start.toISOString().split('T')[0]
      this.consultar()
    },
 
    // Filtra los consumos por rango y calcula todas las métricas
    calcularResultados(consumos, fechaInicio, fechaFin) {

      // Convierte YYYY-MM-DD a fecha LOCAL
      const parseLocalDate = (dateStr, endOfDay = false) => {
        const [year, month, day] = dateStr.split('-').map(Number)

        if (endOfDay) {
          return new Date(year, month - 1, day, 23, 59, 59, 999)
        }

        return new Date(year, month - 1, day, 0, 0, 0, 0)
      }

      const inicio = parseLocalDate(fechaInicio)
      const fin    = parseLocalDate(fechaFin, true)

      // Filtrar consumos del rango
      const delPeriodo = consumos.filter(c => {
        const f = new Date(c.fechaHora)
        return f >= inicio && f <= fin
      })

      // Agrupar por día
      const porDia = {}

      delPeriodo.forEach(c => {
        const d = new Date(c.fechaHora)

        const key =
          `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`

        porDia[key] = (porDia[key] || 0) + c.consumoInstante
      })

      // Total de días del rango
      const msPerDay = 1000 * 60 * 60 * 24

      const totalDias =
        Math.floor((parseLocalDate(fechaFin) - parseLocalDate(fechaInicio)) / msPerDay) + 1

      // Totales
      const totalMl = delPeriodo.reduce((s, c) => s + c.consumoInstante, 0)

      const promedioMl =
        totalDias > 0 ? Math.round(totalMl / totalDias) : 0

      // Días donde sí se cumplió la meta
      const diasMeta =
        Object.values(porDia).filter(v => v >= this.dailyGoal).length

      const rawPct =
        totalDias > 0
          ? Math.round((diasMeta / totalDias) * 100)
          : 0

      return {
        total: `${totalMl.toLocaleString('es-CO')} ml`,
        promedio: `${promedioMl.toLocaleString('es-CO')} ml/día`,
        diasMeta: `${diasMeta} de ${totalDias} días`,
        porcentaje: `${rawPct}%`,
        rawPct
      }
    }
  }
}
</script>
 
<style scoped>
.consultar-view {
  padding: 24px 32px;
  min-height: calc(100vh - 84px);
}
.consultar-container {
  max-width: 860px;
  margin: 0 auto;
}
.section-title {
  font-family: 'Poppins', sans-serif;
  font-size: 1.5rem;
  font-weight: 700;
  color: #1a2a3a;
  margin-bottom: 24px;
}
 
.consultar-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 18px;
  margin-bottom: 20px;
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
  margin-bottom: 16px;
}
 
.form-row {
  display: flex;
  gap: 14px;
  margin-bottom: 16px;
}
.input-group {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.input-group label {
  font-size: 0.82rem;
  font-weight: 600;
  color: #6b8399;
}
.date-input {
  padding: 11px 14px;
  border: 2px solid #e0eeff;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-size: 0.93rem;
  transition: border-color 0.2s;
  width: 100%;
}
.date-input:focus { border-color: #4facfe; outline: none; }
 
.btn-primary {
  padding: 12px 24px;
  background: linear-gradient(135deg, #4facfe, #00c6ff);
  color: white;
  border: none;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
  width: 100%;
}
.btn-primary:hover { transform: translateY(-1px); box-shadow: 0 6px 20px rgba(79,172,254,0.35); }
.btn-primary:disabled { opacity: 0.6; cursor: not-allowed; transform: none; box-shadow: none; }
 
/* Resultado */
.periodo-label {
  font-size: 0.85rem;
  color: #6b8399;
  margin-bottom: 16px;
  font-weight: 600;
}
.resultado-stats {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
  margin-bottom: 16px;
}
.resultado-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px;
  background: #f5fbff;
  border-radius: 12px;
}
.resultado-icon { font-size: 1.4rem; }
.resultado-val { display: block; font-weight: 800; font-size: 1.05rem; color: #4facfe; }
.resultado-lbl { font-size: 0.75rem; color: #6b8399; font-weight: 600; }
 
.resultado-bar-wrap { margin-top: 6px; }
.resultado-bar-bg { background: #d0e8ff; border-radius: 100px; height: 12px; overflow: hidden; margin-bottom: 6px; }
.resultado-bar-fill { height: 100%; background: linear-gradient(90deg, #4facfe, #00c6ff); border-radius: 100px; transition: width 0.6s; }
.resultado-bar-label { font-size: 0.78rem; color: #6b8399; font-weight: 600; }
 
/* Quick */
.quick-buttons { display: flex; flex-direction: column; gap: 10px; }
.btn-quick {
  padding: 11px 16px;
  background: #f0f8ff;
  border: 2px solid #d0e8ff;
  color: #0077cc;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
  text-align: left;
}
.btn-quick:hover { background: linear-gradient(135deg, #4facfe, #00c6ff); color: white; border-color: transparent; }
 
/* History table */
.history-card { margin-top: 4px; }
.history-table { width: 100%; border-collapse: collapse; font-size: 0.9rem; }
.history-table th {
  text-align: left; padding: 10px 14px;
  background: #f0f8ff; color: #6b8399;
  font-weight: 700; font-size: 0.82rem;
}
.history-table td {
  padding: 12px 14px;
  border-bottom: 1px solid #f0f5ff;
  color: #1a2a3a;
}
.history-table tr:last-child td { border-bottom: none; }
.badge { padding: 4px 10px; border-radius: 20px; font-size: 0.78rem; font-weight: 700; }
.badge.green { background: #e8fdf0; color: #00a040; }
.badge.yellow { background: #fff8e1; color: #d08000; }
.badge.red { background: #fff0f0; color: #d32f2f; }
</style>