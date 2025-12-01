<template>
  <div class="form-container">
    <form @submit.prevent="submitForm" novalidate>
      <!-- Información Básica -->
      <div class="section-title">Información Personal</div>

      <div class="columns is-multiline">
        <!-- Nombres: ocupa fila completa -->
        <div class="column is-12">
          <div class="field">
            <label class="label">Nombres *</label>
            <div class="control">
              <input
                v-model="formData.nombres"
                class="input"
                type="text"
                required
                placeholder="NOMBRE COMPLETO"
                aria-label="Nombres"
                maxlength="120"
              />
            </div>
          </div>
        </div>

        <!-- Dos inputs por fila a partir de aquí -->
        <div class="column is-half">
          <div class="field">
            <label class="label">Apellido Paterno *</label>
            <div class="control">
              <input
                v-model="formData.apellido_paterno"
                class="input"
                type="text"
                required
                placeholder="APELLIDO PATERNO"
                aria-label="Apellido paterno"
                maxlength="80"
              />
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Apellido Materno</label>
            <div class="control">
              <input
                v-model="formData.apellido_materno"
                class="input"
                type="text"
                placeholder="APELLIDO MATERNO"
                aria-label="Apellido materno"
                maxlength="80"
              />
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">CI</label>
            <div class="control">
              <input
                v-model="formData.ci"
                class="input"
                type="text"
                placeholder="CI"
                aria-label="Cédula de identidad"
                inputmode="numeric"
                maxlength="12"
              />
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Expedido</label>
            <div class="control">
              <div class="select is-fullwidth">
                <select v-model="formData.ci_expedido" aria-label="Departamento expedición">
                  <option value="">-</option>
                  <option value="LP">LP - LA PAZ</option>
                  <option value="OR">OR - ORURO</option>
                  <option value="CB">CB - COCHABAMBA</option>
                  <option value="SC">SC - SANTA CRUZ</option>
                  <option value="PT">PT - POTOSÍ</option>
                  <option value="TJ">TJ - TARIJA</option>
                  <option value="CH">CH - CHUQUISACA</option>
                  <option value="BN">BN - BENI</option>
                  <option value="PD">PD - PANDO</option>
                </select>
              </div>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Fecha Nacimiento</label>
            <div class="control">
              <input
                v-model="formData.fecha_nacimiento"
                class="input"
                type="date"
                aria-label="Fecha de nacimiento"
                :max="maxDate"
              />
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Email</label>
            <div class="control">
              <input
                v-model="formData.email"
                class="input"
                type="email"
                placeholder="CORREO@EJEMPLO.COM"
                aria-label="Correo electrónico"
              />
            </div>
          </div>
        </div>
      </div>

      <!-- Documentos -->
      <div class="section-title">Documentos</div>

      <div class="columns is-multiline">
        <div class="column is-half">
          <div class="field">
            <label class="label">Foto de CI</label>
            <div class="file-upload-zone" :class="{ 'has-preview': formData.ci_foto_url || ciPreviewUrl }">
              <div v-if="formData.ci_foto_url || ciPreviewUrl" class="preview-container">
                <img :src="ciPreviewUrl || formData.ci_foto_url" alt="CI" class="preview-image" />
                <button
                  type="button"
                  class="button is-danger is-small delete-btn"
                  @click="eliminarCIFoto"
                >
                  <i class="mdi mdi-delete"></i> Eliminar
                </button>
              </div>
              <div v-else class="upload-box">
                <input
                  ref="ciFileInput"
                  type="file"
                  accept="image/jpeg,image/jpg,image/png,image/webp"
                  @change="handleCIFileChange"
                  style="display: none"
                />
                <i class="mdi mdi-camera upload-icon"></i>
                <button
                  type="button"
                  class="button is-success"
                  @click="$refs.ciFileInput.click()"
                >
                  <i class="mdi mdi-upload"></i>
                  <span>Seleccionar Imagen</span>
                </button>
                <p class="help">JPG, PNG, WEBP (máx 10MB)</p>
              </div>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Documento ABC</label>
            <div class="file-upload-zone" :class="{ 'has-preview': formData.documento_abc_url || abcPreviewUrl }">
              <div v-if="formData.documento_abc_url || abcPreviewUrl" class="preview-container">
                <div class="document-preview">
                  <i class="mdi mdi-file-document-outline document-icon"></i>
                  <a :href="abcPreviewUrl || formData.documento_abc_url" target="_blank" class="document-link">
                    Ver Documento
                  </a>
                </div>
                <button
                  type="button"
                  class="button is-danger is-small delete-btn"
                  @click="eliminarDocumentoABC"
                >
                  <i class="mdi mdi-delete"></i> Eliminar
                </button>
              </div>
              <div v-else class="upload-box">
                <input
                  ref="abcFileInput"
                  type="file"
                  accept="application/pdf,image/jpeg,image/jpg,image/png"
                  @change="handleABCFileChange"
                  style="display: none"
                />
                <i class="mdi mdi-file-document upload-icon"></i>
                <button
                  type="button"
                  class="button is-success"
                  @click="$refs.abcFileInput.click()"
                >
                  <i class="mdi mdi-upload"></i>
                  <span>Seleccionar Archivo</span>
                </button>
                <p class="help">PDF o Imagen (máx 20MB)</p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Información Laboral -->
      <div class="section-title">Información Laboral</div>

      <div class="columns is-multiline">
        <div class="column is-half">
          <div class="field">
            <label class="label">Sección</label>
            <div class="control">
              <input
                v-model="formData.seccion"
                class="input"
                type="text"
                placeholder="NOMBRE DE SECCION"
                list="secciones-list"
                aria-label="Sección"
              />
              <datalist id="secciones-list">
                <option v-for="seccion in seccionesDisponibles" :key="seccion" :value="seccion" />
              </datalist>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Cuadrilla</label>
            <div class="control">
              <input
                v-model="formData.cuadrilla"
                class="input"
                type="text"
                placeholder="NOMBRE DE CUADRILLA"
                list="cuadrillas-list"
                aria-label="Cuadrilla"
              />
              <datalist id="cuadrillas-list">
                <option v-for="cuadrilla in cuadrillasDisponibles" :key="cuadrilla" :value="cuadrilla" />
              </datalist>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Jefe de Cuadrilla</label>
            <div class="control">
              <input
                v-model="formData.jefe_cuadrilla"
                class="input"
                type="text"
                placeholder="JEFE DE CUADRILLA"
                list="cargos-cuadrilla-list"
                aria-label="Jefe de cuadrilla"
              />
              <datalist id="cargos-cuadrilla-list">
                <option value="JEFE DE CUADRILLA" />
                <option value="SUB JEFE DE CUADRILLA" />
                <option value="TESORERO DE CUADRILLA" />
              </datalist>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Delegado de Sección</label>
            <div class="control">
              <label class="checkbox delegado-checkbox">
                <input type="checkbox" v-model="esDelegado" />
                Es Delegado de Sección
              </label>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Ocupación</label>
            <div class="control">
              <input
                v-model="formData.ocupacion"
                class="input"
                type="text"
                placeholder="OCUPACIÓN (EJ: PERFORISTA)"
                list="ocupaciones-list"
                aria-label="Ocupación"
              />
              <datalist id="ocupaciones-list">
                <option v-for="ocupacion in ocupacionesDisponibles" :key="ocupacion" :value="ocupacion" />
              </datalist>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Código Asegurado</label>
            <div class="control">
              <input
                v-model="formData.codigo_asegurado"
                class="input"
                type="text"
                placeholder="CÓDIGO ASEGURADO"
                aria-label="Código asegurado"
              />
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">CUA</label>
            <div class="control">
              <input
                v-model="formData.cua"
                class="input"
                type="text"
                placeholder="CUA"
                aria-label="CUA"
              />
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Estado Asegurado</label>
            <div class="control">
              <input
                v-model="formData.estado_asegurado"
                class="input"
                type="text"
                placeholder="ESTADO ASEGURADO (EJ: ACTIVO)"
                list="estados-asegurado-list"
                aria-label="Estado asegurado"
              />
              <datalist id="estados-asegurado-list">
                <option v-for="estado in estadosAseguradoDisponibles" :key="estado" :value="estado" />
              </datalist>
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Fecha Ingreso</label>
            <div class="control">
              <input
                v-model="formData.fecha_ingreso"
                class="input"
                type="date"
                aria-label="Fecha de ingreso"
                :max="today"
              />
            </div>
          </div>
        </div>

        <div class="column is-half">
          <div class="field">
            <label class="label">Estado</label>
            <div class="control">
              <label class="checkbox activo-checkbox">
                <input type="checkbox" v-model="formData.is_active" />
                Cooperativista Activo
              </label>
            </div>
          </div>
        </div>
      </div>

      <div v-if="!formData.is_active" class="columns">
        <div class="column is-8">
          <div class="field">
            <label class="label">Motivo de Baja</label>
            <div class="control">
              <textarea
                v-model="formData.motivo_baja"
                class="textarea"
                rows="2"
                placeholder="DESCRIBIR MOTIVO DE BAJA"
                aria-label="Motivo de baja"
              ></textarea>
            </div>
          </div>
        </div>

        <div class="column is-4">
          <div class="field">
            <label class="label">Fecha de Baja</label>
            <div class="control">
              <input
                v-model="formData.fecha_baja"
                class="input"
                type="date"
                aria-label="Fecha de baja"
              />
            </div>
          </div>
        </div>
      </div>

      <!-- Botones -->
      <div class="buttons-container">
        <button
          type="button"
          class="button is-light"
          @click="$emit('cancel')"
          :disabled="loading"
        >
          Cancelar
        </button>
        <button
          type="submit"
          class="button is-primary"
          :class="{ 'is-loading': loading }"
          :disabled="loading"
        >
          {{ isEdit ? 'ACTUALIZAR' : 'CREAR' }} COOPERATIVISTA
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'

