<template>
  <div class="container">
    <div class="hero is-primary is-bold mb-5">
      <div class="hero-body">
        <h1 class="title">Gestión de Períodos de Asistencia</h1>
        <p class="subtitle">Administra períodos mensuales con apertura automática</p>
      </div>
    </div>

    <div v-if="periodStore.error" class="notification is-danger is-light">
      <button class="delete" @click="periodStore.error = null"></button>
      <strong>Error:</strong> {{ periodStore.error }}
    </div>

    <div class="columns">
      <div class="column">
        <div class="box">
          <button class="button is-primary is-medium" @click="showCreateModal = true">
            <span class="icon">
              <i class="mdi mdi-plus-circle"></i>
            </span>
            <span>Crear Nuevo Período</span>
          </button>
        </div>
      </div>
    </div>

    <div class="box">
      <h2 class="title is-5 mb-4">Filtros de Búsqueda</h2>
      <div class="columns">
        <div class="column is-3">
          <div class="field">
            <label class="label">Mes</label>
            <div class="control has-icons-left">
              <div class="select is-fullwidth">
                <select v-model="filters.mes">
                  <option :value="undefined">Todos los meses</option>
                  <option v-for="m in 12" :key="m" :value="m">{{ getMonthName(m) }}</option>
                </select>
              </div>
              <span class="icon is-left">
                <i class="mdi mdi-calendar-month"></i>
              </span>
            </div>
          </div>
        </div>
        <div class="column is-3">
          <div class="field">
            <label class="label">Año</label>
            <div class="control has-icons-left">
              <div class="select is-fullwidth">
                <select v-model="filters.anio">
                  <option :value="undefined">Todos los años</option>
                  <option v-for="y in years" :key="y" :value="y">{{ y }}</option>
                </select>
              </div>
              <span class="icon is-left">
                <i class="mdi mdi-calendar"></i>
              </span>
            </div>
          </div>
        </div>
        <div class="column is-3">
          <div class="field">
            <label class="label">Estado</label>
            <div class="control has-icons-left">
              <div class="select is-fullwidth">
                <select v-model="filters.is_open">
                  <option :value="undefined">Todos</option>
                  <option :value="true">Abiertos</option>
                  <option :value="false">Cerrados</option>
                </select>
              </div>
              <span class="icon is-left">
                <i class="mdi mdi-filter"></i>
              </span>
            </div>
          </div>
        </div>
        <div class="column is-3">
          <div class="field">
            <label class="label">&nbsp;</label>
            <button class="button is-info is-fullwidth" @click="loadPeriods">
              <span class="icon">
                <i class="mdi mdi-magnify"></i>
              </span>
              <span>Buscar</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="box">
      <div v-if="periodStore.loading" class="has-text-centered py-6">
        <span class="icon is-large has-text-primary">
          <i class="mdi mdi-loading mdi-spin mdi-48px"></i>
        </span>
        <p class="mt-3">Cargando períodos...</p>
      </div>

      <div v-else-if="periodStore.periods.length === 0" class="has-text-centered py-6">
        <span class="icon is-large has-text-grey-light">
          <i class="mdi mdi-calendar-blank mdi-48px"></i>
        </span>
        <p class="mt-3 has-text-grey">No hay períodos registrados</p>
      </div>

      <div v-else class="table-container">
        <table class="table is-fullwidth is-striped is-hoverable">
          <thead>
            <tr>
              <th>Período</th>
              <th>Fecha</th>
              <th>Horario</th>
              <th class="has-text-centered">Duración</th>
              <th class="has-text-centered">Estado</th>
              <th class="has-text-centered">Asistencias</th>
              <th class="has-text-centered">Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="period in periodStore.periods" :key="period.id">
              <td>
                <strong>{{ period.nombre }}</strong>
                <br>
                <small class="has-text-grey">{{ period.descripcion }}</small>
              </td>
              <td>{{ formatDate(period.fecha_asistencia) }}</td>
              <td>
                <span class="icon-text">
                  <span class="icon has-text-info">
                    <i class="mdi mdi-clock-start"></i>
                  </span>
                  <span>{{ period.hora_inicio.substring(0, 5) }}</span>
                </span>
                <br>
                <span class="icon-text">
                  <span class="icon has-text-danger">
                    <i class="mdi mdi-clock-end"></i>
                  </span>
                  <span>{{ period.hora_fin.substring(0, 5) }}</span>
                </span>
              </td>
              <td class="has-text-centered">
                <div v-if="!period.is_open" class="dropdown is-hoverable">
                  <div class="dropdown-trigger">
                    <button class="button is-small is-light">
                      <span class="icon">
                        <i class="mdi mdi-timer"></i>
                      </span>
                      <span>{{ calculateDuration(period.hora_inicio, period.hora_fin) }}</span>
                      <span class="icon is-small">
                        <i class="mdi mdi-chevron-down"></i>
                      </span>
                    </button>
                  </div>
                  <div class="dropdown-menu">
                    <div class="dropdown-content">
                      <a 
                        v-for="shortcut in periodStore.durationShortcuts" 
                        :key="shortcut.label"
                        class="dropdown-item"
                        @click="extendDuration(period.id, shortcut.minutes)"
                      >
                        <span class="icon">
                          <i class="mdi mdi-plus"></i>
                        </span>
                        {{ shortcut.label }}
                      </a>
                    </div>
                  </div>
                </div>
                <span v-else class="tag is-medium">
                  <span class="icon">
                    <i class="mdi mdi-timer"></i>
                  </span>
                  <span>{{ calculateDuration(period.hora_inicio, period.hora_fin) }}</span>
                </span>
              </td>
              <td class="has-text-centered">
                <span v-if="periodStore.shouldBeOpen(period)" class="tag is-warning is-medium">
                  <span class="icon">
                    <i class="mdi mdi-lock-open-variant"></i>
                  </span>
                  <span>Apertura automática</span>
                </span>
                <span v-else-if="period.is_open && periodStore.isOpenOutOfTime(period)" class="tag is-warning is-medium">
                  <span class="icon">
                    <i class="mdi mdi-alert-circle"></i>
                  </span>
                  <span>Abierto fuera de tiempo</span>
                </span>
                <span v-else-if="period.is_open" class="tag is-success is-medium">
                  <span class="icon">
                    <i class="mdi mdi-lock-open"></i>
                  </span>
                  <span>Abierto</span>
                </span>
                <span v-else-if="periodStore.getPeriodStatus(period) === 'programado'" class="tag is-info is-medium">
                  <span class="icon">
                    <i class="mdi mdi-clock-outline"></i>
                  </span>
                  <span>Programado</span>
                </span>
                <span v-else-if="periodStore.getPeriodStatus(period) === 'en_curso'" class="tag is-primary is-medium">
                  <span class="icon">
                    <i class="mdi mdi-clock"></i>
                  </span>
                  <span>En Curso</span>
                </span>
                <span v-else-if="periodStore.getPeriodStatus(period) === 'finalizado'" class="tag is-dark is-medium">
                  <span class="icon">
                    <i class="mdi mdi-check-circle"></i>
                  </span>
                  <span>Finalizado</span>
                </span>
                <span v-else class="tag is-danger is-medium">
                  <span class="icon">
                    <i class="mdi mdi-close-circle"></i>
                  </span>
                  <span>Inactivo</span>
                </span>
              </td>
              <td class="has-text-centered">
                <div class="field is-grouped is-grouped-centered">
                  <p class="control">
                    <span class="tag is-large">
                      <strong>{{ period.total_marked }}</strong> / {{ period.total_expected }}
                    </span>
                  </p>
                </div>
                <div v-if="period.total_expected > 0" class="mt-2">
                  <progress 
                    class="progress is-small" 
                    :class="{
                      'is-success': Math.round((period.total_marked / period.total_expected) * 100) >= 80,
                      'is-warning': Math.round((period.total_marked / period.total_expected) * 100) >= 50 && Math.round((period.total_marked / period.total_expected) * 100) < 80,
                      'is-danger': Math.round((period.total_marked / period.total_expected) * 100) < 50
                    }"
                    :value="period.total_marked" 
                    :max="period.total_expected"
                  >
                    {{ Math.round((period.total_marked / period.total_expected) * 100) }}%
                  </progress>
                  <small class="has-text-grey">{{ Math.round((period.total_marked / period.total_expected) * 100) }}%</small>
                </div>
              </td>
              <td>
                <div class="buttons are-small is-centered">
                  <button 
                    class="button is-primary"
                    @click="viewPeriodDetails(period.id)"
                    title="Ver Detalles"
                  >
                    <span class="icon">
                      <i class="mdi mdi-eye"></i>
                    </span>
                  </button>
                  
                  <button 
                    class="button is-info"
                    @click="viewStats(period.id)"
                    title="Estadísticas"
                  >
                    <span class="icon">
                      <i class="mdi mdi-chart-bar"></i>
                    </span>
                  </button>
                  
                  <button 
                    v-if="!period.is_open && period.is_active" 
                    class="button is-success"
                    @click="openPeriod(period.id)"
                    :disabled="periodStore.loading"
                    title="Abrir Período"
                  >
                    <span class="icon">
                      <i class="mdi mdi-lock-open"></i>
                    </span>
                  </button>
                  
                  <button 
                    v-if="period.is_open" 
                    class="button is-warning"
                    @click="closePeriod(period.id)"
                    :disabled="periodStore.loading"
                    title="Cerrar Período"
                  >
                    <span class="icon">
                      <i class="mdi mdi-lock"></i>
                    </span>
                  </button>
                  
                  <button 
                    v-if="periodStore.getPeriodStatus(period) !== 'en_curso'"
                    class="button is-light"
                    @click="editPeriod(period)"
                    title="Editar"
                  >
                    <span class="icon">
                      <i class="mdi mdi-pencil"></i>
                    </span>
                  </button>
                  
                  <button 
                    class="button is-danger"
                    @click="confirmDelete(period.id)"
                    :disabled="periodStore.loading || period.is_open"
                    title="Eliminar"
                  >
                    <span class="icon">
                      <i class="mdi mdi-delete"></i>
                    </span>
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div class="modal" :class="{ 'is-active': showCreateModal || showEditModal }">
      <div class="modal-background" @click="closeModal"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">
            <span class="icon-text">
              <span class="icon">
                <i class="mdi mdi-calendar-plus"></i>
              </span>
              <span>{{ showEditModal ? 'Editar Período' : 'Crear Nuevo Período' }}</span>
            </span>
          </p>
          <button class="delete" @click="closeModal"></button>
        </header>
        <section class="modal-card-body">
          <div class="field">
            <label class="label">Nombre del Período *</label>
            <div class="control has-icons-left">
              <input 
                class="input" 
                type="text" 
                placeholder="Ej: Reunión Mensual - Enero 2025" 
                v-model="formData.nombre"
              >
              <span class="icon is-left">
                <i class="mdi mdi-format-title"></i>
              </span>
            </div>
          </div>

          <div class="field">
            <label class="label">Descripción</label>
            <div class="control">
              <textarea 
                class="textarea" 
                placeholder="Descripción opcional del período..."
                v-model="formData.descripcion"
                rows="2"
              ></textarea>
            </div>
          </div>

          <div class="columns">
            <div class="column is-6">
              <div class="field">
                <label class="label">Mes *</label>
                <div class="control has-icons-left">
                  <div class="select is-fullwidth">
                    <select v-model="formData.mes">
                      <option v-for="m in 12" :key="m" :value="m">{{ getMonthName(m) }}</option>
                    </select>
                  </div>
                  <span class="icon is-left">
                    <i class="mdi mdi-calendar-month"></i>
                  </span>
                </div>
              </div>
            </div>
            <div class="column is-6">
              <div class="field">
                <label class="label">Año *</label>
                <div class="control has-icons-left">
                  <div class="select is-fullwidth">
                    <select v-model="formData.anio">
                      <option v-for="y in years" :key="y" :value="y">{{ y }}</option>
                    </select>
                  </div>
                  <span class="icon is-left">
                    <i class="mdi mdi-calendar"></i>
                  </span>
                </div>
              </div>
            </div>
          </div>

          <div class="field">
            <label class="label">Fecha de Asistencia *</label>
            <div class="control has-icons-left">
              <input 
                class="input" 
                type="date" 
                v-model="formData.fecha_asistencia"
                :min="minDate"
              >
              <span class="icon is-left">
                <i class="mdi mdi-calendar-today"></i>
              </span>
            </div>
            <p v-if="isDateInPast" class="help is-danger">
              La fecha seleccionada está en el pasado
            </p>
          </div>

          <div class="columns">
            <div class="column is-6">
              <div class="field">
                <label class="label">Hora de Inicio *</label>
                <div class="control has-icons-left">
                  <input 
                    class="input" 
                    type="time" 
                    v-model="formData.hora_inicio"
                    :min="isToday ? minTime : undefined"
                  >
                  <span class="icon is-left">
                    <i class="mdi mdi-clock-start"></i>
                  </span>
                </div>
                <p v-if="isTimeInPast" class="help is-danger">
                  La hora seleccionada ya pasó
                </p>
              </div>
            </div>
            <div class="column is-6">
              <div class="field">
                <label class="label">Hora de Fin *</label>
                <div class="control has-icons-left">
                  <input 
                    class="input" 
                    type="time" 
                    v-model="formData.hora_fin"
                  >
                  <span class="icon is-left">
                    <i class="mdi mdi-clock-end"></i>
                  </span>
                </div>
              </div>
            </div>
          </div>

          <div class="notification is-info is-light">
            <p><strong>Nota:</strong> Los períodos en curso se abrirán automáticamente para registro de asistencia.</p>
          </div>
        </section>
        <footer class="modal-card-foot">
          <button 
            class="button is-primary" 
            @click="savePeriod"
            :disabled="!canSavePeriod || periodStore.loading"
          >
            <span class="icon">
              <i class="mdi mdi-content-save"></i>
            </span>
            <span v-if="periodStore.loading">Guardando...</span>
            <span v-else>{{ showEditModal ? 'Actualizar' : 'Crear Período' }}</span>
          </button>
          <button class="button" @click="closeModal">Cancelar</button>
        </footer>
      </div>
    </div>

    <div class="modal" :class="{ 'is-active': showStatsModal }">
      <div class="modal-background" @click="showStatsModal = false"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">
            <span class="icon-text">
              <span class="icon">
                <i class="mdi mdi-chart-line"></i>
              </span>
              <span>Estadísticas del Período</span>
            </span>
          </p>
          <button class="delete" @click="showStatsModal = false"></button>
        </header>
        <section class="modal-card-body" v-if="periodStore.periodStats">
          <div class="columns is-multiline">
            <div class="column is-6">
              <div class="box has-background-success-light">
                <p class="heading">Asistencias Válidas</p>
                <p class="title">{{ periodStore.periodStats.valid_attendances }}</p>
              </div>
            </div>
            <div class="column is-6">
              <div class="box has-background-warning-light">
                <p class="heading">Asistencias Inválidas</p>
                <p class="title">{{ periodStore.periodStats.invalid_attendances }}</p>
              </div>
            </div>
            <div class="column is-6">
              <div class="box has-background-info-light">
                <p class="heading">Total Registrado</p>
                <p class="title">{{ periodStore.periodStats.total_marked }}</p>
              </div>
            </div>
            <div class="column is-6">
              <div class="box has-background-light">
                <p class="heading">Total Esperado</p>
                <p class="title">{{ periodStore.periodStats.total_expected }}</p>
              </div>
            </div>
          </div>
        </section>
        <footer class="modal-card-foot">
          <button class="button" @click="showStatsModal = false">Cerrar</button>
        </footer>
      </div>
    </div>

    <div class="modal" :class="{ 'is-active': showDeleteModal }">
      <div class="modal-background" @click="showDeleteModal = false"></div>
      <div class="modal-card">
        <header class="modal-card-head has-background-danger">
          <p class="modal-card-title has-text-white">
            <span class="icon-text">
              <span class="icon">
                <i class="mdi mdi-alert-circle"></i>
              </span>
              <span>Confirmar Eliminación</span>
            </span>
          </p>
          <button class="delete" @click="showDeleteModal = false"></button>
        </header>
        <section class="modal-card-body">
          <article class="message is-danger">
            <div class="message-body">
              <p class="mb-3">¿Está seguro de que desea eliminar este período?</p>
              <p><strong>Esta acción no se puede deshacer.</strong></p>
            </div>
          </article>
        </section>
        <footer class="modal-card-foot">
          <button 
            class="button is-danger" 
            @click="deletePeriod"
            :disabled="periodStore.loading"
          >
            <span class="icon">
              <i class="mdi mdi-delete"></i>
            </span>
            <span v-if="periodStore.loading">Eliminando...</span>
            <span v-else>Eliminar</span>
          </button>
          <button class="button" @click="showDeleteModal = false">Cancelar</button>
        </footer>
      </div>
    </div>
  </div>
