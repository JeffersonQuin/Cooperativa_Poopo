<template>
  <div class="container">
    <div class="hero is-info is-bold mb-5" v-if="periodStore.selectedPeriod">
      <div class="hero-body">
        <div class="level">
          <div class="level-left">
            <div class="level-item">
              <div>
                <h1 class="title has-text-white">{{ periodStore.selectedPeriod.nombre }}</h1>
                <p class="subtitle has-text-white-bis">{{ formatDate(periodStore.selectedPeriod.fecha_asistencia) }}</p>
              </div>
            </div>
          </div>
          <div class="level-right">
            <div class="level-item">
              <button class="button is-light" @click="router.back()">
                <span class="icon">
                  <i class="mdi mdi-arrow-left"></i>
                </span>
                <span>Volver</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="attendanceStore.error || periodStore.error || cooperativistaStore.error" class="notification is-danger is-light">
      <button class="delete" @click="clearErrors"></button>
      <strong>Error:</strong> {{ attendanceStore.error || periodStore.error || cooperativistaStore.error }}
    </div>

    <div class="box" v-if="periodStore.selectedPeriod">
      <div class="columns is-multiline">
        <div class="column is-6">
          <div class="field is-grouped is-grouped-multiline">
            <div class="control">
              <div class="tags has-addons">
                <span class="tag is-dark">
                  <span class="icon">
                    <i class="mdi mdi-clock-outline"></i>
                  </span>
                  <span>Horario</span>
                </span>
                <span class="tag is-info is-medium">
                  {{ periodStore.selectedPeriod.hora_inicio.substring(0, 5) }} - {{ periodStore.selectedPeriod.hora_fin.substring(0, 5) }}
                </span>
              </div>
            </div>
            <div class="control">
              <div class="tags has-addons">
                <span class="tag is-dark">
                  <span class="icon">
                    <i class="mdi mdi-information"></i>
                  </span>
                  <span>Estado</span>
                </span>
                <span v-if="periodStore.selectedPeriod.is_open" class="tag is-success is-medium">
                  <span class="icon">
                    <i class="mdi mdi-lock-open"></i>
                  </span>
                  <span>Abierto</span>
                </span>
                <span v-else class="tag is-dark is-medium">
                  <span class="icon">
                    <i class="mdi mdi-lock"></i>
                  </span>
                  <span>Cerrado</span>
                </span>
              </div>
            </div>
          </div>
        </div>
        <div class="column is-6">
          <div class="field is-grouped is-grouped-multiline is-pulled-right">
            <div class="control">
              <div class="tags has-addons">
                <span class="tag is-dark">
                  <span class="icon">
                    <i class="mdi mdi-account-multiple-check"></i>
                  </span>
                  <span>Asistencias</span>
                </span>
                <span class="tag is-primary is-medium">
                  <strong>{{ periodStore.selectedPeriod.total_marked }}</strong> / {{ periodStore.selectedPeriod.total_expected }}
                </span>
              </div>
            </div>
            <div class="control" v-if="periodStore.selectedPeriod.total_expected > 0">
              <div class="tags has-addons">
                <span class="tag is-dark">
                  <span class="icon">
                    <i class="mdi mdi-percent"></i>
                  </span>
                  <span>Porcentaje</span>
                </span>
                <span class="tag is-medium" :class="{
                  'is-success': Math.round((periodStore.selectedPeriod.total_marked / periodStore.selectedPeriod.total_expected) * 100) >= 80,
                  'is-warning': Math.round((periodStore.selectedPeriod.total_marked / periodStore.selectedPeriod.total_expected) * 100) >= 50 && Math.round((periodStore.selectedPeriod.total_marked / periodStore.selectedPeriod.total_expected) * 100) < 80,
                  'is-danger': Math.round((periodStore.selectedPeriod.total_marked / periodStore.selectedPeriod.total_expected) * 100) < 50
                }">
                  {{ Math.round((periodStore.selectedPeriod.total_marked / periodStore.selectedPeriod.total_expected) * 100) }}%
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="box">
      <div class="level">
        <div class="level-left">
          <div class="level-item">
            <h2 class="title is-4">Lista de Asistencias</h2>
          </div>
        </div>
        <div class="level-right">
          <div class="level-item">
            <button class="button is-primary" @click="showAddModal = true">
              <span class="icon">
                <i class="mdi mdi-plus-circle"></i>
              </span>
              <span>Agregar Asistencia Manual</span>
            </button>
          </div>
        </div>
      </div>

      <div class="field">
        <label class="label">Buscar por CI</label>
        <div class="control has-icons-left has-icons-right">
          <input 
            class="input is-medium" 
            type="text" 
            placeholder="Ingrese CI del cooperativista..." 
            v-model="ciFilter"
          >
          <span class="icon is-left is-medium">
            <i class="mdi mdi-magnify"></i>
          </span>
          <span class="icon is-right is-medium" v-if="ciFilter">
            <a @click="ciFilter = ''">
              <i class="mdi mdi-close-circle"></i>
            </a>
          </span>
        </div>
      </div>
    </div>

    <div class="box">
      <div v-if="attendanceStore.loading" class="has-text-centered py-6">
        <span class="icon is-large has-text-primary">
          <i class="mdi mdi-loading mdi-spin mdi-48px"></i>
        </span>
        <p class="mt-3">Cargando asistencias...</p>
      </div>

      <div v-else-if="filteredAttendances.length === 0" class="has-text-centered py-6">
        <span class="icon is-large has-text-grey-light">
          <i class="mdi mdi-account-off mdi-48px"></i>
        </span>
        <p class="mt-3 has-text-grey">No hay asistencias registradas</p>
      </div>

      <div v-else class="table-container">
        <table class="table is-fullwidth is-striped is-hoverable">
          <thead>
            <tr>
              <th>CI</th>
              <th>Cooperativista</th>
              <th>Fecha</th>
              <th>Hora</th>
              <th>Ubicación GPS</th>
              <th class="has-text-centered">Distancia</th>
              <th class="has-text-centered">Estado</th>
              <th class="has-text-centered">Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="attendance in filteredAttendances" :key="attendance.id">
              <td>
                <span class="tag is-medium">{{ getCooperativistaById(attendance.cooperativista_id)?.ci || 'N/A' }}</span>
              </td>
              <td>
                <strong>{{ getCooperativistaById(attendance.cooperativista_id)?.nombres || 'Cooperativista no encontrado' }}</strong>
                <br>
                <small class="has-text-grey">{{ getCooperativistaById(attendance.cooperativista_id)?.apellidos || '' }}</small>
              </td>
              <td>{{ formatDate(attendance.fecha) }}</td>
              <td>
                <span class="icon-text">
                  <span class="icon has-text-info">
                    <i class="mdi mdi-clock"></i>
                  </span>
                  <span>{{ attendance.hora.substring(0, 5) }}</span>
                </span>
              </td>
              <td>
                <small class="has-text-grey-dark">
                  <span class="icon-text">
                    <span class="icon">
                      <i class="mdi mdi-map-marker"></i>
                    </span>
                    <span>{{ attendance.location_lat.toFixed(6) }}, {{ attendance.location_lon.toFixed(6) }}</span>
                  </span>
                </small>
              </td>
              <td class="has-text-centered">
                <span v-if="attendance.distance_meters !== null" class="tag is-medium" :class="{
                  'is-success': attendance.distance_meters <= 50,
                  'is-warning': attendance.distance_meters > 50 && attendance.distance_meters <= 100,
                  'is-danger': attendance.distance_meters > 100
                }">
                  <span class="icon">
                    <i class="mdi mdi-map-marker-distance"></i>
                  </span>
                  <span>{{ attendance.distance_meters }}m</span>
                </span>
                <span v-else class="tag is-medium">N/A</span>
              </td>
              <td class="has-text-centered">
                <span v-if="attendance.is_valid" class="tag is-success is-medium">
                  <span class="icon">
                    <i class="mdi mdi-check-circle"></i>
                  </span>
                  <span>Válida</span>
                </span>
                <span v-else class="tag is-warning is-medium">
                  <span class="icon">
                    <i class="mdi mdi-alert-circle"></i>
                  </span>
                  <span>Pendiente</span>
                </span>
              </td>
              <td>
                <div class="buttons are-small is-centered">
                  <button 
                    v-if="attendance.notes"
                    class="button is-info"
                    @click="showNotes(attendance.notes)"
                    title="Ver notas"
                  >
                    <span class="icon">
                      <i class="mdi mdi-note-text"></i>
                    </span>
                  </button>
                  <button 
                    class="button is-danger"
                    @click="confirmDelete(attendance.id)"
                    title="Eliminar registro"
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

    <div class="modal" :class="{ 'is-active': showAddModal }">
      <div class="modal-background" @click="closeAddModal"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">
            <span class="icon-text">
              <span class="icon">
                <i class="mdi mdi-account-plus"></i>
              </span>
              <span>Agregar Asistencia Manual</span>
            </span>
          </p>
          <button class="delete" @click="closeAddModal"></button>
        </header>
        <section class="modal-card-body">
          <div class="field">
            <label class="label">CI del Cooperativista *</label>
            <div class="control has-icons-left">
              <input 
                class="input is-medium" 
                type="text" 
                placeholder="Ingrese CI..." 
                v-model="manualForm.ci"
                list="cooperativistas-list"
                @input="onCiInput"
              >
              <span class="icon is-left">
                <i class="mdi mdi-card-account-details"></i>
              </span>
              <datalist id="cooperativistas-list">
                <option v-for="coop in cooperativistaStore.cooperativistas" :key="coop.id" :value="coop.ci">
                  {{ coop.ci }} - {{ coop.nombres }} {{ coop.apellidos }}
                </option>
              </datalist>
            </div>
          </div>

          <div v-if="selectedCooperativista" class="field">
            <label class="label">Cooperativista Seleccionado</label>
            <div class="box has-background-info-light">
              <div class="media">
                <div class="media-left">
                  <span class="icon is-large has-text-info">
                    <i class="mdi mdi-account-circle mdi-48px"></i>
                  </span>
                </div>
                <div class="media-content">
                  <p class="title is-6">{{ selectedCooperativista.nombres }} {{ selectedCooperativista.apellidos }}</p>
                  <p class="subtitle is-7"><strong>CI:</strong> {{ selectedCooperativista.ci }}</p>
                </div>
              </div>
            </div>
          </div>

          <div class="field">
            <label class="label">Hora de Registro *</label>
            <div class="control has-icons-left">
              <input 
                class="input is-medium" 
                type="time" 
                v-model="manualForm.hora"
              >
              <span class="icon is-left">
                <i class="mdi mdi-clock"></i>
              </span>
            </div>
          </div>

          <div class="field">
            <label class="label">Motivo del Registro Manual *</label>
            <div class="control has-icons-left">
              <div class="select is-fullwidth is-medium">
                <select v-model="manualForm.motivo">
                  <option value="">Seleccione un motivo...</option>
                  <option value="presente_sin_registro">Cooperativista en reunión pero no registró asistencia</option>
                  <option value="retraso_presente">Retraso pero estuvo en la reunión</option>
                  <option value="problema_dispositivo">Problemas con el dispositivo</option>
                  <option value="admin_ingresado">Ingresado por el administrador</option>
                  <option value="justificacion_especial">Justificación especial</option>
                  <option value="otro">Otro motivo...</option>
                </select>
              </div>
              <span class="icon is-left">
                <i class="mdi mdi-text-box"></i>
              </span>
            </div>
          </div>

          <div v-if="manualForm.motivo === 'otro'" class="field">
            <label class="label">Especifique el motivo *</label>
            <div class="control">
              <textarea 
                class="textarea" 
                placeholder="Describa el motivo..."
                v-model="manualForm.motivoOtro"
                rows="3"
              ></textarea>
            </div>
          </div>

          <div class="notification is-warning is-light">
            <p><strong>Nota:</strong> Los registros manuales se marcarán con coordenadas predeterminadas de Oruro.</p>
          </div>
        </section>
        <footer class="modal-card-foot">
          <button 
            class="button is-primary" 
            @click="saveManualAttendance"
            :disabled="!canSaveManual || attendanceStore.loading"
          >
            <span class="icon">
              <i class="mdi mdi-content-save"></i>
            </span>
            <span v-if="attendanceStore.loading">Guardando...</span>
            <span v-else>Guardar Asistencia</span>
          </button>
          <button class="button" @click="closeAddModal">Cancelar</button>
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
              <p class="mb-3">¿Está seguro de que desea eliminar este registro de asistencia?</p>
              <p><strong>Esta acción no se puede deshacer.</strong></p>
            </div>
          </article>
        </section>
        <footer class="modal-card-foot">
          <button 
            class="button is-danger" 
            @click="deleteAttendance"
            :disabled="attendanceStore.loading"
          >
            <span class="icon">
              <i class="mdi mdi-delete"></i>
            </span>
            <span v-if="attendanceStore.loading">Eliminando...</span>
            <span v-else>Eliminar</span>
          </button>
          <button class="button" @click="showDeleteModal = false">Cancelar</button>
        </footer>
      </div>
    </div>

    <div class="modal" :class="{ 'is-active': showNotesModal }">
      <div class="modal-background" @click="showNotesModal = false"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">
            <span class="icon-text">
              <span class="icon">
                <i class="mdi mdi-note-text"></i>
              </span>
              <span>Notas del Registro</span>
            </span>
          </p>
          <button class="delete" @click="showNotesModal = false"></button>
        </header>
        <section class="modal-card-body">
          <div class="content">
            <div class="box has-background-light">
              {{ currentNotes }}
            </div>
          </div>
        </section>
        <footer class="modal-card-foot">
          <button class="button" @click="showNotesModal = false">Cerrar</button>
        </footer>
      </div>
    </div>
  </div>
