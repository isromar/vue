<script setup>
import { computed, onMounted, ref, watch } from 'vue'

const emit = defineEmits(['eventoEmitido'])

const prioridadOrden = { Alta: 1, Media: 2, Baja: 3 }
const listaTareas = ref([])
const nombreTarea = ref('')
const prioridadTarea = ref('Baja')
const fechaTarea = ref(obtenerFechaHoy())
const filtroActual = ref('todas')
const terminoBusqueda = ref('')
const formularioAbierto = ref(false)
const errorFormulario = ref('')
let numId = 0

function obtenerFechaHoy() {
  const hoy = new Date()
  const año = hoy.getFullYear()
  const mes = String(hoy.getMonth() + 1).padStart(2, '0')
  const dia = String(hoy.getDate()).padStart(2, '0')
  return `${año}-${mes}-${dia}`
}

function leerLocalStorage(key, fallback) {
  const valor = localStorage.getItem(key)
  if (!valor) return fallback

  try {
    return JSON.parse(valor)
  } catch (error) {
    console.error(`No se pudo leer la lista guardada (${key}).`, error)
    return fallback
  }
}

const tareasCompletadas = computed(() => listaTareas.value.filter((tarea) => tarea.done))
const tareasActivas = computed(() => listaTareas.value.filter((tarea) => !tarea.done))
const porcentajeCompletado = computed(() => {
  if (!listaTareas.value.length) return 0
  return Math.round((tareasCompletadas.value.length / listaTareas.value.length) * 100)
})

const listaTareasFiltradas = computed(() => {
  const busqueda = terminoBusqueda.value.trim().toLocaleLowerCase('es')
  let tareas = [...listaTareas.value].sort((a, b) => {
    const prioridadA = prioridadOrden[a.prioridad] ?? 4
    const prioridadB = prioridadOrden[b.prioridad] ?? 4
    return prioridadA - prioridadB || (a.creada ?? 0) - (b.creada ?? 0)
  })

  if (filtroActual.value === 'completadas') {
    tareas = tareas.filter((tarea) => tarea.done)
  } else if (filtroActual.value === 'activas') {
    tareas = tareas.filter((tarea) => !tarea.done)
  }

  if (busqueda) {
    tareas = tareas.filter((tarea) =>
      tarea.content.toLocaleLowerCase('es').includes(busqueda)
    )
  }

  return tareas
})

function guardarLista() {
  localStorage.setItem('listadoLocal', JSON.stringify(listaTareas.value))
  localStorage.setItem('listadoLocalOld', JSON.stringify(listaTareas.value))
  localStorage.setItem('numId', String(numId))
}

function abrirFormulario() {
  formularioAbierto.value = true
  errorFormulario.value = ''
  requestAnimationFrame(() => {
    document.getElementById('nombre-nueva-tarea')?.focus()
  })
}

function cerrarFormulario() {
  formularioAbierto.value = false
  errorFormulario.value = ''
}

function anyadirTarea() {
  if (!nombreTarea.value.trim()) {
    errorFormulario.value = 'Escribe un nombre para la tarea.'
    return
  }

  if (!fechaTarea.value) {
    errorFormulario.value = 'Selecciona una fecha para la tarea.'
    return
  }

  numId += 1
  listaTareas.value.push({
    id: numId,
    content: nombreTarea.value.trim(),
    prioridad: prioridadTarea.value,
    prioridad_id: prioridadOrden[prioridadTarea.value],
    done: false,
    fecha: fechaTarea.value,
    creada: Date.now()
  })

  nombreTarea.value = ''
  prioridadTarea.value = 'Baja'
  fechaTarea.value = obtenerFechaHoy()
  cerrarFormulario()
}

function cambiarEstado(tarea) {
  tarea.done = !tarea.done
}

function cambiarPrioridad(tarea, prioridad) {
  tarea.prioridad = prioridad
  tarea.prioridad_id = prioridadOrden[prioridad]
}

function borrarTarea(id) {
  listaTareas.value = listaTareas.value.filter((tarea) => tarea.id !== id)
}

function borrarTareasCompletadas() {
  listaTareas.value = listaTareas.value.filter((tarea) => !tarea.done)
}

function borrarTareasTodas() {
  if (!listaTareas.value.length) return
  if (!window.confirm('¿Quieres borrar todas las tareas? Esta acción no se puede deshacer.')) return

  listaTareas.value = []
  filtroActual.value = 'todas'
  terminoBusqueda.value = ''
  emit('eventoEmitido')
}

function seleccionarFiltro(filtro) {
  filtroActual.value = filtro
}

