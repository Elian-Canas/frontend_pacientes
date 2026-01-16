<template>
  <div>
    <el-dialog
      title="Editar Paciente"
      :visible.sync="localVisible"
      width="50%"
      @close="closeModal"
      append-to-body
    >
      <el-form
        :model="form"
        :rules="rules"
        ref="formRef"
        label-width="160px"
        v-loading="loading"
      >
        <!-- Tipo de Documento -->
        <el-form-item label="Tipo de documento" prop="tipo_documento_id" v-if="tiposDocumentos.length > 0">
          <el-select
            v-model="form.tipo_documento_id"
            placeholder="Seleccione"
            filterable
          >
            <el-option
              v-for="tipo in tiposDocumentos"
              :key="tipo.id"
              :label="tipo.nombre"
              :value="tipo.id"
            />
          </el-select>
        </el-form-item>

        <!-- Número de Documento -->
        <el-form-item label="Número de Documento" prop="numero_documento">
          <el-input v-model="form.numero_documento" maxlength="20" />
        </el-form-item>

        <!-- Nombres -->
        <el-form-item label="Primer Nombre" prop="nombre1">
          <el-input v-model="form.nombre1" maxlength="50" />
        </el-form-item>
        <el-form-item label="Segundo Nombre">
          <el-input v-model="form.nombre2" maxlength="50" />
        </el-form-item>

        <!-- Apellidos -->
        <el-form-item label="Primer Apellido" prop="apellido1">
          <el-input v-model="form.apellido1" maxlength="50" />
        </el-form-item>
        <el-form-item label="Segundo Apellido">
          <el-input v-model="form.apellido2" maxlength="50" />
        </el-form-item>

        <!-- Género -->
        <el-form-item label="Genero" prop="genero_id" v-if="generos.length > 0">
          <el-select
            v-model="form.genero_id"
            placeholder="Seleccione"
            filterable
          >
            <el-option
              v-for="gen in generos"
              :key="gen.id"
              :label="gen.nombre"
              :value="gen.id"
            />
          </el-select>
        </el-form-item>

        <!-- Correo -->
        <el-form-item label="Correo Electrónico" prop="correo">
          <el-input v-model="form.correo" type="email" maxlength="100" />
        </el-form-item>

        <!-- Departamento y Municipio -->
        <el-form-item label="Departamento" prop="departamento_id">
          <el-select
            v-model="form.departamento_id"
            placeholder="Seleccione"
            filterable
            v-if="departamentos.length > 0"
          >
            <el-option
              v-for="depto in departamentos"
              :key="depto.id"
              :label="depto.nombre"
              :value="depto.id"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="Municipio" prop="municipio_id" v-if="municipios.length > 0">
          <el-select
            v-model="form.municipio_id"
            placeholder="Seleccione"
            filterable
          >
            <el-option
              v-for="mun in municipios"
              :key="mun.id"
              :label="mun.nombre"
              :value="mun.id"
            />
          </el-select>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="closeModal">Cancelar</el-button>
        <el-button type="primary" @click="submitForm" :loading="saving">Actualizar</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
const API_BASE_URL = process.env.VUE_APP_API_BASE_URL;