</template>

<script setup>
import dayjs from 'dayjs'
import 'dayjs/locale/es'
import { useAttendancePeriodStore } from '~/stores/attendancePeriod'
import { useAttendanceStore } from '~/stores/attendance'
import { useCooperativistasStore } from '~/stores/cooperativistas'

dayjs.locale('es')

definePageMeta({
  layout: 'dashboard',
  middleware: 'auth'
})

const router = useRouter()
const route = useRoute()
const periodStore = useAttendancePeriodStore()
const attendanceStore = useAttendanceStore()
const cooperativistaStore = useCooperativistasStore()

const periodId = parseInt(route.params.id)

const showAddModal = ref(false)
const showDeleteModal = ref(false)
const showNotesModal = ref(false)
const attendanceToDelete = ref(null)
const currentNotes = ref('')
const ciFilter = ref('')

const manualForm = ref({
  ci: '',
  hora: '',
  motivo: '',
  motivoOtro: ''
})

const selectedCooperativista = ref(null)

const filteredAttendances = computed(() => {
  if (!ciFilter.value) {
    return attendanceStore.attendances
  }
  
  return attendanceStore.attendances.filter(attendance => {
    const cooperativista = getCooperativistaById(attendance.cooperativista_id)
    return cooperativista?.ci?.includes(ciFilter.value) || false
  })
})