function formatoFecha(fecha) {
  if (!fecha) return 'Sin fecha'
  const [año, mes, dia] = fecha.split('-').map(Number)
  const fechaLocal = new Date(año, mes - 1, dia)
  return new Intl.DateTimeFormat('es-ES', {
    day: 'numeric',
    month: 'short',
    year: 'numeric'
  }).format(fechaLocal)
}

const fechaActual = computed(() => new Intl.DateTimeFormat('es-ES', {
  weekday: 'long',
  day: 'numeric',
  month: 'long'
}).format(new Date()))

watch(listaTareas, guardarLista, { deep: true })

onMounted(() => {
  const tareasGuardadas = leerLocalStorage('listadoLocal', null)
  const tareasAntiguas = leerLocalStorage('listadoLocalOld', [])
  listaTareas.value = Array.isArray(tareasGuardadas)
    ? tareasGuardadas
    : Array.isArray(tareasAntiguas)
      ? tareasAntiguas
      : []
  numId = Math.max(
    Number(localStorage.getItem('numId')) || 0,
    ...listaTareas.value.map((tarea) => Number(tarea.id) || 0)
  )
})
</script>

<template>
  <main class="sereno-app">
    <aside class="sidebar" aria-label="Navegación principal">
      <a class="brand" href="#inicio" aria-label="Hecho, ir al inicio">
        <span class="brand-mark" aria-hidden="true">
          <svg viewBox="0 0 24 24">
            <path d="m5 12 4.2 4.2L19 6.8" />
          </svg>
        </span>
        <span class="brand-copy">
          <strong>hecho</strong>
          <small>TU ESPACIO DE TAREAS</small>
        </span>
      </a>

      <p class="side-label">MENÚ</p>
      <nav class="side-nav">
        <a class="nav-link" href="#resumen">
          <svg viewBox="0 0 24 24" aria-hidden="true"><rect x="4" y="4" width="6" height="6" rx="1" /><rect x="14" y="4" width="6" height="6" rx="1" /><rect x="4" y="14" width="6" height="6" rx="1" /><rect x="14" y="14" width="6" height="6" rx="1" /></svg>
          Resumen
        </a>
        <button class="nav-link" :class="{ active: filtroActual === 'todas' }" type="button" @click="seleccionarFiltro('todas')">
          <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M8 5h11M8 12h11M8 19h11" /><path d="m3.5 5 .8.8L5.8 4M3.5 12l.8.8 1.5-1.8M3.5 19l.8.8 1.5-1.8" /></svg>
          Mis tareas
        </button>
        <button class="nav-link" :class="{ active: filtroActual === 'activas' }" type="button" @click="seleccionarFiltro('activas')">
          <svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="12" cy="12" r="8.5" /><path d="M12 7v5l3 2" /></svg>
          Pendientes
        </button>
        <button class="nav-link" :class="{ active: filtroActual === 'completadas' }" type="button" @click="seleccionarFiltro('completadas')">
          <svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="12" cy="12" r="8.5" /><path d="m8 12 2.5 2.5L16 9" /></svg>
          Completadas
        </button>
      </nav>

      <div class="sidebar-progress">
        <p class="side-label">TU PROGRESO</p>
        <strong>{{ tareasCompletadas.length }} de {{ listaTareas.length }} tareas</strong>
        <div
          class="progress-track"
          role="progressbar"
          :aria-valuenow="porcentajeCompletado"
          aria-valuemin="0"
          aria-valuemax="100"
          :aria-label="`${porcentajeCompletado}% de tareas completadas`"
        >
          <span :style="{ width: `${porcentajeCompletado}%` }"></span>
        </div>
        <small>{{ porcentajeCompletado === 100 && listaTareas.length ? '¡Todo listo, buen trabajo!' : 'Vas por buen camino' }}</small>
      </div>

      <div class="sidebar-profile">
        <span class="avatar" aria-hidden="true">IR</span>
        <span><strong>Isabel Rodenas</strong><small>Plan personal</small></span>
      </div>
    </aside>

    <section id="inicio" class="main-content">
      <header class="topbar">
        <span class="today-label">{{ fechaActual }}</span>
        <span class="avatar top-avatar" aria-label="Isabel Rodenas">IR</span>
      </header>

      <section id="resumen" class="welcome-section">
        <div>
          <p class="eyebrow">TU ESPACIO, A TU RITMO</p>
          <h1>Hola, Isabel <span aria-hidden="true">👋</span></h1>
          <p class="welcome-copy">Un paso cada vez. Esto es lo que tienes para hoy.</p>
        </div>
        <span class="date-chip">{{ fechaActual }}</span>
      </section>

      <section class="summary-cards" aria-label="Resumen de tareas">
        <article class="summary-card">
          <span class="summary-icon total-icon" aria-hidden="true">▤</span>
          <div><strong>{{ listaTareas.length }}</strong><span>Tareas totales</span><small>En tu lista</small></div>
        </article>
        <article class="summary-card">
          <span class="summary-icon active-icon" aria-hidden="true">○</span>
          <div><strong>{{ tareasActivas.length }}</strong><span>Por completar</span><small>Vas muy bien</small></div>
        </article>
        <article class="summary-card">
          <span class="summary-icon done-icon" aria-hidden="true">✓</span>
          <div><strong>{{ tareasCompletadas.length }}</strong><span>Completadas</span><small>¡Buen trabajo!</small></div>
        </article>
      </section>

      <section id="mis-tareas" class="tasks-section">
        <div class="tasks-heading">
          <div>
            <h2>Tus tareas</h2>
            <p>Organizadas para que te centres en lo importante.</p>
          </div>
          <label class="search-field">
            <svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="10.8" cy="10.8" r="6.8" /><path d="m16 16 4 4" /></svg>
            <input v-model="terminoBusqueda" type="search" placeholder="Buscar una tarea..." aria-label="Buscar una tarea" />
          </label>
        </div>

        <div class="task-toolbar">
          <div class="filter-tabs" role="group" aria-label="Filtrar tareas">
            <button type="button" :class="{ selected: filtroActual === 'todas' }" @click="seleccionarFiltro('todas')">
              Todas <span>{{ listaTareas.length }}</span>
            </button>
            <button type="button" :class="{ selected: filtroActual === 'activas' }" @click="seleccionarFiltro('activas')">
              Activas <span>{{ tareasActivas.length }}</span>
            </button>
            <button type="button" :class="{ selected: filtroActual === 'completadas' }" @click="seleccionarFiltro('completadas')">
              Completadas <span>{{ tareasCompletadas.length }}</span>
            </button>
          </div>
          <button class="add-task-button" type="button" @click="abrirFormulario">
            <span aria-hidden="true">+</span> Nueva tarea
          </button>
        </div>

        <form v-if="formularioAbierto" class="new-task-form" @submit.prevent="anyadirTarea">
          <div class="form-heading">
            <div><h3>Añadir una tarea</h3><p>Un objetivo claro es un buen comienzo.</p></div>
            <button class="close-form" type="button" aria-label="Cerrar formulario" @click="cerrarFormulario">×</button>
          </div>
          <label class="form-control-group task-name-field">
            <span>Nombre de la tarea</span>
            <input
              id="nombre-nueva-tarea"
              v-model="nombreTarea"
              type="text"
              maxlength="120"
              placeholder="¿Qué tienes que hacer?"
              @input="errorFormulario = ''"
            />
          </label>
          <div class="form-row">
            <label class="form-control-group">
              <span>Fecha</span>
              <input v-model="fechaTarea" type="date" />
            </label>
            <fieldset class="priority-choice">
              <legend>Prioridad</legend>
              <label v-for="prioridad in ['Alta', 'Media', 'Baja']" :key="prioridad">
                <input v-model="prioridadTarea" type="radio" name="prioridad" :value="prioridad" />
                <span :class="`priority-${prioridad.toLowerCase()}`">{{ prioridad }}</span>
              </label>
            </fieldset>
          </div>
          <p v-if="errorFormulario" class="form-error" role="alert">{{ errorFormulario }}</p>
          <div class="form-actions">
            <button class="cancel-button" type="button" @click="cerrarFormulario">Cancelar</button>
            <button class="add-task-button" type="submit">Guardar tarea</button>
          </div>
        </form>

        <div class="task-list">
          <div v-if="listaTareasFiltradas.length" class="task-list-header">
            <span>Nombre de la tarea</span><span>Fecha</span><span>Prioridad</span><span class="sr-only">Acciones</span>
          </div>
          <article v-for="tarea in listaTareasFiltradas" :key="tarea.id" class="task-row">
            <button
              class="task-checkbox"
              :class="{ checked: tarea.done }"
              type="button"
              :aria-label="tarea.done ? `Marcar ${tarea.content} como activa` : `Marcar ${tarea.content} como completada`"
              :aria-pressed="tarea.done"
              @click="cambiarEstado(tarea)"
            >
              <svg v-if="tarea.done" viewBox="0 0 20 20" aria-hidden="true"><path d="m4 10 4 4 8-8" /></svg>
            </button>
            <div class="task-title" :class="{ completed: tarea.done }">
              <strong>{{ tarea.content }}</strong>
            </div>
            <span class="task-date">{{ formatoFecha(tarea.fecha) }}</span>
            <div class="task-priority-controls" :aria-label="`Prioridad: ${tarea.prioridad}`">
              <span class="priority-pill" :class="`priority-${tarea.prioridad.toLowerCase()}`">{{ tarea.prioridad }}</span>
              <select
                :value="tarea.prioridad"
                :aria-label="`Cambiar prioridad de ${tarea.content}`"
                @change="cambiarPrioridad(tarea, $event.target.value)"
              >
                <option value="Alta">Alta</option>
                <option value="Media">Media</option>
                <option value="Baja">Baja</option>
              </select>
            </div>
            <button class="delete-task-button" type="button" :aria-label="`Borrar ${tarea.content}`" @click="borrarTarea(tarea.id)">
              <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M4 7h16M10 11v6m4-6v6M6 7l1 14h10l1-14M9 7V4h6v3" /></svg>
            </button>
          </article>

          <div v-if="!listaTareasFiltradas.length" class="empty-state">
            <span class="empty-icon" aria-hidden="true">✓</span>
            <h3>{{ terminoBusqueda ? 'No encontramos esa tarea' : 'Tu lista está despejada' }}</h3>
            <p>{{ terminoBusqueda ? 'Prueba con otras palabras.' : 'Añade una tarea y empieza a organizar tu día.' }}</p>
            <button v-if="!terminoBusqueda && filtroActual === 'todas'" class="text-action" type="button" @click="abrirFormulario">
              Añadir mi primera tarea
            </button>
          </div>
        </div>

        <div v-if="listaTareas.length" class="completed-actions">
          <span>{{ tareasCompletadas.length }} {{ tareasCompletadas.length === 1 ? 'tarea completada' : 'tareas completadas' }}</span>
          <div class="list-actions">
            <button v-if="tareasCompletadas.length" type="button" @click="borrarTareasCompletadas">Borrar completadas</button>
            <button class="clear-all-button" type="button" @click="borrarTareasTodas">Borrar todas</button>
          </div>
        </div>
      </section>

      <footer class="inspiration-card">
        <span class="inspiration-icon" aria-hidden="true">✦</span>
        <div>
          <strong>Un buen día empieza con una buena prioridad.</strong>
          <p>Tu lista se guarda automáticamente en este navegador.</p>
        </div>
        <span class="inspiration-sparkle" aria-hidden="true">✧</span>
      </footer>
    </section>
  </main>
