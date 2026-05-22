<template>
  <div class="estadisticas-view">
    <div class="estadisticas-container">
      <h2 class="section-title">📊 Estadísticas</h2>
 
      <div class="stats-grid">
 
        <!-- Avance general - Donut SVG -->
        <div class="card">
          <h3>Avance General (Semana)</h3>
          <div class="donut-wrap">
            <svg viewBox="0 0 160 160" class="donut-svg">
              <circle cx="80" cy="80" r="60" fill="none" stroke="#e0eeff" stroke-width="18"/>
              <circle
                cx="80" cy="80" r="60"
                fill="none"
                stroke="#4facfe"
                stroke-width="18"
                stroke-linecap="round"
                stroke-dasharray="376.99"
                :stroke-dashoffset="376.99 - (376.99 * Math.min(goalPercent, 100) / 100)"
                transform="rotate(-90 80 80)"
                style="transition: stroke-dashoffset 0.8s ease"
              />
            </svg>
            <div class="donut-center">
              <span class="donut-value">{{ goalPercent }}%</span>
              <span class="donut-label">completado</span>
            </div>
          </div>
          <p class="card-desc">
            {{ daysMetGoal }} de 7 días en que cumpliste tu meta esta semana
          </p>
        </div>
 
        <!-- Consumo Semana - Donut SVG -->
        <div class="card">
          <h3>Consumo Semana</h3>
          <div class="donut-wrap">
            <svg viewBox="0 0 160 160" class="donut-svg">
              <circle cx="80" cy="80" r="60" fill="none" stroke="#e0eeff" stroke-width="18"/>
              <circle
                cx="80" cy="80" r="60"
                fill="none"
                stroke="#00c6ff"
                stroke-width="18"
                stroke-linecap="round"
                stroke-dasharray="376.99"
                :stroke-dashoffset="376.99 - (376.99 * Math.min(weekPercent, 100) / 100)"
                transform="rotate(-90 80 80)"
                style="transition: stroke-dashoffset 0.8s ease"
              />
            </svg>
            <div class="donut-center">
              <span class="donut-value" style="color:#00c6ff">{{ weekPercent }}%</span>
              <span class="donut-label">del objetivo</span>
            </div>
          </div>
          <p class="card-desc">
            Consumiste {{ weekTotalConsumed.toLocaleString() }} ml de {{ weekTotalGoal.toLocaleString() }} ml totales de la semana
          </p>
        </div>
 
        <!-- Hoy -->
        <div class="card">
          <h3>💧 Hoy</h3>
          <div class="donut-wrap">
            <svg viewBox="0 0 160 160" class="donut-svg">
              <circle cx="80" cy="80" r="60" fill="none" stroke="#e0eeff" stroke-width="18"/>
              <circle
                cx="80" cy="80" r="60"
                fill="none"
                stroke="#00c853"
                stroke-width="18"
                stroke-linecap="round"
                stroke-dasharray="376.99"
                :stroke-dashoffset="376.99 - (376.99 * Math.min(todayPercent, 100) / 100)"
                transform="rotate(-90 80 80)"
                style="transition: stroke-dashoffset 0.8s ease"
              />
            </svg>
            <div class="donut-center">
              <span class="donut-value" style="color:#00c853">{{ todayPercent }}%</span>
              <span class="donut-label">hoy</span>
            </div>
          </div>
          <p class="card-desc">{{ waterToday }} ml de {{ dailyGoal }} ml consumidos hoy</p>
        </div>
 
        <!-- Mejor día -->
        <div class="card highlight-card">
          <h3>🏅 Mejor Día</h3>
          <p class="big-stat">{{ bestDay.name }}</p>
          <p class="stat-sub">{{ bestDay.consumed.toLocaleString() }} ml — {{ bestDay.percent }}% de la meta</p>
        </div>
 
        <!-- Racha -->
        <div class="card highlight-card">
          <h3>🔥 Racha</h3>
          <p class="big-stat">{{ racha }} {{ racha === 1 ? 'día' : 'días' }}</p>
          <p class="stat-sub">Consecutivos cumpliendo la meta</p>
        </div>
 
        <!-- Total semana -->
        <div class="card highlight-card">
          <h3>📦 Total Semanal</h3>
          <p class="big-stat">{{ weekTotalConsumed.toLocaleString() }} ml</p>
          <p class="stat-sub">Promedio: {{ avgDiario.toLocaleString() }} ml/día</p>
        </div>
 
      </div>
    </div>
  </div>