</template>

<script setup>
import dayjs from 'dayjs'
import 'dayjs/locale/es'
import { useAttendancePeriodStore } from '~/stores/attendancePeriod'

dayjs.locale('es')

definePageMeta({
  layout: 'dashboard',
  middleware: 'auth'
})

const router = useRouter()
const periodStore = useAttendancePeriodStore()

const showCreateModal = ref(false)
const showEditModal = ref(false)
const showStatsModal = ref(false)
const showDeleteModal = ref(false)
const periodToDelete = ref(null)
const editingPeriodId = ref(null)

const filters = ref({
  mes: null,
  anio: new Date().getFullYear(),
  is_open: undefined
})

const formData = ref({
  nombre: '',
  descripcion: '',
  mes: new Date().getMonth() + 1,
  anio: new Date().getFullYear(),
  fecha_asistencia: '',
  hora_inicio: '07:00',
  hora_fin: '19:00',
  is_active: true
})

const years = computed(() => {
  const currentYear = new Date().getFullYear()
  return [currentYear - 1, currentYear, currentYear + 1]
})

const minDate = computed(() => dayjs().format('YYYY-MM-DD'))
const minTime = computed(() => dayjs().format('HH:mm'))

const isToday = computed(() => {
  if (!formData.value.fecha_asistencia) return false
  return dayjs(formData.value.fecha_asistencia).isSame(dayjs(), 'day')
})

