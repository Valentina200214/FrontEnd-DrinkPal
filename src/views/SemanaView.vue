<template>
  <div class="semana-view">
    <div class="semana-container">
      <h2 class="section-title">📅 Consumo Semanal</h2>

      <!-- Resumen semanal -->
      <div class="week-summary">
        <div class="summary-card" v-for="day in weeklyData" :key="day.short">
          <div class="day-short">{{ day.short }}</div>
          <div class="day-bar-wrap">
            <div
              class="day-bar-fill"
              :style="{ height: Math.min(day.percent, 110) + '%', background: barColor(day.percent) }"
            ></div>
          </div>
          <div class="day-percent" :style="{ color: barColor(day.percent) }">{{ day.percent }}%</div>
        </div>
      </div>

      <!-- Lista expandible de días -->
      <div class="days-list">
        <div
          v-for="(day, index) in weeklyData"
          :key="index"
          class="day-item"
        >
          <div class="day-header" @click="toggleDay(index)">
            <div class="day-info">
              <span class="day-name">{{ day.day }}</span>
              <span class="day-consumed" :style="{ color: barColor(day.percent) }">
                {{ day.consumed }} ml ({{ day.percent }}%)
              </span>
            </div>
            <div class="day-status">
              <span v-if="day.percent >= 100" class="badge green">✅ Meta</span>
              <span v-else-if="day.percent >= 80" class="badge yellow">🔶 Casi</span>
              <span v-else class="badge red">❌ Incompleto</span>
              <span class="chevron" :class="{ open: openDay === index }">▾</span>
            </div>
          </div>

          <!-- Detalle expandido -->
          <div v-if="openDay === index" class="day-details">
            <p class="detail-subtitle">Consumo por franjas horarias</p>
            <div class="hourly-bars">
              <div
                v-for="(val, hi) in day.hourly"
                :key="hi"
                class="hourly-col"
              >
                <div class="hourly-bar-bg">
                  <div
                    class="hourly-bar-fill"
                    :style="{ height: (maxHourly(day.hourly) > 0 ? (val / maxHourly(day.hourly)) * 100 : 0) + '%' }"
                  ></div>
                </div>
                <span class="hourly-label">{{ hourLabels[hi] }}</span>
                <span class="hourly-val">{{ val }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'SemanaView',
  props: {
    dailyGoal: { type: Number, default: 2000 }
  },
  data() {
    return {
      openDay: null,
      hourLabels: ['12am','2am','4am','6am','8am','10am','12pm','2pm','4pm','6pm','8pm','10pm'],
      weeklyData: this.buildEmptyWeek(),
      // Guardamos los consumos crudos para poder reprocesar si cambia dailyGoal
      consumosCrudos: []
    }
  },
  watch: {
    // Cuando el usuario cambia la meta desde Perfil, recalcula porcentajes al instante
    dailyGoal() {
      this.procesarConsumos(this.consumosCrudos)
    }
  },
  mounted() {
    this.fetchSemanaData()
    this._pollingInterval = setInterval(this.fetchSemanaData, 30000)
  },
  beforeUnmount() {
    clearInterval(this._pollingInterval)
  },
  methods: {
    toggleDay(index) {
      this.openDay = this.openDay === index ? null : index
    },
    barColor(percent) {
      if (percent >= 100) return '#00c853'
      if (percent >= 80)  return '#ffb300'
      return '#ff5252'
    },
    maxHourly(arr) {
      const m = Math.max(...arr)
      return m > 0 ? m : 1
    },
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
      return DIAS.map(d => ({ ...d, consumed: 0, goal: 2000, percent: 0, hourly: new Array(12).fill(0) }))
    },
    async fetchSemanaData() {
      try {
        const token  = localStorage.getItem('token')
        const userId = localStorage.getItem('userId')
        if (!token || !userId) return

        const response = await axios.get(
          `http://'+process.env.VITE_API_HOST+':3000/usuarios/${userId}/consumos/semana`,
          { headers: { 'x-auth-token': token } }
        )

        // Guardamos los crudos para poder recalcular si cambia la meta
        this.consumosCrudos = response.data.msg || []
        this.procesarConsumos(this.consumosCrudos)
      } catch (error) {
        console.error('Error al obtener consumos de la semana:', error)
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

        if (inicioDia > hoy) {
          return { ...d, consumed: 0, goal: this.dailyGoal, percent: 0, hourly: new Array(12).fill(0) }
        }

        const delDia = consumos.filter(c => {
          const f = new Date(c.fechaHora)
          return f >= inicioDia && f <= finDia
        })

        const consumed = delDia.reduce((s, c) => s + c.consumoInstante, 0)
        // Usa this.dailyGoal (reactivo) en lugar del valor hardcodeado
        const percent  = Math.round((consumed / this.dailyGoal) * 100)

        const hourly = new Array(12).fill(0)
        delDia.forEach(c => {
          const hora = new Date(c.fechaHora).getHours()
          const slot = Math.floor(hora / 2)
          hourly[slot] += c.consumoInstante
        })

        return { ...d, consumed, goal: this.dailyGoal, percent, hourly }
      })
    }
  }
}
</script>

