<template>
  <div id="app-wrapper">
    <!-- LOGIN SCREEN -->
    <div v-if="!isLoggedIn" class="login-screen">
      <div class="login-card">
        <div class="login-logo">
          <img :src="require('@/assets/escudo.png')" alt="DrinkPal Logo" class="login-logo-img" />
          <h1>DrinkPal</h1>
          <p>Tu compañero inteligente de hidratación</p>
        </div>

        <div class="tab-switcher">
          <button :class="['tab-btn', activeTab === 'login' ? 'active' : '']" @click="activeTab = 'login'">
            Iniciar Sesión
          </button>
          <button :class="['tab-btn', activeTab === 'register' ? 'active' : '']" @click="activeTab = 'register'">
            Registrarse
          </button>
        </div>

        <div v-if="activeTab === 'login'" class="form-group">
          <input v-model="loginUser" type="text" placeholder="Usuario o Email" />
          <input v-model="loginPass" type="password" placeholder="Contraseña" />
          <button class="btn-primary full-width" @click="login_fcn()">Ingresar</button>
        
        <div class="mensaje-error">
          <p>{{ mensajelogin }}</p>
        </div>
        
        </div>

        <div v-if="activeTab === 'register'" class="form-group">
          <input v-model="regName" type="text" placeholder="Nombre completo*" />
          <input v-model="regUser" type="text" placeholder="Correo*" />
          <input v-model="regId" type="text" placeholder="ID del Producto*" />
          <input v-model="regPass" type="password" placeholder="Contraseña*" />
          <input v-model="regTel" type="text" placeholder="Telefono" />
          <input v-model="regUbi" type="text" placeholder="Localidad" />
          <button class="btn-primary full-width" @click="registro_fcn()">Crear Cuenta</button>
        
        <div class="mensaje-error">
          <p>{{ mensajeRegistro }}</p>
        </div>
        
        </div>

      </div>
    </div>

    <!-- APP ADMIN -->
    <div v-if="isLoggedIn && isAdmin" class="main-app">
      <AdminView @logout="logout" />
    </div>

    <!-- APP PRINCIPAL (usuarios normales) -->
    <div v-if="isLoggedIn && !isAdmin" class="main-app">

      <!-- HEADER -->
      <header class="app-header">
        <div class="header-logo">
          <img :src="require('@/assets/escudo.png')" alt="Logo" class="header-logo-img" />
        </div>

        <nav class="header-nav">
          <button
            v-for="tab in tabs"
            :key="tab.id"
            :class="['nav-link', currentScreen === tab.id ? 'active' : '']"
            @click="currentScreen = tab.id"
          >
            {{ tab.label }}
          </button>
        </nav>

        <div class="header-right">
          <div class="header-user">
            <span class="user-name">{{ currentUser.name }}</span>
            <div class="header-actions">
              <button class="btn-outline small" @click="currentScreen = 'perfil'">Perfil</button>
              <button class="btn-danger small" @click="logout">Salir</button>
            </div>
          </div>
          <div class="battery-section">
            <span class="battery-label">Dispositivo</span>
            <img :src="batteryImage" alt="Batería" class="battery-icon" />
          </div>
        </div>
      </header>

      <main class="app-content">
        <HomeView
          v-if="currentScreen === 'inicio'"
          :waterToday="waterToday"
          :dailyGoal="dailyGoal"
          :currentUser="currentUser"
          :racha="racha"
          @agua-registrada="onAguaRegistrada"
        />
        <SemanView v-else-if="currentScreen === 'semana'" :dailyGoal="dailyGoal" />
        <EstadisticasView
          v-else-if="currentScreen === 'estadisticas'"
          :waterToday="waterToday"
          :dailyGoal="dailyGoal"
        />
        <ConsultarView v-else-if="currentScreen === 'consultar'" />
        <PerfilView
          v-else-if="currentScreen === 'perfil'"
          :currentUser="currentUser"
          :dailyGoal="dailyGoal"
          :racha="racha"
          :waterToday="waterToday"
          @logout="logout"
          @meta-actualizada="onMetaActualizada"
        />
      </main>
    </div>
  </div>
</template>

<script>
import HomeView from './views/HomeView.vue'
import SemanView from './views/SemanaView.vue'
import EstadisticasView from './views/EstadisticasView.vue'
import ConsultarView from './views/ConsultarView.vue'
import PerfilView from './views/PerfilView.vue'
import AdminView from './views/AdminView.vue'
import axios from 'axios'