const isDateInPast = computed(() => {
  if (!formData.value.fecha_asistencia) return false
  return dayjs(formData.value.fecha_asistencia).isBefore(dayjs(), 'day')
})

const isTimeInPast = computed(() => {
  if (!isToday.value || !formData.value.hora_inicio) return false
  const selectedDateTime = dayjs(`${formData.value.fecha_asistencia}T${formData.value.hora_inicio}`)
  return selectedDateTime.isBefore(dayjs())
})

const canSavePeriod = computed(() => {
  return !!formData.value.nombre &&
         !!formData.value.fecha_asistencia &&
         !!formData.value.hora_inicio &&
         !!formData.value.hora_fin &&
         !isDateInPast.value &&
         !isTimeInPast.value
})

const getMonthName = (month) => {
  const months = [
    'Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio',
    'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre'
  ]
  return months[month - 1]
}

const formatDate = (dateString) => {
  const date = dayjs(`${dateString}T00:00:00`)
  return date.locale('es').format('dddd, D [de] MMMM [de] YYYY')
}

const calculateDuration = (startTime, endTime) => {
  if (!startTime || !endTime) return ''
  const start = dayjs(`2000-01-01T${startTime}`)
  let end = dayjs(`2000-01-01T${endTime}`)

  if (end.isBefore(start)) {
    end = end.add(1, 'day')
  }

  const diffMinutes = end.diff(start, 'minute')
  const hours = Math.floor(diffMinutes / 60)
  const minutes = diffMinutes % 60

  if (hours > 0) {
    return minutes > 0 ? `${hours}h ${minutes}m` : `${hours}h`
  }
  return `${minutes}m`
}