</template>
 
<script>
import axios from 'axios'
 
export default {
  name: 'EstadisticasView',
  props: {
    waterToday: { type: Number, default: 0 },
    dailyGoal:  { type: Number, default: 2000 }
  },
  data() {
    return {
      racha: 0,
      weeklyData: this.buildEmptyWeek()
    }
  },
  computed: {
    todayPercent() {
      return Math.round((this.waterToday / this.dailyGoal) * 100)
    },
    // Días en que se cumplió la meta
    daysMetGoal() {
      return this.weeklyData.filter(d => d.consumed >= this.dailyGoal).length
    },
    // % de días que cumplieron la meta (Avance General)
    goalPercent() {
      return Math.round((this.daysMetGoal / 7) * 100)
    },
    // Total consumido en la semana
    weekTotalConsumed() {
      return this.weeklyData.reduce((sum, d) => sum + d.consumed, 0)
    },
    // Meta total de la semana
    weekTotalGoal() {
      return this.dailyGoal * 7
    },
    // % de consumo semanal respecto a la meta total
    weekPercent() {
      if (this.weekTotalGoal === 0) return 0
      return Math.round((this.weekTotalConsumed / this.weekTotalGoal) * 100)
    },
    // Mejor día de la semana
    bestDay() {
      if (!this.weeklyData.length) return { name: '—', consumed: 0, percent: 0 }
      const best = this.weeklyData.reduce((a, b) => a.consumed >= b.consumed ? a : b)
      return {
        name:     best.consumed > 0 ? best.day : '—',
        consumed: best.consumed,
        percent:  best.percent
      }
    },
    // Promedio diario (solo días con datos)
    avgDiario() {
      const daysWithData = this.weeklyData.filter(d => d.consumed > 0)
      if (!daysWithData.length) return 0
      const total = daysWithData.reduce((s, d) => s + d.consumed, 0)
      return Math.round(total / daysWithData.length)
    }
  },
  mounted() {
    this.fetchEstadisticas()
    this._pollingInterval = setInterval(this.fetchEstadisticas, 30000)
  },
  beforeUnmount() {
    clearInterval(this._pollingInterval)
  },
  methods: {
    getMondayOfWeek(date) {
      const d = new Date(date)
      const day = d.getDay()
      const diff = day === 0 ? -6 : 1 - day
      d.setDate(d.getDate() + diff)
      d.setHours(0, 0, 0, 0)
      return d
    },
    buildEmptyWeek() {
      const DIAS = [
        { day: 'Lunes',     short: 'Lun' },
        { day: 'Martes',    short: 'Mar' },
        { day: 'Miércoles', short: 'Mié' },
        { day: 'Jueves',    short: 'Jue' },
        { day: 'Viernes',   short: 'Vie' },
        { day: 'Sábado',    short: 'Sáb' },
        { day: 'Domingo',   short: 'Dom' },
      ]
      return DIAS.map(d => ({ ...d, consumed: 0, percent: 0 }))
    },
    async fetchEstadisticas() {
      try {
        const token  = localStorage.getItem('token')
        const userId = localStorage.getItem('userId')
        if (!token || !userId) return
 
        // Traer consumos de la semana y racha en paralelo
        const [semanaRes, rachaRes] = await Promise.all([
          axios.get(
            `http://${process.env.VUE_APP_API_HOST}:3000/usuarios/${userId}/consumos/semana`,
            { headers: { 'x-auth-token': token } }
          ),
          axios.get(
            `http://${process.env.VUE_APP_API_HOST}:3000/usuarios/${userId}/racha`,
            { headers: { 'x-auth-token': token } }
          )
        ])
 
        this.racha = rachaRes.data.racha ?? 0
        const consumos = semanaRes.data.msg || []
        this.procesarConsumos(consumos)
      } catch (error) {
        console.error('Error al obtener estadísticas:', error)
      }
    },
    procesarConsumos(consumos) {
      const hoy   = new Date()
      const lunes = this.getMondayOfWeek(hoy)
 
      const DIAS = [
        { day: 'Lunes',     short: 'Lun' },
        { day: 'Martes',    short: 'Mar' },
        { day: 'Miércoles', short: 'Mié' },
        { day: 'Jueves',    short: 'Jue' },
        { day: 'Viernes',   short: 'Vie' },
        { day: 'Sábado',    short: 'Sáb' },
        { day: 'Domingo',   short: 'Dom' },
      ]
 
      this.weeklyData = DIAS.map((d, i) => {
        const diaFecha = new Date(lunes)
        diaFecha.setDate(lunes.getDate() + i)
 
        const inicioDia = new Date(diaFecha)
        inicioDia.setHours(0, 0, 0, 0)
        const finDia = new Date(diaFecha)
        finDia.setHours(23, 59, 59, 999)
 
        // Días futuros → ceros
        if (inicioDia > hoy) {
          return { ...d, consumed: 0, percent: 0 }
        }
 
        const delDia  = consumos.filter(c => {
          const f = new Date(c.fechaHora)
          return f >= inicioDia && f <= finDia
        })
        const consumed = delDia.reduce((s, c) => s + c.consumoInstante, 0)
        const percent  = Math.round((consumed / this.dailyGoal) * 100)
 
        return { ...d, consumed, percent }
      })
    },
    statusColor(p) {
      if (p >= 100) return '#00c853'
      if (p >= 80) return '#ffb300'
      return '#ff5252'
    },
    statusClass(p) {
      if (p >= 100) return 'green'
      if (p >= 80) return 'yellow'
      return 'red'
    }
  }
}
</script>
 