const props = defineProps({
  cooperativista: { type: Object, default: null }
})

const emit = defineEmits(['submit', 'cancel', 'guardar'])

const authStore = useAuthStore()
const store = useCooperativistasStore()
const loading = ref(false)
const ciFileInput = ref(null)
const abcFileInput = ref(null)

const isEdit = computed(() => !!props.cooperativista)

const today = new Date().toISOString().split('T')[0]
const maxDate = today

const formData = ref({
  apellido_paterno: '',
  apellido_materno: '',
  nombres: '',
  ci: '',
  ci_expedido: '',
  ci_foto_url: '',
  documento_abc_url: '',
  fecha_nacimiento: '',
  email: '',
  seccion: '',
  cuadrilla: '',
  jefe_cuadrilla: '',
  delegado_seccion: '',
  ocupacion: '',
  codigo_asegurado: '',
  cua: '',
  estado_asegurado: '',
  fecha_ingreso: '',
  is_active: true,
  motivo_baja: '',
  fecha_baja: ''
})

// Checkbox para delegado
const esDelegado = ref(false)

// Previews y archivos temporales para cuando se está creando
const ciTempFile = ref(null)
const abcTempFile = ref(null)
const ciPreviewUrl = ref('')
const abcPreviewUrl = ref('')

// Cargar datos si es edición
if (props.cooperativista) {
  Object.assign(formData.value, props.cooperativista)
  esDelegado.value = !!props.cooperativista.delegado_seccion && props.cooperativista.delegado_seccion.toLowerCase().includes('delegado')
}