const loadPeriods = async () => {
  try {
    await periodStore.fetchPeriods(filters.value)
  } catch (error) {
    console.error('Error al cargar períodos:', error)
  }
}

const openPeriod = async (periodId) => {
  if (!confirm('¿Abrir este período para que los cooperativistas puedan marcar asistencia?')) {
    return
  }

  try {
    await periodStore.openPeriod(periodId)
    await loadPeriods()
  } catch (error) {
    console.error('Error al abrir período:', error)
  }
}

const closePeriod = async (periodId) => {
  if (!confirm('¿Cerrar este período? Los cooperativistas ya no podrán marcar asistencia.')) {
    return
  }

  try {
    await periodStore.closePeriod(periodId)
    await loadPeriods()
  } catch (error) {
    console.error('Error al cerrar período:', error)
  }
}

const viewStats = async (periodId) => {
  try {
    await periodStore.fetchPeriodStats(periodId)
    showStatsModal.value = true
  } catch (error) {
    console.error('Error al obtener estadísticas:', error)
  }
}

const viewPeriodDetails = (periodId) => {
  router.push(`/asistencias/${periodId}`)
}

const editPeriod = (period) => {
  editingPeriodId.value = period.id
  formData.value = {
    nombre: period.nombre,
    descripcion: period.descripcion || '',
    mes: period.mes,
    anio: period.anio,
    fecha_asistencia: period.fecha_asistencia,
    hora_inicio: period.hora_inicio ? period.hora_inicio.substring(0, 5) : '07:00',
    hora_fin: period.hora_fin ? period.hora_fin.substring(0, 5) : '19:00',
    is_active: period.is_active
  }
  showEditModal.value = true
}