const canSaveManual = computed(() => {
  return selectedCooperativista.value && 
         manualForm.value.hora && 
         manualForm.value.motivo &&
         (manualForm.value.motivo !== 'otro' || manualForm.value.motivoOtro)
})

const formatDate = (dateString) => {
  const date = dayjs(`${dateString}T00:00:00`)
  return date.locale('es').format('dddd, D [de] MMMM [de] YYYY')
}

const getCooperativistaById = (id) => {
  return cooperativistaStore.cooperativistas.find(c => c.id === id)
}

const onCiInput = () => {
  const cooperativista = cooperativistaStore.cooperativistas.find(c => c.ci === manualForm.value.ci)
  selectedCooperativista.value = cooperativista || null
}

const loadData = async () => {
  try {
    await Promise.all([
      periodStore.fetchPeriod(periodId),
      attendanceStore.fetchAttendancesByPeriod(periodId),
      cooperativistaStore.cargarCooperativistas()
    ])
  } catch (error) {
    console.error('Error al cargar datos:', error)
  }
}

const clearErrors = () => {
  attendanceStore.error = null
  periodStore.error = null
  cooperativistaStore.error = null
}

const showNotes = (notes) => {
  currentNotes.value = notes
  showNotesModal.value = true
}

const confirmDelete = (attendanceId) => {
  attendanceToDelete.value = attendanceId
  showDeleteModal.value = true
}