</template>

<style>
:root {
  font-family: Inter, "Segoe UI", sans-serif;
  color: #1e3029;
  background: #f3f5f2;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-optical-sizing: auto;
  font-weight: 400;
}

* { box-sizing: border-box; }

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
  background: #f3f5f2;
}

button, input, select { font: inherit; }
button { cursor: pointer; }

.sereno-app {
  --sereno-ink: #1e3029;
  --sereno-muted: #829088;
  --sereno-accent: #2b806a;
  --sereno-border: #e5eae5;
  --sereno-soft: #f1f6f2;
  display: grid;
  grid-template-columns: 250px minmax(0, 1fr);
  gap: 40px;
  width: min(100% - 48px, 1600px);
  min-height: calc(100vh - 44px);
  margin: 22px auto;
  padding: 18px;
  border: 1px solid var(--sereno-border);
  border-radius: 25px;
  background: #fbfcfa;
}

.sidebar {
  position: sticky;
  top: 18px;
  display: flex;
  flex-direction: column;
  align-self: start;
  min-height: calc(100vh - 80px);
  padding: 24px 16px 18px;
  border-radius: 20px;
  background: #fff;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 0 7px 45px;
  color: var(--sereno-ink);
  text-decoration: none;
}

.brand-mark {
  display: grid;
  width: 42px;
  height: 42px;
  place-items: center;
  border-radius: 13px;
  background: var(--sereno-accent);
}

