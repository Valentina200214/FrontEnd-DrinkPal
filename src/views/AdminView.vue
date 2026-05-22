<template>
  <div class="admin-view">

    <!-- HEADER -->
    <header class="admin-header">
      <div class="header-left">
        <img :src="escudo" alt="Escudo DrinkPal" class="escudo-img" />

        <div>
          <h1>Rol Administrativo</h1>

          <p class="header-subtitle">
            Tu liderazgo impulsa el crecimiento y bienestar de toda la comunidad DrinkPal 💧
          </p>
        </div>
      </div>

      <div class="header-message">
        <p>
          "Cada mejora que realizas ayuda a miles de usuarios a construir hábitos saludables."
        </p>
      </div>

      <button class="logout-btn" @click="$emit('logout')">
        Cerrar Sesión
      </button>
    </header>


    <!-- BODY -->
    <main class="admin-body">

      <!-- PANEL IZQUIERDO -->
      <section class="left-section">

        <!-- GOTA -->
        <div class="gota-container">
          <img
            :src="gotaAdmin"
            alt="Gota Administrativa"
            class="gota-admin"
          />
        </div>

        <!-- PERFIL -->
        <div class="card">
          <h2>👤 Perfil Administrativo</h2>

          <div class="profile-info">
            <p><strong>Nombre:</strong> {{ adminUser.nombre }}</p>
            <p><strong>Correo:</strong> {{ adminUser.correo }}</p>
            <p><strong>Rol:</strong> {{ adminUser.rol }}</p>
          </div>
        </div>

        <!-- PRODUCTOS -->
        <div class="card">
          <h2>📦 Gestión de Productos</h2>

          <div class="input-group">
            <label>ID del producto</label>

            <input
              type="text"
              v-model="productId"
              placeholder="Ej: DP-1234"
            />

            <span v-if="mensajeProducto" :class="['feedback-msg', mensajeProducto.startsWith('✅') ? 'success' : 'error']">
              {{ mensajeProducto }}
            </span>
          </div>

          <div class="button-group">
            <button class="primary-btn" @click="crearProducto">
              Crear registro del producto
            </button>
          </div>
        </div>

        <!-- USUARIOS -->
        <div class="card">
          <h2>👥 Gestión de Usuarios</h2>

          <div class="input-group">
            <label>Correo del usuario</label>

            <input
              type="text"
              v-model="userCorreo"
              placeholder="Ingrese correo del usuario"
            />

            <span v-if="mensajeUsuario" :class="['feedback-msg', mensajeUsuario.startsWith('✅') ? 'success' : 'error']">
              {{ mensajeUsuario }}
            </span>
          </div>

          <button class="danger-btn full-btn" @click="eliminarUsuario">
            Eliminar usuario
          </button>
        </div>

      </section>


      <!-- PANEL DERECHO -->
      <section class="right-section">

        <!-- ── CONSULTAR INFORMACIÓN ─────────────────────────────── -->
        <div class="card filters-card">

          <h2>📊 Consultar Información</h2>

          <!-- FILTROS -->
          <div class="filters-grid">

            <div class="input-group">
              <label>Fecha inicio</label>
              <input type="date" v-model="fechaInicio" />
            </div>

            <div class="input-group">
              <label>Fecha fin</label>
              <input type="date" v-model="fechaFin" />
            </div>

            <div class="input-group">
              <label>Correo del usuario</label>

              <input
                type="text"
                v-model="filtroUsuario"
                placeholder="Filtrar por correo"
              />
            </div>

            <div class="input-group">
              <label>ID Producto</label>

              <input
                type="text"
                v-model="filtroProducto"
                placeholder="Ej: DP-1234"
              />
            </div>

          </div>

          <!-- BOTONES -->
          <div class="consulta-actions">

            <button class="primary-btn consulta-btn" @click="consultarInfo" :disabled="cargando">
              {{ cargando ? 'Consultando...' : 'Consultar información' }}
            </button>

            <button class="secondary-btn consulta-btn" @click="mostrarTodos" :disabled="cargando">
              Mostrar todos los usuarios
            </button>

          </div>

          <!-- MENSAJE CONSULTA -->
          <p v-if="mensajeConsulta" class="feedback-msg" :class="mensajeConsulta.startsWith('❌') ? 'error' : 'success'">
            {{ mensajeConsulta }}
          </p>

          <!-- TOOLBAR TABLA -->
          <div class="table-toolbar" v-if="registros.length > 0">
            <span class="tabla-modo-label">
              {{ modoTabla === 'usuarios' ? '👤 Vista: Usuarios únicos' : '📋 Vista: Registros de consumo' }}
            </span>
            <button class="download-btn" @click="descargarCSV" title="Descargar CSV">
              ⬇ CSV
            </button>
          </div>

          <!-- TABLA -->
          <div class="table-wrapper">

            <div class="table-scroll">

              <table>

                <thead>
                  <tr>
                    <th>Nombre</th>
                    <th>Correo</th>
                    <th>ID Producto</th>
                    <th>{{ modoTabla === 'usuarios' ? 'Consumo Total (ml)' : 'Consumo (ml)' }}</th>
                    <th>Racha</th>
                    <th>{{ modoTabla === 'usuarios' ? 'Fecha Registro' : 'Fecha / Hora' }}</th>
                  </tr>
                </thead>

                <tbody>

                  <tr v-if="registros.length === 0 && !cargando">
                    <td colspan="6" style="text-align:center; color:#90a4ae; padding: 28px;">
                      Sin registros. Usa los filtros y presiona "Consultar información".
                    </td>
                  </tr>

                  <tr
                    v-for="(reg, idx) in registrosPaginados"
                    :key="idx"
                  >
                    <td>{{ reg.nombre }}</td>
                    <td>{{ reg.correo }}</td>
                    <td>{{ reg.idProducto }}</td>
                    <td>{{ reg.consumo }}</td>
                    <td>{{ reg.racha }} días</td>
                    <td>{{ reg.fecha }}</td>
                  </tr>

                </tbody>

              </table>

            </div>

          </div>

          <!-- PAGINACIÓN -->
          <div class="pagination-wrapper" v-if="registros.length > 0">
            <span class="total-registros">Total: {{ registros.length }} registros</span>

            <div class="pagination-btns" v-if="totalPaginas > 1">
              <button
                class="pag-btn"
                @click="paginaActual = Math.max(1, paginaActual - 1)"
                :disabled="paginaActual === 1"
              >‹</button>

              <template v-for="(p, i) in paginasVisibles" :key="i">
                <span v-if="p === '...'" class="pag-ellipsis">…</span>
                <button
                  v-else
                  class="pag-btn"
                  :class="{ 'pag-active': p === paginaActual }"
                  @click="paginaActual = p"
                >{{ p }}</button>
              </template>

              <button
                class="pag-btn"
                @click="paginaActual = Math.min(totalPaginas, paginaActual + 1)"
                :disabled="paginaActual === totalPaginas"
              >›</button>
            </div>
          </div>

        </div>

        <!-- ── CONSULTAR ID PRODUCTOS ─────────────────────────────── -->
        <div class="card">
          <h2>🔍 Consultar ID Productos</h2>

          <div class="button-group">
            <button
              class="secondary-btn"
              @click="consultarProductos('disponibles')"
              :disabled="cargandoProductos"
            >
              {{ cargandoProductos && tipoProductos === 'disponibles' ? 'Cargando...' : '✅ Ver disponibles' }}
            </button>

            <button
              class="secondary-btn"
              @click="consultarProductos('en-uso')"
              :disabled="cargandoProductos"
            >
              {{ cargandoProductos && tipoProductos === 'en-uso' ? 'Cargando...' : '🔒 Ver en uso' }}
            </button>
          </div>

          <p v-if="mensajeProductosConsulta" class="feedback-msg" :class="mensajeProductosConsulta.startsWith('❌') ? 'error' : 'success'" style="margin-top:12px;">
            {{ mensajeProductosConsulta }}
          </p>

          <!-- TABLA PRODUCTOS -->
          <div class="table-wrapper" v-if="mostrarTablaProductos && productosData.length > 0">
            <p class="productos-subtitulo">
              {{ tipoProductos === 'disponibles' ? `🟢 Productos disponibles (${productosData.length})` : `🔴 Productos en uso (${productosData.length})` }}
            </p>

            <div class="table-scroll table-scroll-sm">
              <table>
                <thead>
                  <tr>
                    <th>ID Producto</th>
                    <th v-if="tipoProductos === 'en-uso'">Correo usuario</th>
                    <th v-if="tipoProductos === 'en-uso'">Nombre usuario</th>
                    <th v-if="tipoProductos === 'disponibles'">Estado</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(prod, idx) in productosData" :key="idx">
                    <td>{{ prod.id_producto }}</td>
                    <td v-if="tipoProductos === 'en-uso'">{{ prod.correo }}</td>
                    <td v-if="tipoProductos === 'en-uso'">{{ prod.nombre }}</td>
                    <td v-if="tipoProductos === 'disponibles'">
                      <span class="badge-disponible">Disponible</span>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

        </div>

        <!-- ── CREAR USUARIO ADMIN ────────────────────────────────── -->
        <div class="card">
          <h2>🛡️ Crear Usuario Administrador</h2>

          <div class="filters-grid">
            <div class="input-group">
              <label>Nombre</label>
              <input
                type="text"
                v-model="newAdmin.nombre"
                placeholder="Nombre completo"
              />
            </div>

            <div class="input-group">
              <label>Correo</label>
              <input
                type="email"
                v-model="newAdmin.correo"
                placeholder="correo@ejemplo.com"
              />
            </div>

            <div class="input-group">
              <label>Clave</label>
              <input
                type="password"
                v-model="newAdmin.clave"
                placeholder="Contraseña"
              />
            </div>

            <div class="input-group">
              <label>ID Producto</label>
              <input
                type="text"
                value="DP-admin"
                disabled
                style="background:#f0f4f8; color:#78909c; cursor:not-allowed;"
              />
            </div>
          </div>

          <span v-if="mensajeNuevoAdmin" :class="['feedback-msg', mensajeNuevoAdmin.startsWith('✅') ? 'success' : 'error']" style="display:block; margin-bottom:12px;">
            {{ mensajeNuevoAdmin }}
          </span>

          <button
            class="primary-btn"
            @click="crearAdmin"
            :disabled="cargandoAdmin"
            style="width:100%;"
          >
            {{ cargandoAdmin ? 'Creando...' : 'Crear administrador' }}
          </button>
        </div>

      </section>

    </main>


    <!-- FOOTER -->
    <footer class="admin-footer">

      <div>
        <h3>DrinkPal</h3>

        <p>
          Plataforma inteligente para monitoreo e hidratación saludable.
        </p>
      </div>

      <div>
        <p><strong>Correo:</strong> soporte@drinkpal.com</p>
        <p><strong>Teléfono:</strong> +57 300 000 0000</p>
      </div>

      <div>
        <p>© 2026 DrinkPal</p>
        <p>Todos los derechos reservados.</p>
      </div>

    </footer>

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'AdminView',

  data() {
    return {

      escudo: require('@/assets/escudo.png'),
      gotaAdmin: require('@/assets/Gota_admin.png'),

      // ── Perfil admin ──────────────────────────────────────────
      adminUser: {
        nombre: 'Cargando...',
        correo: 'Cargando...',
        rol: 'admin'
      },

      // ── Gestión de Productos ──────────────────────────────────
      productId: '',
      mensajeProducto: '',

      // ── Gestión de Usuarios ───────────────────────────────────
      userCorreo: '',
      mensajeUsuario: '',

      // ── Consultar Información ─────────────────────────────────
      fechaInicio: '',
      fechaFin: '',
      filtroUsuario: '',
      filtroProducto: '',
      cargando: false,
      mensajeConsulta: '',
      modoTabla: 'consumos', // 'consumos' | 'usuarios'

      // Filas de la tabla
      registros: [],

      // Paginación
      paginaActual: 1,
      registrosPorPagina: 20,

      // ── Consultar ID Productos ────────────────────────────────
      productosData: [],
      tipoProductos: '',
      mostrarTablaProductos: false,
      cargandoProductos: false,
      mensajeProductosConsulta: '',

      // ── Crear Admin ───────────────────────────────────────────
      newAdmin: { correo: '', nombre: '', clave: '' },
      mensajeNuevoAdmin: '',
      cargandoAdmin: false,
    }
  },

  computed: {
    registrosPaginados() {
      const inicio = (this.paginaActual - 1) * this.registrosPorPagina
      return this.registros.slice(inicio, inicio + this.registrosPorPagina)
    },

    totalPaginas() {
      return Math.max(1, Math.ceil(this.registros.length / this.registrosPorPagina))
    },

    paginasVisibles() {
      const t = this.totalPaginas
      const c = this.paginaActual
      if (t <= 9) return Array.from({ length: t }, (_, i) => i + 1)

      const set = new Set(
        [1, 2, c - 1, c, c + 1, t - 1, t].filter(p => p >= 1 && p <= t)
      )
      const sorted = [...set].sort((a, b) => a - b)
      const result = []
      for (let i = 0; i < sorted.length; i++) {
        if (i > 0 && sorted[i] - sorted[i - 1] > 1) result.push('...')
        result.push(sorted[i])
      }
      return result
    }
  },

  mounted() {
    this.fetchAdminProfile()
  },

  methods: {

    // ─────────────────────────────────────────────────────────────
    //  Perfil
    // ─────────────────────────────────────────────────────────────
    async fetchAdminProfile() {
      try {
        const token = localStorage.getItem('token')
        if (!token) return

        const res = await axios.get(`http://${import.meta.env.VITE_API_HOST}:3000/me`, {
          headers: { 'x-auth-token': token }
        })

        const u = res.data
        this.adminUser = {
          nombre: u.nombre  || 'Admin',
          correo: u.correo  || '-',
          rol: (u.rol && u.rol.cargo) ? u.rol.cargo : 'admin'
        }
      } catch (e) {
        console.error('Error al cargar perfil admin:', e)
      }
    },

    // ─────────────────────────────────────────────────────────────
    //  Gestión de Productos
    // ─────────────────────────────────────────────────────────────
    async crearProducto() {
      this.mensajeProducto = ''

      const regex = /^DP-\d{4}$/
      if (!this.productId.trim()) {
        this.mensajeProducto = '❌ Ingresa un ID de producto'
        return
      }
      if (!regex.test(this.productId.trim())) {
        this.mensajeProducto = '❌ El formato debe ser DP-XXXX (ej: DP-1234)'
        return
      }

      try {
        const token = localStorage.getItem('token')

        await axios.post(
          `http://${import.meta.env.VITE_API_HOST}:3000/productos`,
          { id_producto: this.productId.trim() },
          { headers: { 'x-auth-token': token } }
        )

        this.mensajeProducto = `✅ Producto ${this.productId.trim()} creado correctamente`
        this.productId = ''

      } catch (e) {
        if (e.response && e.response.data && e.response.data.msg) {
          this.mensajeProducto = `❌ ${e.response.data.msg}`
        } else {
          this.mensajeProducto = '❌ Error al crear el producto'
        }
      }
    },

    // ─────────────────────────────────────────────────────────────
    //  Gestión de Usuarios — error interno solo a consola
    // ─────────────────────────────────────────────────────────────
    async eliminarUsuario() {
      this.mensajeUsuario = ''

      if (!this.userCorreo.trim()) {
        this.mensajeUsuario = '❌ Ingresa el correo del usuario'
        return
      }

      const confirmado = confirm(
        `¿Seguro que deseas eliminar al usuario "${this.userCorreo.trim()}"?\nEsta acción no se puede deshacer.`
      )
      if (!confirmado) return

      try {
        const token = localStorage.getItem('token')

        const res = await axios.delete(
          `http://${import.meta.env.VITE_API_HOST}:3000/admin/usuarios/por-correo`,
          {
            headers: { 'x-auth-token': token },
            data: { correo: this.userCorreo.trim() }
          }
        )

        this.mensajeUsuario = `✅ ${res.data.msg}`
        this.userCorreo = ''

      } catch (e) {
        // Errores internos de Mongoose/Mongo solo van a consola
        console.error('Error al eliminar usuario:', e)

        if (e.response && e.response.data && e.response.data.msg) {
          const msg = e.response.data.msg

          // Si es un error interno de Mongoose, mostrar mensaje amigable
          const esErrorInterno =
            msg.includes('Cast to ObjectId') ||
            msg.includes('ObjectId') ||
            msg.includes('BSONType') ||
            msg.includes('BSONError') ||
            msg.includes('CastError')

          if (esErrorInterno) {
            this.mensajeUsuario = '❌ No se encontró ningún usuario con ese correo'
          } else {
            this.mensajeUsuario = `❌ ${msg}`
          }
        } else if (e.response && e.response.status === 404) {
          this.mensajeUsuario = '❌ No se encontró ningún usuario con ese correo'
        } else {
          this.mensajeUsuario = '❌ Error al eliminar el usuario'
        }
      }
    },

    // ─────────────────────────────────────────────────────────────
    //  Consultar Información (por filtros)
    // ─────────────────────────────────────────────────────────────
    async consultarInfo() {
      this.cargando = true
      this.registros = []
      this.mensajeConsulta = ''
      this.paginaActual = 1
      this.modoTabla = 'consumos'

      try {
        const token = localStorage.getItem('token')
        const params = {}

        if (this.fechaInicio)           params.fechaInicio  = this.fechaInicio
        if (this.fechaFin)              params.fechaFin     = this.fechaFin
        if (this.filtroUsuario.trim())  params.correo       = this.filtroUsuario.trim()
        if (this.filtroProducto.trim()) params.id_producto  = this.filtroProducto.trim()

        const res = await axios.get(`http://${import.meta.env.VITE_API_HOST}:3000/admin/consumos`, {
          headers: { 'x-auth-token': token },
          params
        })

        const consumos = res.data.msg || []

        this.registros = consumos.map(c => ({
          nombre:     c.usuarioFinal ? c.usuarioFinal.nombre      : '-',
          correo:     c.usuarioFinal ? c.usuarioFinal.correo      : '-',
          idProducto: c.usuarioFinal ? c.usuarioFinal.id_producto : '-',
          consumo:    `${c.consumoInstante} ml`,
          racha:      c.racha ?? 0,
          fecha:      new Date(c.fechaHora).toLocaleString('es-CO')
        }))

        if (this.registros.length === 0) {
          this.mensajeConsulta = '⚠️ No se encontraron registros con los filtros aplicados.'
        }

      } catch (e) {
        console.error('Error al consultar consumos:', e)
        this.mensajeConsulta = '❌ Error al obtener los datos. Intenta de nuevo.'
      } finally {
        this.cargando = false
      }
    },

    // ─────────────────────────────────────────────────────────────
    //  Mostrar todos los usuarios (sin repetidos)
    // ─────────────────────────────────────────────────────────────
    async mostrarTodos() {
      this.cargando = true
      this.registros = []
      this.mensajeConsulta = ''
      this.paginaActual = 1
      this.modoTabla = 'usuarios'
      this.fechaInicio    = ''
      this.fechaFin       = ''
      this.filtroUsuario  = ''
      this.filtroProducto = ''

      try {
        const token = localStorage.getItem('token')

        const res = await axios.get(`http://${import.meta.env.VITE_API_HOST}:3000/admin/usuarios-resumen`, {
          headers: { 'x-auth-token': token }
        })

        const usuarios = res.data.msg || []

        this.registros = usuarios.map(u => ({
          nombre:     u.nombre     || '-',
          correo:     u.correo     || '-',
          idProducto: u.idProducto || '-',
          consumo:    `${u.consumoTotal ?? 0} ml`,
          racha:      u.racha      ?? 0,
          fecha:      u.fechaRegistro
            ? new Date(u.fechaRegistro).toLocaleDateString('es-CO')
            : '-'
        }))

        if (this.registros.length === 0) {
          this.mensajeConsulta = '⚠️ No hay usuarios registrados.'
        }

      } catch (e) {
        console.error('Error al obtener usuarios:', e)
        this.mensajeConsulta = '❌ Error al obtener los datos. Intenta de nuevo.'
      } finally {
        this.cargando = false
      }
    },

    // ─────────────────────────────────────────────────────────────
    //  Consultar ID Productos
    // ─────────────────────────────────────────────────────────────
    async consultarProductos(tipo) {
      // Si ya está mostrando el mismo tipo, alternar (colapsar)
      if (this.mostrarTablaProductos && this.tipoProductos === tipo) {
        this.mostrarTablaProductos = false
        return
      }

      this.cargandoProductos = true
      this.mensajeProductosConsulta = ''
      this.tipoProductos = tipo
      this.productosData = []
      this.mostrarTablaProductos = false

      try {
        const token = localStorage.getItem('token')

        const endpoint = tipo === 'disponibles'
          ? `http://${import.meta.env.VITE_API_HOST}:3000/admin/productos/disponibles`
          : `http://${import.meta.env.VITE_API_HOST}:3000/admin/productos/en-uso`

        const res = await axios.get(endpoint, {
          headers: { 'x-auth-token': token }
        })

        this.productosData = res.data.msg || []
        this.mostrarTablaProductos = true

        if (this.productosData.length === 0) {
          this.mensajeProductosConsulta = `⚠️ No hay productos ${tipo === 'disponibles' ? 'disponibles' : 'en uso'} actualmente.`
        }

      } catch (e) {
        console.error('Error al consultar productos:', e)
        this.mensajeProductosConsulta = '❌ Error al obtener los productos.'
      } finally {
        this.cargandoProductos = false
      }
    },

    // ─────────────────────────────────────────────────────────────
    //  Descarga CSV
    // ─────────────────────────────────────────────────────────────
    descargarCSV() {
      if (this.registros.length === 0) return

      const encabezados = [
        'Nombre',
        'Correo',
        'ID Producto',
        this.modoTabla === 'usuarios' ? 'Consumo Total (ml)' : 'Consumo (ml)',
        'Racha (días)',
        this.modoTabla === 'usuarios' ? 'Fecha Registro' : 'Fecha / Hora'
      ]

      const filas = this.registros.map(r => [
        `"${r.nombre}"`,
        `"${r.correo}"`,
        `"${r.idProducto}"`,
        `"${r.consumo}"`,
        `"${r.racha}"`,
        `"${r.fecha}"`
      ])

      const csvContent = [encabezados.join(','), ...filas.map(f => f.join(','))].join('\n')
      const BOM = '\uFEFF' // para que Excel abra bien con tildes
      const blob = new Blob([BOM + csvContent], { type: 'text/csv;charset=utf-8;' })
      const url = URL.createObjectURL(blob)
      const link = document.createElement('a')
      const fecha = new Date().toISOString().slice(0, 10)
      link.href = url
      link.setAttribute('download', `drinkpal_${this.modoTabla}_${fecha}.csv`)
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      URL.revokeObjectURL(url)
    },

    // ─────────────────────────────────────────────────────────────
    //  Crear Usuario Admin
    // ─────────────────────────────────────────────────────────────
    async crearAdmin() {
      this.mensajeNuevoAdmin = ''

      if (!this.newAdmin.correo.trim()) {
        this.mensajeNuevoAdmin = '❌ El correo es obligatorio'
        return
      }
      if (!this.newAdmin.nombre.trim()) {
        this.mensajeNuevoAdmin = '❌ El nombre es obligatorio'
        return
      }
      if (!this.newAdmin.clave.trim()) {
        this.mensajeNuevoAdmin = '❌ La clave es obligatoria'
        return
      }

      this.cargandoAdmin = true

      try {
        const token = localStorage.getItem('token')

        const res = await axios.post(
          `http://${import.meta.env.VITE_API_HOST}:3000/admin/crear-admin`,
          {
            correo: this.newAdmin.correo.trim(),
            nombre: this.newAdmin.nombre.trim(),
            clave:  this.newAdmin.clave.trim()
          },
          { headers: { 'x-auth-token': token } }
        )

        this.mensajeNuevoAdmin = `✅ ${res.data.msg}`
        this.newAdmin = { correo: '', nombre: '', clave: '' }

      } catch (e) {
        console.error('Error al crear admin:', e)
        if (e.response && e.response.data && e.response.data.msg) {
          this.mensajeNuevoAdmin = `❌ ${e.response.data.msg}`
        } else {
          this.mensajeNuevoAdmin = '❌ Error al crear el administrador'
        }
      } finally {
        this.cargandoAdmin = false
      }
    }

  }
}
</script>