const savePeriod = async () => {
  try {
    const data = {
      nombre: formData.value.nombre,
      descripcion: formData.value.descripcion,
      mes: parseInt(formData.value.mes),
      anio: parseInt(formData.value.anio),
      fecha_asistencia: formData.value.fecha_asistencia,
      hora_inicio: formData.value.hora_inicio + ':00',
      hora_fin: formData.value.hora_fin + ':00',
      is_active: formData.value.is_active
    }

    if (showEditModal.value) {
      await periodStore.updatePeriod(editingPeriodId.value, data)
    } else {
      await periodStore.createPeriod(data)
    }

    closeModal()
    await loadPeriods()
  } catch (error) {
    console.error('Error al guardar período:', error)
  }
}

const confirmDelete = (periodId) => {
  periodToDelete.value = periodId
  showDeleteModal.value = true
}

const deletePeriod = async () => {
  try {
    await periodStore.deletePeriod(periodToDelete.value)
    showDeleteModal.value = false
    periodToDelete.value = null
    await loadPeriods()
  } catch (error) {
    console.error('Error al eliminar período:', error)
  }
}

const extendDuration = async (periodId, additionalMinutes) => {
  try {
    await periodStore.extendPeriodDuration(periodId, additionalMinutes)
    await loadPeriods()
  } catch (error) {
    console.error('Error al extender duración:', error)
  }
}

const closeModal = () => {
  showCreateModal.value = false
  showEditModal.value = false
  editingPeriodId.value = null
  formData.value = {
    nombre: '',
    descripcion: '',
    mes: new Date().getMonth() + 1,
    anio: new Date().getFullYear(),
    fecha_asistencia: '',
    hora_inicio: '07:00',
    hora_fin: '19:00',
    is_active: true
  }
}

onMounted(() => {
  loadPeriods()
})

useHead({
  title: 'Gestión de Períodos de Asistencia'
})
</script>