const deleteAttendance = async () => {
  try {
    await attendanceStore.deleteAttendance(attendanceToDelete.value)
    showDeleteModal.value = false
    attendanceToDelete.value = null
    await loadData()
  } catch (error) {
    console.error('Error al eliminar asistencia:', error)
  }
}

const saveManualAttendance = async () => {
  if (!selectedCooperativista.value || !periodStore.selectedPeriod) return

  const notes = manualForm.value.motivo === 'otro' 
    ? manualForm.value.motivoOtro 
    : getMotivoDescription(manualForm.value.motivo)

  const attendanceData = {
    cooperativista_id: selectedCooperativista.value.id,
    period_id: periodId,
    device_id: 'ADMIN_MANUAL',
    fecha: periodStore.selectedPeriod.fecha_asistencia,
    hora: manualForm.value.hora + ':00',
    location_lat: -17.9833,
    location_lon: -67.1167,
    notes: `REGISTRO MANUAL - ${notes}`
  }

  try {
    await attendanceStore.createAttendance(attendanceData)
    closeAddModal()
    await loadData()
  } catch (error) {
    console.error('Error al guardar asistencia manual:', error)
  }
}

const getMotivoDescription = (motivo) => {
  const motivos = {
    'presente_sin_registro': 'Cooperativista en reunión pero no registró asistencia',
    'retraso_presente': 'Retraso pero estuvo en la reunión',
    'problema_dispositivo': 'Problemas con el dispositivo',
    'admin_ingresado': 'Ingresado por el administrador',
    'justificacion_especial': 'Justificación especial'
  }
  return motivos[motivo] || motivo
}

const closeAddModal = () => {
  showAddModal.value = false
  manualForm.value = {
    ci: '',
    hora: '',
    motivo: '',
    motivoOtro: ''
  }
  selectedCooperativista.value = null
}

onMounted(() => {
  loadData()
})

useHead({
  title: 'Detalle de Período de Asistencia'
})
</script>