export default {
  name: "EditPatientModal",
  props: {
    visible: { type: Boolean, required: true },
    patientId: { type: [Number, String], required: true }
  },
  data() {
    return {
      localVisible: this.visible,
      loading: false,
      saving: false,
      form: {
        tipo_documento_id: null,
        numero_documento: "",
        nombre1: "",
        nombre2: "",
        apellido1: "",
        apellido2: "",
        genero_id: null,
        correo: "",
        departamento_id: null,
        municipio_id: null
      },
      // Catálogos
      tiposDocumento: [],
      generos: [],
      departamentos: [],
      municipios: [],

      rules: {
        tipo_documento_id: [{ required: true, message: "Requerido", trigger: "change" }],
        numero_documento: [{ required: true, message: "Requerido", trigger: "blur" }],
        nombre1: [{ required: true, message: "Requerido", trigger: "blur" }],
        apellido1: [{ required: true, message: "Requerido", trigger: "blur" }],
        genero_id: [{ required: true, message: "Requerido", trigger: "change" }],
        correo: [
          { required: true, message: "Requerido", trigger: "blur" },
          { type: "email", message: "Formato de correo inválido", trigger: "blur" }
        ],
        departamento_id: [{ required: true, message: "Requerido", trigger: "change" }],
        municipio_id: [{ required: true, message: "Requerido", trigger: "change" }]
      }
    };
  },
  watch: {
    visible: {
      handler(val) {
        this.localVisible = val;
        if (val && this.patientId) {
          this.loadCatalogs();
          this.fetchPatientDetails(this.patientId);
        }
      },
      immediate: true
    },
    patientId: {
      handler(newId) {
        if (newId && this.localVisible) {
          this.fetchPatientDetails(newId);
        }
      }
    }
  },
  methods: {
    async loadCatalogs() {
      try {
        const headers = { 'ngrok-skip-browser-warning': 'true' };
        
        // Cargar tipos de documento
        const tiposRes = await fetch(`${API_BASE_URL}/api/tipo-documentos`, { headers });
        if (tiposRes.ok) {
          this.tiposDocumento = await tiposRes.json();
        }

        // Cargar géneros
        const generosRes = await fetch(`${API_BASE_URL}/api/generos`, { headers });
        if (generosRes.ok) {
          this.generos = await generosRes.json();
        }

        // Cargar departamentos
        const deptosRes = await fetch(`${API_BASE_URL}/api/departamentos`, { headers });
        if (deptosRes.ok) {
          this.departamentos = await deptosRes.json();
        }

        // Esperar a que Vue procese los cambios
        await this.$nextTick();
        
      } catch (err) {
        console.error("Error al cargar catálogos:", err);
        this.$message.error("Error al cargar opciones");
      }
    },
    async fetchPatientDetails(id) {
      this.loading = true;
      try {
        const token = localStorage.getItem('access_token');
        const headers = {
          'ngrok-skip-browser-warning': 'true',
          ...(token ? { 'Authorization': `Bearer ${token}` } : {})
        };

        const res = await fetch(`${API_BASE_URL}/api/pacientes/${id}`, { headers });
        if (!res.ok) throw new Error("Error al cargar el paciente");
        const data = await res.json();

        // ✅ Cargar municipios ANTES de asignar el form
        if (data.departamento_id) {
          const munRes = await fetch(`${API_BASE_URL}/api/municipios`, {
            headers: { 'ngrok-skip-browser-warning': 'true' }
          });
          if (munRes.ok) {
            this.municipios = await munRes.json();
          }
        }

        if (data.departamento_id) {
          const depaRes = await fetch(`${API_BASE_URL}/api/departamentos`, {
            headers: { 'ngrok-skip-browser-warning': 'true' }
          });
          if (depaRes.ok) {
            this.departamentos = await depaRes.json();
          }
        }

        const genRes = await fetch(`${API_BASE_URL}/api/generos`, {
          headers: { 'ngrok-skip-browser-warning': 'true' }
          });
        if (genRes.ok) {
          this.generos = await genRes.json();
        }

        const tiposRes = await fetch(`${API_BASE_URL}/api/tipo-documentos`, {
          headers: { 'ngrok-skip-browser-warning': 'true' }
          });
        if (tiposRes.ok) {
          this.tiposDocumentos = await tiposRes.json();
        }

        // ✅ Asignar los valores usando $set para asegurar reactividad
        await this.$nextTick();
        
        this.$set(this.form, 'tipo_documento_id', Number(data.tipo_documento_id));
        this.$set(this.form, 'numero_documento', data.numero_documento || "");
        this.$set(this.form, 'nombre1', data.primer_nombre || "");
        this.$set(this.form, 'nombre2', data.segundo_nombre || "");
        this.$set(this.form, 'apellido1', data.primer_apellido || "");
        this.$set(this.form, 'apellido2', data.segundo_apellido || "");
        this.$set(this.form, 'genero_id', Number(data.genero_id));
        this.$set(this.form, 'correo', data.correo || "");
        this.$set(this.form, 'departamento_id', Number(data.departamento_id));
        this.$set(this.form, 'municipio_id', Number(data.municipio_id));
        this.$set(this.form, 'departamento', (data.departamento));
        this.$set(this.form, 'genero', (data.genero));
        this.$set(this.form, 'tipo_documento', (data.tipo_documento));

        // ✅ Esperar a que Vue actualice el DOM
        await this.$nextTick();

      } catch (err) {
        console.error("Error al cargar paciente:", err);
        this.$message.error("Error al cargar los datos del paciente");
      } finally {
        this.loading = false;
      }
    },

    async submitForm() {
      await this.$refs.formRef.validate(async (valid) => {
        if (!valid) return;

        this.saving = true;
        try {
          const token = localStorage.getItem('access_token');
          if (!token) {
            this.$message.error("Sesión expirada. Por favor inicia sesión nuevamente.");
            return;
          }

          const payload = {
            id: this.patientId,
            tipo_documento_id: this.form.tipo_documento_id,
            numero_documento: this.form.numero_documento,
            primer_nombre: this.form.nombre1,
            segundo_nombre: this.form.nombre2 || null,
            primer_apellido: this.form.apellido1,
            segundo_apellido: this.form.apellido2 || null,
            genero_id: this.form.genero_id,
            departamento_id: this.form.departamento_id,
            municipio_id: this.form.municipio_id,
            correo: this.form.correo
          };

          const response = await fetch(`${API_BASE_URL}/api/pacientes/${this.patientId}`, {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
              "ngrok-skip-browser-warning": "true",
              "Authorization": `Bearer ${token}`
            },
            body: JSON.stringify(payload)
          });

          if (!response.ok) {
            const errorData = await response.json().catch(() => ({}));
            throw new Error(errorData.message || "Error al actualizar el paciente");
          }

          this.$message.success("Paciente actualizado correctamente");
          this.$emit("saved");
          this.closeModal();
        } catch (err) {
          console.error("Error al guardar:", err);
          this.$message.error(err.message || "Error al guardar los cambios");
        } finally {
          this.saving = false;
        }
      });
    },

    closeModal() {
      this.localVisible = false;
      this.$emit("update:visible", false);
      this.$emit("close");
    }
  }
};
</script>

<style scoped>
.dialog-footer {
  text-align: right;
}
.el-form-item .el-input,
.el-form-item .el-select {
  width: 100%;
}
</style>