// --- Datos para datalist ---
const cuadrillasDisponibles = computed(() => store.cuadrillas || [])
const seccionesDisponibles = computed(() => store.secciones || [])
const ocupacionesDisponibles = computed(() => store.ocupaciones || [])
const estadosAseguradoDisponibles = computed(() => {
  const est = new Set((store.cooperativistas || []).filter(c => c.estado_asegurado).map(c => c.estado_asegurado))
  return Array.from(est).sort()
})

const handleCIFileChange = async (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  if (file.size > 10 * 1024 * 1024) {
    alert('La imagen es muy pesada (máximo 10MB)')
    return
  }

  // Si estamos editando, subir inmediatamente
  if (isEdit.value) {
    await subirArchivo(file, 'ci-foto')
    return
  }

  // Si es creación: guardar temporalmente para subir después
  ciTempFile.value = file
  try { 
    ciPreviewUrl.value = URL.createObjectURL(file) 
  } catch (e) { 
    ciPreviewUrl.value = '' 
  }
}

const handleABCFileChange = async (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  if (file.size > 20 * 1024 * 1024) {
    alert('El archivo es muy pesado (máximo 20MB)')
    return
  }
  
  if (isEdit.value) {
    await subirArchivo(file, 'documento-abc')
    return
  }
  
  abcTempFile.value = file
  try { 
    abcPreviewUrl.value = URL.createObjectURL(file) 
  } catch (e) { 
    abcPreviewUrl.value = '' 
  }
}