<style scoped>

.admin-view {
  min-height: 100vh;
  background: #f0f8ff;
  display: flex;
  flex-direction: column;
  color: #1e3a5f;
  overflow-x: hidden;
}

/* HEADER */

.admin-header {
  background: linear-gradient(135deg, #4facfe, #00c6fb);
  padding: 24px 40px;

  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;

  gap: 20px;

  color: white;

  box-shadow: 0 4px 20px rgba(0,0,0,0.12);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 18px;
}

.escudo-img {
  width: 85px;
  height: 85px;
  object-fit: contain;

  background: rgba(255,255,255,0.15);

  border-radius: 18px;
  padding: 8px;
}

.admin-header h1 {
  font-size: 2rem;
  margin-bottom: 6px;
}

.header-subtitle {
  font-size: 0.95rem;
  opacity: 0.95;
}

.header-message {
  max-width: 400px;

  background: rgba(255,255,255,0.15);

  padding: 16px;

  border-radius: 18px;

  backdrop-filter: blur(10px);
}

.logout-btn {
  padding: 10px 20px;

  background: rgba(255,255,255,0.2);

  color: white;

  border: 2px solid rgba(255,255,255,0.5);

  border-radius: 12px;

  font-weight: 700;

  cursor: pointer;

  transition: 0.3s;
}

.logout-btn:hover {
  background: rgba(255,255,255,0.35);
}

/* BODY */

.admin-body {
  flex: 1;

  display: grid;

  grid-template-columns: 360px 1fr;

  gap: 24px;

  padding: 24px;

  width: 100%;
  max-width: 100%;

  box-sizing: border-box;
}

/* LEFT */

.left-section {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.gota-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.gota-admin {
  width: 220px;

  object-fit: contain;

  filter: drop-shadow(0 10px 30px rgba(79,172,254,0.35));

  animation: floating 4s ease-in-out infinite;
}

@keyframes floating {

  0% {
    transform: translateY(0px);
  }

  50% {
    transform: translateY(-15px);
  }

  100% {
    transform: translateY(0px);
  }
}

/* RIGHT */

.right-section {
  width: 100%;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

/* CARDS */

.card {
  background: white;

  border-radius: 24px;

  padding: 24px;

  box-shadow: 0 8px 25px rgba(79,172,254,0.12);

  border: 1px solid rgba(79,172,254,0.15);
}

.card h2 {
  color: #1976d2;
  margin-bottom: 20px;
  font-size: 1.3rem;
}

/* PERFIL */

.profile-info {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.profile-info p {
  background: #f5fbff;

  padding: 12px;

  border-radius: 12px;
}

/* INPUTS */

.input-group {
  display: flex;
  flex-direction: column;

  gap: 8px;

  margin-bottom: 18px;
}

.input-group label {
  font-weight: 600;
  color: #1565c0;
}

.input-group input {
  padding: 12px 14px;

  border-radius: 12px;

  border: 1px solid #cfd8dc;

  outline: none;

  transition: 0.3s;
}

.input-group input:focus {
  border-color: #4facfe;

  box-shadow: 0 0 0 4px rgba(79,172,254,0.15);
}

/* FEEDBACK */

.feedback-msg {
  font-size: 0.88rem;
  font-weight: 600;
  padding: 8px 12px;
  border-radius: 10px;
  margin-top: 4px;
}

.feedback-msg.success {
  background: #e8f5e9;
  color: #2e7d32;
}

.feedback-msg.error {
  background: #ffebee;
  color: #c62828;
}

/* BOTONES */

.button-group,
.consulta-actions {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.primary-btn,
.secondary-btn,
.danger-btn {
  border: none;

  padding: 12px 18px;

  border-radius: 14px;

  cursor: pointer;

  font-weight: 600;

  transition: 0.3s;
}

.primary-btn {
  background: linear-gradient(135deg, #4facfe, #00c6fb);
  color: white;
}

.primary-btn:hover {
  transform: translateY(-2px);

  box-shadow: 0 8px 20px rgba(79,172,254,0.35);
}

.primary-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.secondary-btn {
  background: #e3f2fd;
  color: #1565c0;
}

.secondary-btn:hover {
  background: #d4ecff;
}

.secondary-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.danger-btn {
  background: #ffebee;
  color: #d32f2f;
}

.danger-btn:hover {
  background: #ffcdd2;
}

.full-btn {
  width: 100%;
}

/* FILTROS */

.filters-grid {
  display: grid;

  grid-template-columns: repeat(2, 1fr);

  gap: 16px;
}

/* TOOLBAR TABLA */

.table-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
  margin-bottom: 4px;
}

.tabla-modo-label {
  font-size: 0.82rem;
  font-weight: 600;
  color: #78909c;
}

.download-btn {
  background: #e8f5e9;
  color: #2e7d32;
  border: none;
  border-radius: 10px;
  padding: 7px 13px;
  font-size: 0.82rem;
  font-weight: 700;
  cursor: pointer;
  transition: 0.2s;
}

.download-btn:hover {
  background: #c8e6c9;
  transform: translateY(-1px);
}

/* TABLA */

.table-wrapper {
  margin-top: 12px;

  width: 100%;
}

.table-scroll {
  max-height: 420px;

  overflow-y: auto;
  overflow-x: auto;

  border-radius: 16px;

  border: 1px solid #dbefff;
}

.table-scroll-sm {
  max-height: 280px;
}

/* SCROLL */

.table-scroll::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}

.table-scroll::-webkit-scrollbar-thumb {
  background: #90caf9;
  border-radius: 10px;
}

.table-scroll::-webkit-scrollbar-track {
  background: #e3f2fd;
}

/* TABLA */

table {
  width: 100%;
  border-collapse: collapse;
  min-width: 500px;
}

thead {
  position: sticky;
  top: 0;
  z-index: 2;
}

th {
  background: linear-gradient(135deg, #4facfe, #00c6fb);

  color: white;

  padding: 14px;

  text-align: left;

  white-space: nowrap;
}

td {
  padding: 14px;

  border-bottom: 1px solid #e3f2fd;

  background: white;
}

tr:hover td {
  background: #f7fcff;
}

/* PAGINACIÓN */

.pagination-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 16px;
  padding-top: 12px;
  border-top: 1px solid #e3f2fd;
}

.total-registros {
  font-size: 0.85rem;
  color: #78909c;
  font-weight: 600;
}

.pagination-btns {
  display: flex;
  gap: 6px;
  align-items: center;
  flex-wrap: wrap;
}

.pag-btn {
  min-width: 36px;
  height: 36px;
  border: 1px solid #cfd8dc;
  background: white;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 600;
  font-size: 0.85rem;
  color: #1565c0;
  transition: 0.2s;
  padding: 0 8px;
}

.pag-btn:hover:not(:disabled) {
  background: #e3f2fd;
  border-color: #4facfe;
}

.pag-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.pag-active {
  background: linear-gradient(135deg, #4facfe, #00c6fb) !important;
  color: white !important;
  border-color: #4facfe !important;
}

.pag-ellipsis {
  color: #90a4ae;
  font-weight: 600;
  padding: 0 4px;
}

/* CONSULTAR PRODUCTOS */

.productos-subtitulo {
  font-size: 0.88rem;
  font-weight: 700;
  color: #1565c0;
  margin-bottom: 10px;
  margin-top: 16px;
}

.badge-disponible {
  background: #e8f5e9;
  color: #2e7d32;
  border-radius: 8px;
  padding: 4px 10px;
  font-size: 0.78rem;
  font-weight: 700;
}

/* FOOTER */

.admin-footer {
  background: #1565c0;

  color: white;

  padding: 24px 40px;

  display: flex;
  justify-content: space-between;

  flex-wrap: wrap;

  gap: 24px;
}

.admin-footer h3 {
  margin-bottom: 8px;
}

/* RESPONSIVE */

@media (max-width: 1100px) {

  .admin-body {
    grid-template-columns: 1fr;
  }

  .left-section {
    order: 1;
  }

  .right-section {
    order: 2;
  }

  .gota-admin {
    width: 180px;
  }
}

@media (max-width: 768px) {

  .admin-header {
    padding: 20px;
  }

  .admin-body {
    padding: 18px;
  }

  .filters-grid {
    grid-template-columns: 1fr;
  }

  .button-group,
  .consulta-actions {
    flex-direction: column;
  }

  .primary-btn,
  .secondary-btn,
  .danger-btn {
    width: 100%;
  }

  table {
    min-width: 500px;
  }

  .pagination-wrapper {
    flex-direction: column;
    align-items: flex-start;
  }
}

</style>