.brand-mark svg { width: 23px; fill: none; stroke: #fff; stroke-width: 2.4; stroke-linecap: round; stroke-linejoin: round; }
.brand-copy { display: flex; flex-direction: column; }
.brand-copy strong { font-size: 23px; line-height: 1; letter-spacing: -.8px; }
.brand-copy small { margin-top: 5px; color: var(--sereno-muted); font-size: 8px; font-weight: 700; letter-spacing: .45px; }
.side-label { margin: 0 10px 12px; color: var(--sereno-muted); font-size: 10px; font-weight: 750; letter-spacing: .8px; }
.side-nav { display: grid; gap: 5px; }

.nav-link {
  display: flex;
  min-height: 48px;
  align-items: center;
  gap: 14px;
  padding: 0 13px;
  border: 0;
  border-radius: 12px;
  background: transparent;
  color: var(--sereno-muted);
  font-size: 14px;
  text-align: left;
  text-decoration: none;
  transition: background .18s ease, color .18s ease;
}

.nav-link:hover { background: var(--sereno-soft); color: var(--sereno-accent); }
.nav-link.active { background: var(--sereno-accent); color: #fff; font-weight: 650; }
.nav-link svg { width: 18px; height: 18px; flex: 0 0 18px; fill: none; stroke: currentColor; stroke-width: 1.7; stroke-linecap: round; stroke-linejoin: round; }

.sidebar-progress {
  margin: 37px 4px 0;
  padding: 20px 6px 0;
  border-top: 1px solid var(--sereno-border);
}

.sidebar-progress .side-label { margin: 0 0 13px; }
.sidebar-progress strong { font-size: 14px; }
.sidebar-progress small { display: block; margin-top: 12px; color: var(--sereno-muted); font-size: 11px; }
.progress-track { height: 9px; margin-top: 14px; overflow: hidden; border-radius: 8px; background: #e7ece8; }
.progress-track span { display: block; height: 100%; border-radius: inherit; background: var(--sereno-accent); transition: width .25s ease; }

.sidebar-profile {
  display: flex;
  align-items: center;
  gap: 11px;
  margin: auto 4px 0;
  padding: 17px 3px 0;
  border-top: 1px solid var(--sereno-border);
}

.avatar { display: grid; width: 40px; height: 40px; flex: 0 0 40px; place-items: center; border-radius: 50%; background: #e49b76; color: #fff; font-size: 12px; font-weight: 700; }
.sidebar-profile span:last-child { display: grid; gap: 3px; }
.sidebar-profile strong { font-size: 12px; }
.sidebar-profile small { color: var(--sereno-muted); font-size: 10px; }
.main-content { min-width: 0; padding: 0 8px 8px 0; }
.topbar { display: flex; height: 54px; align-items: center; justify-content: space-between; }
.today-label { color: var(--sereno-muted); font-size: 11px; font-weight: 750; letter-spacing: .7px; text-transform: uppercase; }
.top-avatar { width: 42px; height: 42px; }
.welcome-section { display: flex; align-items: end; justify-content: space-between; gap: 16px; margin: 20px 0 33px; }
.eyebrow { margin: 0 0 8px; color: var(--sereno-accent); font-size: 10px; font-weight: 750; letter-spacing: 1.1px; }
.welcome-section h1 { margin: 0; color: var(--sereno-ink); font-size: clamp(29px, 3vw, 36px); letter-spacing: -1.2px; line-height: 1.2; }
.welcome-copy { margin: 8px 0 0; color: var(--sereno-muted); font-size: 15px; }
.date-chip { padding: 9px 16px; border-radius: 20px; background: #eaf3ee; color: var(--sereno-accent); font-size: 11px; font-weight: 700; text-transform: capitalize; white-space: nowrap; }

.summary-cards { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 20px; }
.summary-card { display: flex; min-height: 120px; align-items: center; gap: 17px; padding: 20px; border: 1px solid var(--sereno-border); border-radius: 18px; background: #fff; }
.summary-icon { display: grid; width: 43px; height: 43px; flex: 0 0 43px; place-items: center; border-radius: 15px; font-size: 20px; font-weight: 700; }
.total-icon { background: #eaf3ee; color: var(--sereno-accent); }
.active-icon { background: #fff2d8; color: #b57d2f; }
.done-icon { background: #e5f3ec; color: #648d71; }
.summary-card div { display: grid; grid-template-columns: auto 1fr; align-items: baseline; column-gap: 12px; }
.summary-card strong { grid-row: span 2; color: var(--sereno-ink); font-size: 31px; line-height: 1; }
.summary-card div span { color: var(--sereno-ink); font-size: 13px; font-weight: 700; }
.summary-card div small { color: var(--sereno-muted); font-size: 11px; }

.tasks-section { margin-top: 34px; }
.tasks-heading { display: flex; align-items: end; justify-content: space-between; gap: 18px; }
.tasks-heading h2 { margin: 0; color: var(--sereno-ink); font-size: 22px; letter-spacing: -.4px; }
.tasks-heading p { margin: 5px 0 0; color: var(--sereno-muted); font-size: 12px; }
.search-field { display: flex; width: min(320px, 42%); height: 46px; align-items: center; gap: 10px; padding: 0 14px; border: 1px solid var(--sereno-border); border-radius: 12px; background: #fff; }
.search-field:focus-within { border-color: #9ac5b5; box-shadow: 0 0 0 3px #eaf3ee; }
.search-field svg { width: 19px; height: 19px; flex: 0 0 19px; fill: none; stroke: var(--sereno-muted); stroke-width: 1.8; stroke-linecap: round; }
.search-field input { width: 100%; min-width: 0; border: 0; outline: 0; background: transparent; color: var(--sereno-ink); font-size: 12px; }
.search-field input::placeholder { color: var(--sereno-muted); }

.task-toolbar { display: flex; min-height: 62px; align-items: center; justify-content: space-between; gap: 12px; margin-top: 20px; padding: 8px 12px; border-radius: 16px; background: var(--sereno-soft); }
.filter-tabs { display: flex; flex-wrap: wrap; gap: 5px; }
.filter-tabs button { min-height: 40px; padding: 0 14px; border: 0; border-radius: 11px; background: transparent; color: var(--sereno-muted); font-size: 12px; }
.filter-tabs button.selected { background: #fff; color: var(--sereno-accent); font-weight: 700; box-shadow: 0 1px 3px #1e30290c; }
.filter-tabs button span { margin-left: 4px; }
.add-task-button { display: inline-flex; min-height: 40px; flex: 0 0 auto; align-items: center; justify-content: center; gap: 7px; padding: 0 17px; border: 0; border-radius: 22px; background: var(--sereno-accent); color: #fff; font-size: 12px; font-weight: 700; transition: background .18s ease, transform .18s ease; }
.add-task-button:hover { transform: translateY(-1px); background: #236d5a; }
.add-task-button span { font-size: 18px; line-height: 1; }

.new-task-form { margin-top: 14px; padding: 22px; border: 1px solid var(--sereno-border); border-radius: 18px; background: #fff; box-shadow: 0 12px 30px #1e30290a; }
.form-heading { display: flex; align-items: start; justify-content: space-between; margin-bottom: 20px; }
.form-heading h3 { margin: 0; font-size: 17px; }
.form-heading p { margin: 4px 0 0; color: var(--sereno-muted); font-size: 12px; }
.close-form { width: 32px; height: 32px; border: 0; border-radius: 50%; background: var(--sereno-soft); color: var(--sereno-muted); font-size: 21px; line-height: 1; }
.form-control-group { display: grid; gap: 7px; color: var(--sereno-ink); font-size: 12px; font-weight: 650; }
.form-control-group input { width: 100%; min-height: 43px; padding: 0 12px; border: 1px solid var(--sereno-border); border-radius: 10px; outline: 0; background: #fff; color: var(--sereno-ink); }
.form-control-group input:focus { border-color: #9ac5b5; box-shadow: 0 0 0 3px #eaf3ee; }
.form-control-group input::placeholder { color: #a4ada7; }
.form-row { display: grid; grid-template-columns: minmax(170px, .7fr) 1.3fr; align-items: end; gap: 22px; margin-top: 16px; }
.priority-choice { display: flex; flex-wrap: wrap; align-items: center; gap: 8px; margin: 0; padding: 0; border: 0; }
.priority-choice legend { width: 100%; margin-bottom: 7px; color: var(--sereno-ink); font-size: 12px; font-weight: 650; }
.priority-choice label { cursor: pointer; }
.priority-choice input { position: absolute; width: 1px; height: 1px; opacity: 0; }
.priority-choice input:focus-visible + span { outline: 2px solid var(--sereno-accent); outline-offset: 2px; }
.priority-choice span { display: inline-flex; min-height: 34px; align-items: center; padding: 0 12px; border: 1px solid transparent; border-radius: 18px; font-size: 11px; font-weight: 700; }
.priority-choice input:not(:checked) + span { border-color: var(--sereno-border); background: #fff; color: var(--sereno-muted); }
.form-error { margin: 12px 0 0; color: #b65847; font-size: 12px; }
.form-actions { display: flex; justify-content: end; gap: 10px; margin-top: 20px; }
.cancel-button { min-height: 40px; padding: 0 15px; border: 1px solid var(--sereno-border); border-radius: 20px; background: #fff; color: var(--sereno-muted); font-size: 12px; }

.task-list { margin-top: 16px; overflow: hidden; border: 1px solid var(--sereno-border); border-radius: 18px; background: #fff; }
.task-list-header, .task-row { display: grid; grid-template-columns: 24px minmax(0, 1fr) 150px 150px 36px; align-items: center; gap: 16px; padding: 0 22px; }
.task-list-header { min-height: 43px; border-bottom: 1px solid var(--sereno-border); color: var(--sereno-muted); font-size: 10px; font-weight: 700; letter-spacing: .5px; text-transform: uppercase; }
.task-row { min-height: 70px; border-bottom: 1px solid var(--sereno-border); }
.task-row:last-child { border-bottom: 0; }
.task-checkbox { display: none; }
.task-list-header span:first-child { grid-column: 2; }
.task-list-header span:nth-child(2) { grid-column: 3; }
.task-list-header span:nth-child(3) { grid-column: 4; }
.task-list-header .sr-only { grid-column: 5; }
.task-checkbox { display: grid; width: 23px; height: 23px; place-items: center; padding: 0; border: 1.5px solid #d5ded8; border-radius: 7px; background: #fff; color: #fff; }
.task-checkbox:hover { border-color: var(--sereno-accent); }
.task-checkbox.checked { border-color: var(--sereno-accent); background: var(--sereno-accent); }
.task-checkbox svg { width: 17px; fill: none; stroke: currentColor; stroke-width: 2; stroke-linecap: round; stroke-linejoin: round; }
.task-title { display: grid; min-width: 0; gap: 4px; }
.task-title strong { overflow: hidden; color: var(--sereno-ink); font-size: 13px; font-weight: 600; text-overflow: ellipsis; white-space: nowrap; }
.task-date { color: var(--sereno-muted); font-size: 11px; text-transform: capitalize; }
.task-title.completed strong { color: #929b95; text-decoration: line-through; }
.priority-pill { display: inline-flex; width: fit-content; min-width: 78px; min-height: 29px; align-items: center; justify-content: center; padding: 0 12px; border-radius: 20px; font-size: 11px; font-weight: 700; }
.priority-alta { background: #fce9e5; color: #b65847; }
.priority-media { background: #fff2d8; color: #a87522; }
.priority-baja { background: #e5f3ec; color: #438364; }
.task-priority-controls { display: flex; align-items: center; gap: 7px; }
.task-priority-controls select { width: 24px; height: 30px; padding: 0; border: 0; outline: 0; background: transparent; color: var(--sereno-muted); cursor: pointer; font-size: 11px; }
.delete-task-button { display: grid; width: 32px; height: 32px; place-items: center; border: 0; border-radius: 9px; background: transparent; color: #98a39c; opacity: .7; transition: background .18s ease, color .18s ease, opacity .18s ease; }
.delete-task-button:hover { background: #fce9e5; color: #b65847; opacity: 1; }
.delete-task-button svg { width: 17px; fill: none; stroke: currentColor; stroke-width: 1.7; stroke-linecap: round; stroke-linejoin: round; }
.task-list-header .sr-only { position: absolute; width: 1px; height: 1px; overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; }

.empty-state { display: grid; justify-items: center; padding: 52px 20px; text-align: center; }
.empty-icon { display: grid; width: 48px; height: 48px; place-items: center; border-radius: 16px; background: #eaf3ee; color: var(--sereno-accent); font-size: 23px; font-weight: 700; }
.empty-state h3 { margin: 15px 0 4px; color: var(--sereno-ink); font-size: 16px; }
.empty-state p { margin: 0; color: var(--sereno-muted); font-size: 12px; }
.text-action { margin-top: 16px; padding: 0; border: 0; background: transparent; color: var(--sereno-accent); font-size: 12px; font-weight: 700; }
.completed-actions { display: flex; align-items: center; justify-content: space-between; margin-top: 13px; color: var(--sereno-muted); font-size: 11px; }
.list-actions { display: flex; align-items: center; gap: 16px; }
.completed-actions button { padding: 7px 0 7px 10px; border: 0; background: transparent; color: #b65847; font-size: 11px; font-weight: 650; }
.completed-actions .clear-all-button { color: var(--sereno-muted); }
.completed-actions .clear-all-button:hover { color: #b65847; }
.inspiration-card { display: flex; min-height: 90px; align-items: center; gap: 15px; margin-top: 22px; padding: 18px 22px; border-radius: 18px; background: #eaf3ee; }
.inspiration-icon { display: grid; width: 38px; height: 38px; flex: 0 0 38px; place-items: center; border-radius: 13px; background: #fff; color: var(--sereno-accent); font-size: 19px; }
.inspiration-card strong { color: var(--sereno-ink); font-size: 14px; }
.inspiration-card p { margin: 4px 0 0; color: var(--sereno-muted); font-size: 11px; }
.inspiration-sparkle { margin-left: auto; color: var(--sereno-accent); font-size: 28px; }

@media (min-width: 1500px) {
  .sereno-app { grid-template-columns: 250px minmax(0, 1fr); gap: 40px; padding: 18px; }
}

@media (max-width: 1000px) {
  .sereno-app { grid-template-columns: 205px minmax(0, 1fr); gap: 22px; width: min(100% - 28px, 1000px); }
  .sidebar { padding-inline: 12px; }
  .summary-cards { gap: 10px; }
  .summary-card { gap: 10px; padding: 15px 12px; }
  .summary-card div { grid-template-columns: 1fr; gap: 3px; }
  .summary-card strong { grid-row: auto; font-size: 25px; }
  .summary-card div small { display: none; }
  .task-list-header, .task-row { grid-template-columns: 23px minmax(0, 1fr) 115px 125px 30px; gap: 10px; padding-inline: 14px; }
}

@media (max-width: 720px) {
  .sereno-app { display: block; width: 100%; min-height: 100vh; margin: 0; padding: 12px; border: 0; border-radius: 0; }
  .sidebar { position: static; min-height: auto; padding: 13px 14px; }
  .brand { margin: 0 2px 17px; }
  .sidebar > .side-label, .sidebar-progress, .sidebar-profile { display: none; }
  .side-nav { grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 5px; }
  .nav-link { min-height: 42px; justify-content: center; gap: 6px; padding: 0 5px; font-size: 10px; }
  .nav-link svg { width: 15px; height: 15px; flex-basis: 15px; }
  .main-content { padding: 0 3px 12px; }
  .topbar { height: 48px; }
  .today-label { font-size: 9px; }
  .top-avatar { width: 34px; height: 34px; flex-basis: 34px; }
  .welcome-section { align-items: start; margin: 20px 0 22px; }
  .welcome-section h1 { font-size: 29px; }
  .welcome-copy { max-width: 300px; font-size: 13px; }
  .date-chip { display: none; }
  .summary-cards { gap: 8px; }
  .summary-card { min-height: 95px; flex-direction: column; align-items: start; gap: 7px; padding: 11px; border-radius: 14px; }
  .summary-icon { width: 30px; height: 30px; flex-basis: 30px; border-radius: 10px; font-size: 15px; }
  .summary-card div { display: flex; flex-wrap: wrap; align-items: baseline; gap: 5px; }
  .summary-card strong { font-size: 22px; }
  .summary-card div span { font-size: 10px; }
  .tasks-section { margin-top: 26px; }
  .tasks-heading { align-items: stretch; flex-direction: column; }
  .tasks-heading h2 { font-size: 20px; }
  .search-field { width: 100%; }
  .task-toolbar { align-items: stretch; flex-direction: column; padding: 9px; }
  .filter-tabs { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); }
  .filter-tabs button { padding: 0 5px; font-size: 10px; }
  .add-task-button { min-height: 42px; }
  .task-list { border-radius: 14px; }
  .task-list-header { display: none; }
  .task-row { grid-template-columns: 23px minmax(0, 1fr) auto 30px; grid-template-rows: 1fr 1fr; gap: 0 8px; min-height: 69px; padding: 8px 10px; }
  .task-checkbox { grid-column: 1; grid-row: 1 / 3; }
  .task-title { grid-column: 2; grid-row: 1; align-self: end; }
  .task-date { grid-column: 2; grid-row: 2; align-self: start; font-size: 10px; }
  .task-priority-controls { grid-column: 3; grid-row: 1 / 3; }
  .delete-task-button { grid-column: 4; grid-row: 1 / 3; }
  .priority-pill { min-width: 60px; min-height: 25px; padding: 0 8px; font-size: 10px; }
  .task-priority-controls select { width: 14px; font-size: 9px; }
  .task-title strong { font-size: 11px; }
  .delete-task-button { width: 28px; }
  .new-task-form { padding: 17px 13px; }
  .form-row { grid-template-columns: 1fr; gap: 14px; }
  .priority-choice { gap: 6px; }
  .completed-actions { gap: 12px; }
  .inspiration-card { align-items: start; padding: 15px; }
  .inspiration-card strong { font-size: 12px; }
  .inspiration-card p { font-size: 10px; }
  .inspiration-sparkle { display: none; }
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { scroll-behavior: auto !important; transition-duration: .01ms !important; }
}
</style>