<style scoped>
.semana-view {
  padding: 24px 32px;
  min-height: calc(100vh - 84px);
}

.semana-container {
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

/* ===== WEEK SUMMARY BARS ===== */
.week-summary {
  background: white;
  border-radius: 20px;
  padding: 24px;
  box-shadow: 0 6px 24px rgba(79,172,254,0.1);
  display: flex;
  justify-content: space-around;
  align-items: flex-end;
  height: 180px;
  margin-bottom: 28px;
}

.summary-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100%;
  gap: 6px;
}

.day-short {
  font-size: 0.78rem;
  font-weight: 700;
  color: #6b8399;
}

.day-bar-wrap {
  flex: 1;
  width: 32px;
  background: #e8f4ff;
  border-radius: 6px;
  display: flex;
  align-items: flex-end;
  overflow: hidden;
}

.day-bar-fill {
  width: 100%;
  border-radius: 6px;
  transition: height 0.6s ease;
  min-height: 4px;
}

.day-percent {
  font-size: 0.75rem;
  font-weight: 700;
}

/* ===== DAYS LIST ===== */
.days-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.day-item {
  background: white;
  border-radius: 18px;
  box-shadow: 0 4px 16px rgba(79,172,254,0.08);
  overflow: hidden;
}

.day-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 18px 22px;
  cursor: pointer;
  transition: background 0.15s;
}
.day-header:hover { background: #f5fbff; }

.day-info { display: flex; flex-direction: column; gap: 3px; }
.day-name { font-weight: 700; font-size: 1rem; color: #1a2a3a; }
.day-consumed { font-size: 0.9rem; font-weight: 600; }

.day-status { display: flex; align-items: center; gap: 10px; }

.badge {
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 0.78rem;
  font-weight: 700;
}
.badge.green { background: #e8fdf0; color: #00a040; }
.badge.yellow { background: #fff8e1; color: #d08000; }
.badge.red { background: #fff0f0; color: #d32f2f; }

.chevron {
  font-size: 1.1rem;
  color: #8aabbb;
  transition: transform 0.25s;
}
.chevron.open { transform: rotate(180deg); }

/* ===== DAY DETAILS ===== */
.day-details {
  padding: 18px 22px;
  border-top: 1px solid #e8f4ff;
  background: #f8fbff;
}
.detail-subtitle {
  font-size: 0.85rem;
  font-weight: 600;
  color: #6b8399;
  margin-bottom: 14px;
}

.hourly-bars {
  display: flex;
  gap: 6px;
  align-items: flex-end;
  height: 140px;
}

.hourly-col {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  height: 100%;
}

.hourly-bar-bg {
  flex: 1;
  width: 100%;
  background: #d0e8ff;
  border-radius: 6px;
  display: flex;
  align-items: flex-end;
  overflow: hidden;
}

.hourly-bar-fill {
  width: 100%;
  background: linear-gradient(180deg, #4facfe, #00c6ff);
  border-radius: 6px;
  transition: height 0.5s ease;
  min-height: 3px;
}

.hourly-label {
  font-size: 0.62rem;
  color: #8aabbb;
  font-weight: 600;
}
.hourly-val {
  font-size: 0.62rem;
  color: #4facfe;
  font-weight: 700;
}
</style>