<style scoped>
.estadisticas-view {
  padding: 24px 32px;
  min-height: calc(100vh - 84px);
}
.estadisticas-container {
  max-width: 1100px;
  margin: 0 auto;
}
.section-title {
  font-family: 'Poppins', sans-serif;
  font-size: 1.5rem;
  font-weight: 700;
  color: #1a2a3a;
  margin-bottom: 24px;
}
 
.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
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
  font-size: 0.95rem;
  font-weight: 600;
  color: #1a2a3a;
  margin-bottom: 14px;
}
 
/* Donut */
.donut-wrap {
  position: relative;
  width: 160px;
  height: 160px;
  margin: 0 auto 12px;
}
.donut-svg { width: 100%; height: 100%; }
.donut-center {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}
.donut-value {
  display: block;
  font-size: 1.8rem;
  font-weight: 900;
  color: #4facfe;
  line-height: 1;
}
.donut-label {
  font-size: 0.72rem;
  color: #6b8399;
  font-weight: 600;
}
.card-desc {
  font-size: 0.82rem;
  color: #6b8399;
  text-align: center;
  line-height: 1.4;
}
 
/* Highlight cards */
.highlight-card { text-align: center; }
.big-stat { font-size: 2rem; font-weight: 900; color: #4facfe; margin: 10px 0 6px; }
.stat-sub { font-size: 0.85rem; color: #6b8399; }
 
/* Table */
.table-card { margin-top: 4px; }
.stats-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}
.stats-table th {
  text-align: left;
  padding: 10px 14px;
  background: #f0f8ff;
  color: #6b8399;
  font-weight: 700;
  font-size: 0.82rem;
  border-radius: 8px;
}
.stats-table td {
  padding: 12px 14px;
  border-bottom: 1px solid #f0f5ff;
  color: #1a2a3a;
}
.stats-table tr:last-child td { border-bottom: none; }
 
.status-badge {
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 0.78rem;
  font-weight: 700;
}
.status-badge.green { background: #e8fdf0; color: #00a040; }
.status-badge.yellow { background: #fff8e1; color: #d08000; }
.status-badge.red { background: #fff0f0; color: #d32f2f; }
</style>