const subirArchivo = async (file, tipo) => {
  loading.value = true
  
  try {
    const formDataUpload = new FormData()
    formDataUpload.append('file', file)
    
    const response = await $fetch(
      `${authStore.apiUrl}/api/cooperativistas/${props.cooperativista.id}/${tipo}`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${authStore.token}`
        },
        body: formDataUpload
      }
    )
    
    if (tipo === 'ci-foto') {
      formData.value.ci_foto_url = response.url
    } else {
      formData.value.documento_abc_url = response.url
    }
    
    alert('Archivo subido exitosamente')
  } catch (error) {
    console.error('Error subiendo archivo:', error)
    alert('Error al subir archivo: ' + (error.data?.detail || error.message))
  } finally {
    loading.value = false
  }
}

const subirArchivosDespuesDeCrear = async (cooperativistaId) => {
  const uploads = []
  
  // Subir CI si existe
  if (ciTempFile.value) {
    const formDataCI = new FormData()
    formDataCI.append('file', ciTempFile.value)
    
    uploads.push(
      $fetch(`${authStore.apiUrl}/api/cooperativistas/${cooperativistaId}/ci-foto`, {
        method: 'POST',
        headers: { 'Authorization': `Bearer ${authStore.token}` },
        body: formDataCI
      }).catch(err => console.error('Error subiendo CI:', err))
    )
  }
  
  // Subir ABC si existe
  if (abcTempFile.value) {
    const formDataABC = new FormData()
    formDataABC.append('file', abcTempFile.value)
    
    uploads.push(
      $fetch(`${authStore.apiUrl}/api/cooperativistas/${cooperativistaId}/documento-abc`, {
        method: 'POST',
        headers: { 'Authorization': `Bearer ${authStore.token}` },
        body: formDataABC
      }).catch(err => console.error('Error subiendo ABC:', err))
    )
  }
  
  if (uploads.length > 0) {
    await Promise.all(uploads)
  }
}

const eliminarCIFoto = async () => {
  // Si hay preview temporal, eliminar localmente
  if (ciTempFile.value && !isEdit.value) {
    ciTempFile.value = null
    if (ciPreviewUrl.value) { 
      URL.revokeObjectURL(ciPreviewUrl.value)
      ciPreviewUrl.value = '' 
    }
    return
  }

  if (!isEdit.value) return
  if (!confirm('¿Eliminar la foto de CI?')) return
  
  loading.value = true
  
  try {
    await $fetch(
      `${authStore.apiUrl}/api/cooperativistas/${props.cooperativista.id}/ci-foto`,
      {
        method: 'DELETE',
        headers: { 'Authorization': `Bearer ${authStore.token}` }
      }
    )
    
    formData.value.ci_foto_url = ''
    alert('Foto eliminada')
  } catch (error) {
    console.error('Error eliminando foto:', error)
    alert('Error al eliminar foto')
  } finally {
    loading.value = false
  }
}

const eliminarDocumentoABC = async () => {
  if (abcTempFile.value && !isEdit.value) {
    abcTempFile.value = null
    if (abcPreviewUrl.value) { 
      URL.revokeObjectURL(abcPreviewUrl.value)
      abcPreviewUrl.value = '' 
    }
    return
  }

  if (!isEdit.value) return
  if (!confirm('¿Eliminar el documento ABC?')) return
  
  loading.value = true
  
  try {
    await $fetch(
      `${authStore.apiUrl}/api/cooperativistas/${props.cooperativista.id}/documento-abc`,
      {
        method: 'DELETE',
        headers: { 'Authorization': `Bearer ${authStore.token}` }
      }
    )
    
    formData.value.documento_abc_url = ''
    alert('Documento eliminado')
  } catch (error) {
    console.error('Error eliminando documento:', error)
    alert('Error al eliminar documento')
  } finally {
    loading.value = false
  }
}

const submitForm = async () => {
  loading.value = true
  
  try {
    const datos = { ...formData.value }
    
    // Manejar el campo delegado_seccion según el checkbox
    if (esDelegado.value) {
      datos.delegado_seccion = 'DELEGADO'
    } else {
      datos.delegado_seccion = ''
    }
    
    // No enviar URLs de archivos en el payload principal
    delete datos.ci_foto_url
    delete datos.documento_abc_url
    
    // Limpiar valores vacíos
    Object.keys(datos).forEach(key => {
      if (datos[key] === '' || datos[key] === null) {
        delete datos[key]
      }
    })
    
    let cooperativistaCreado
    
    if (isEdit.value) {
      // ACTUALIZAR
      await store.actualizarCooperativista(props.cooperativista.id, datos)
      alert('Cooperativista actualizado exitosamente')
    } else {
      // CREAR
      cooperativistaCreado = await store.crearCooperativista(datos)
      
      // Subir archivos si existen
      if (cooperativistaCreado?.id && (ciTempFile.value || abcTempFile.value)) {
        await subirArchivosDespuesDeCrear(cooperativistaCreado.id)
      }
      
      alert('Cooperativista creado exitosamente')
    }
    
    emit('guardar')
    
  } catch (error) {
    console.error('Error al guardar:', error)
    alert('Error al guardar: ' + (error.data?.detail || error.message))
  } finally {
    loading.value = false
  }
}

// Limpiar URLs al desmontar
onUnmounted(() => {
  if (ciPreviewUrl.value) URL.revokeObjectURL(ciPreviewUrl.value)
  if (abcPreviewUrl.value) URL.revokeObjectURL(abcPreviewUrl.value)
})
</script>

<style scoped>
.form-container {
  background: linear-gradient(to bottom, #0a1a0a 0%, #0f1f0f 50%, #0a1a0a 100%);
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.section-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: #038730;
  margin: 1.5rem 0 1rem 0;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #feea01;
}

.file-upload-zone {
  border: 2px dashed #ddd;
  background: #fafafa;
  border-radius: 8px;
  padding: 1.5rem;
  min-height: 180px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.file-upload-zone.has-preview {
  border-color: #038730;
  background: #f0f9f4;
}

.upload-box {
  text-align: center;
  width: 100%;
}

.upload-icon {
  font-size: 3rem;
  color: #038730;
  margin-bottom: 1rem;
  display: block;
}

.preview-container {
  width: 100%;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.preview-image {
  max-width: 100%;
  max-height: 250px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  border: 2px solid #038730;
}

.document-preview {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.document-icon {
  font-size: 4rem;
  color: #038730;
}

.document-link {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1rem;
  font-weight: 600;
  color: #038730;
  text-decoration: underline;
  transition: color 0.3s ease;
}

.document-link:hover {
  color: #026d27;
}

.delete-btn {
  margin-top: 0.5rem;
}

.delegado-checkbox,
.activo-checkbox {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  background: #f0f9f4;
  border-radius: 6px;
  border: 2px solid #038730;
  font-weight: 600;
  color: #038730;
  cursor: pointer;
  transition: all 0.3s ease;
}

.delegado-checkbox:hover,
.activo-checkbox:hover {
  background: #e0f2e9;
  transform: translateY(-1px);
}

.delegado-checkbox input[type="checkbox"],
.activo-checkbox input[type="checkbox"] {
  margin-right: 0.75rem;
  width: 18px;
  height: 18px;
  cursor: pointer;
}

.buttons-container {
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #ddd;
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
}

.help {
  margin-top: 0.75rem;
  font-size: 0.85rem;
  color: #666;
}

@media (max-width: 768px) {
  .form-container {
    padding: 1rem;
  }
  
  .file-upload-zone {
    min-height: 150px;
    padding: 1rem;
  }
  
  .preview-image {
    max-height: 180px;
  }
}
</style>