export default {
  name: 'App',
  components: { HomeView, SemanView, EstadisticasView, ConsultarView, PerfilView, AdminView },
  data() {
    return {
      isLoggedIn: false,
      isAdmin: false,
      activeTab: 'login',
      currentScreen: 'inicio',
      // currentUser ahora incluye los campos extendidos del perfil
      currentUser: { name: '', id: '', id_producto: '', fechaRegistro: null, metaConsumoMl: 0 },
      waterToday: 0,
      dailyGoal: 2000,
      nivelBateria: this.leerBateria(),
      loginId: '', loginUser: '', loginPass: '',
      regName: '', regUser: '', regId: '', regPass: '', regTel: '', regUbi: '',
      mensajelogin: "", mensajeRegistro: "",
      racha: 0,
      tabs: [
        { id: 'inicio',        label: 'Inicio' },
        { id: 'semana',        label: 'Semana' },
        { id: 'estadisticas',  label: 'Estadísticas' },
        { id: 'consultar',     label: 'Consultar' },
      ]
    }
  },
  computed: {
    batteryImage() {
      if (this.nivel < 10) return require('@/assets/Iconsmind-Outline-Battery-0.512.png')
      if (this.nivel < 30) return require('@/assets/Iconsmind-Outline-Battery-25.512.png')
      if (this.nivel < 60) return require('@/assets/Iconsmind-Outline-Battery-50.512.png')
      if (this.nivel < 80) return require('@/assets/Iconsmind-Outline-Battery-75.512.png')
      return require('@/assets/Iconsmind-Outline-Battery-100.512.png')
    }
  },
  mounted() {
    this.leerBateria()
  },
  methods: {
    login() {
      this.currentUser = {
        name: this.loginUser || 'Usuario DrinkPal',
        id: this.loginId || 'DP-123456',
        id_producto: '', fechaRegistro: null, metaConsumoMl: 0
      }
      this.isLoggedIn = true
      this.currentScreen = 'inicio'
    },
    register() {
      this.currentUser = {
        name: this.regName || 'Usuario DrinkPal',
        id: this.regId || 'DP-123456',
        id_producto: '', fechaRegistro: null, metaConsumoMl: 0
      }
      this.isLoggedIn = true
      this.currentScreen = 'inicio'
    },
    logout() {
      clearInterval(this._pollingInterval)
      this._pollingInterval = null
      this.isLoggedIn = false
      this.isAdmin = false
      this.waterToday = 0
      this.racha = 0
      this.dailyGoal = 2000
      this.currentUser = { name: '', id: '', id_producto: '', fechaRegistro: null, metaConsumoMl: 0 }
      this.activeTab = 'login'
    },
    addWater(amount) {
      this.waterToday = Math.min(this.waterToday + amount, 3500)
    },
    async leerBateria() {
      try {
        const response = await axios.get(`http://${process.env.VUE_APP_API_HOST}:3000/bateria`)
        console.log(response.data)
        this.nivel = response.data.voltaje
      } catch (error) {
        console.error('Error fetching user:', error)
      }
    },

    // ── Carga el perfil completo desde GET /me (usa el token, no necesita ID en URL) ──
    async fetchUserProfile() {
      try {
        const token = localStorage.getItem('token')
        if (!token) return

        const response = await axios.get(
          `http://${process.env.VUE_APP_API_HOST}:3000/me`,
          { headers: { 'x-auth-token': token } }
        )
        const u = response.data

        // Actualiza todos los campos del usuario con los datos reales de la DB
        this.currentUser = {
          name:          u.nombre        || this.currentUser.name,
          id:            u._id           || this.currentUser.id,
          id_producto:   u.id_producto   || '',
          fechaRegistro: u.fechaRegistro || null,
          metaConsumoMl: u.metaConsumoMl || 0
        }

        // Determinar si es admin según el rol poblado
        console.log('DEBUG rol recibido:', u.rol)
        const rolCargo = (u.rol && u.rol.cargo) ? u.rol.cargo.toLowerCase() : ''
        console.log('DEBUG rolCargo:', rolCargo, '→ isAdmin:', rolCargo === 'admin')
        this.isAdmin = rolCargo === 'admin'

        // Si el usuario tiene una meta guardada en la DB, úsala como dailyGoal global
        if (u.metaConsumoMl && u.metaConsumoMl > 0) {
          this.dailyGoal = u.metaConsumoMl
        }
      } catch (error) {
        console.error('Error al cargar perfil del usuario:', error)
      }
    },

    async login_fcn() {
      try {
        const userData = { correo: this.loginUser, clave: this.loginPass }
        const response = await axios.post(`http://${process.env.VUE_APP_API_HOST}:3000/login`, userData)

        this.mensajelogin = ''
        localStorage.setItem('token', response.data.token)
        localStorage.setItem('userId', response.data._id)

        // Datos básicos del login (el nombre real llega desde fetchUserProfile)
        this.currentUser = {
          name:          '',
          id:            response.data._id,
          id_producto:   '',
          fechaRegistro: null,
          metaConsumoMl: 0
        }

        // Carga perfil completo ANTES de mostrar la app para determinar el rol
        // y evitar el flash de HomeView cuando el usuario es administrador
        await this.fetchUserProfile()

        // Ahora sí activamos la sesión, ya sabemos el rol
        this.isLoggedIn = true

        if (this.isAdmin) {
          // Admin va directo a su panel, sin pasar por HomeView
        } else {
          this.currentScreen = 'inicio'
          await this.fetchWaterToday()
          await this.fetchRacha()
          this.iniciarPolling()
        }

      } catch (error) {
        if (error.response && error.response.status === 400) {
          this.mensajelogin = 'Credenciales incorrectas'
        }
        console.error('Error: ', error)
      }
    },

    async fetchRacha() {
      try {
        const token  = localStorage.getItem('token')
        const userId = localStorage.getItem('userId')
        if (!token || !userId) return

        const response = await axios.get(
          `http://${process.env.VUE_APP_API_HOST}:3000/usuarios/${userId}/racha`,
          { headers: { 'x-auth-token': token } }
        )
        console.log(response.data.racha)
        this.racha = response.data.racha ?? 0
      } catch (error) {
        console.error('Error al obtener racha:', error)
      }
    },

    async registro_fcn() {
      try {
        const userData = {
          nombre:      this.regName,
          correo:      this.regUser,
          id_producto: this.regId,
          clave:       this.regPass,
          telefono:    this.regTel,
          localidad:   this.regUbi
        }
        const response = await axios.post(`http://${process.env.VUE_APP_API_HOST}:3000/usuarios`, userData)
        const { token, _id } = response.data
        localStorage.setItem('token', token)
        localStorage.setItem('userId', _id)
        this.mensajeRegistro = 'Registro Exitoso'
        this.isLoggedIn = false
        this.activeTab = 'login'
      } catch (error) {
        if (error.response && error.response.data && error.response.data.msg) {
          this.mensajeRegistro = error.response.data.msg
        } else {
          this.mensajeRegistro = 'Error al conectar con el servidor'
        }
      }
    },

    async fetchWaterToday() {
      try {
        const token  = localStorage.getItem('token')
        const userId = localStorage.getItem('userId')
        if (!token || !userId) return

        const response = await axios.get(
          `http://${process.env.VUE_APP_API_HOST}:3000/usuarios/${userId}/consumos/hoy`,
          { headers: { 'x-auth-token': token } }
        )
        this.waterToday = response.data.total ?? 0
      } catch (error) {
        console.error('Error al obtener consumo de hoy:', error)
      }
    },

    async onAguaRegistrada() {
      await this.fetchWaterToday()
      await this.fetchRacha()
    },

    // Cuando PerfilView guarda una nueva meta, actualiza dailyGoal en toda la app
    onMetaActualizada(nuevaMeta) {
      this.dailyGoal = nuevaMeta
      this.currentUser = {
        ...this.currentUser,
        metaConsumoMl: nuevaMeta
      }
    },

    iniciarPolling() {
      if (this._pollingInterval) clearInterval(this._pollingInterval)
      this._pollingInterval = setInterval(async () => {
        await this.fetchWaterToday()
        await this.fetchRacha()
        await this.fetchUserProfile()   // refresca nombre, meta y datos del perfil
      }, 30000)
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800;900&family=Poppins:wght@400;500;600;700&display=swap');

* { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --primary: #4facfe;
  --primary-dark: #00c6ff;
  --accent: #0077cc;
  --white: #ffffff;
  --bg: #f0f8ff;
  --text: #1a2a3a;
  --text-muted: #6b8399;
  --card-shadow: 0 8px 32px rgba(79,172,254,0.13);
  --radius: 18px;
}

body {
  font-family: 'Nunito', sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
}

/* ===== LOGIN ===== */
.login-screen {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #e0f3ff 0%, #c8e8ff 50%, #d6f0ff 100%);
}
.login-card {
  background: white;
  border-radius: 28px;
  padding: 40px 36px;
  width: 100%;
  max-width: 420px;
  box-shadow: 0 20px 60px rgba(79,172,254,0.18);
}
.login-logo { text-align: center; margin-bottom: 28px; }
.login-logo-img { width: 100px; margin-bottom: 12px; }
.login-logo h1 { font-family: 'Poppins', sans-serif; font-size: 2rem; font-weight: 700; color: var(--accent); }
.login-logo p { color: var(--text-muted); font-size: 0.95rem; margin-top: 4px; }

.mensaje-error { text-align: center; color: #ff0000; font-size: 0.95rem; margin-top: 8px; }

.tab-switcher { display: flex; gap: 8px; margin-bottom: 24px; background: #f0f8ff; border-radius: 14px; padding: 5px; }
.tab-btn { flex: 1; padding: 10px; border: none; border-radius: 10px; background: transparent; color: var(--text-muted); font-family: 'Nunito', sans-serif; font-weight: 600; cursor: pointer; transition: all 0.2s; }
.tab-btn.active { background: white; color: var(--accent); box-shadow: 0 2px 10px rgba(0,0,0,0.08); }

.form-group input { width: 100%; padding: 14px 16px; margin-bottom: 12px; border: 2px solid #e0eeff; border-radius: 12px; font-family: 'Nunito', sans-serif; font-size: 0.97rem; transition: border-color 0.2s; display: block; }
.form-group input:focus { border-color: var(--primary); outline: none; }

/* ===== BUTTONS ===== */
.btn-primary { padding: 13px 24px; background: linear-gradient(135deg, var(--primary), var(--primary-dark)); color: white; border: none; border-radius: 12px; font-family: 'Nunito', sans-serif; font-weight: 700; font-size: 1rem; cursor: pointer; transition: all 0.2s; }
.btn-primary:hover { transform: translateY(-1px); box-shadow: 0 6px 20px rgba(79,172,254,0.35); }
.btn-primary.full-width { width: 100%; }
.btn-outline { padding: 7px 14px; border: 2px solid var(--primary); background: transparent; color: var(--primary); border-radius: 10px; font-family: 'Nunito', sans-serif; font-weight: 600; cursor: pointer; transition: all 0.2s; }
.btn-outline:hover { background: var(--primary); color: white; }
.btn-danger { padding: 7px 14px; border: none; background: #ff4757; color: white; border-radius: 10px; font-family: 'Nunito', sans-serif; font-weight: 600; cursor: pointer; }
.small { font-size: 0.82rem; }

/* ===== HEADER ===== */
.app-header { position: fixed; top: 0; left: 0; right: 0; z-index: 100; background: white; box-shadow: 0 2px 20px rgba(79,172,254,0.15); display: flex; align-items: center; justify-content: space-between; padding: 0 28px; height: 68px; gap: 16px; }
.header-logo-img { height: 67px; }
.header-nav { display: flex; gap: 4px; }
.nav-link { padding: 8px 16px; border: none; background: transparent; color: var(--text-muted); font-family: 'Nunito', sans-serif; font-weight: 700; font-size: 0.92rem; cursor: pointer; border-radius: 10px; transition: all 0.2s; }
.nav-link:hover { background: #f0f8ff; color: var(--accent); }
.nav-link.active { background: linear-gradient(135deg, var(--primary), var(--primary-dark)); color: white; }
.header-right { display: flex; align-items: center; gap: 16px; }
.header-user { display: flex; flex-direction: column; align-items: flex-end; gap: 4px; }
.user-name { font-weight: 700; font-size: 0.88rem; color: var(--text); }
.header-actions { display: flex; gap: 6px; }
.battery-section { display: flex; flex-direction: column; align-items: center; gap: 2px; }
.battery-label { font-size: 0.9rem; color: var(--text-muted); font-weight: 600; }
.battery-icon { height: 50px; }

/* ===== CONTENT ===== */
.app-content { padding-top: 84px; min-height: 100vh; }
</style>