<template>
  <el-dialog
    title="Crear Paciente"
    :visible="visible"
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
      <el-form-item label="Tipo de Documento" prop="tipo_documento_id">
        <el-select
          v-model="form.tipo_documento_id"
          placeholder="Seleccione"
          filterable
        >
          <el-option
            v-for="tipo in tiposDocumento"
            :key="tipo.id"
            :label="tipo.nombre"
            :value="tipo.id"
          />
        </el-select>
      </el-form-item>

      <!-- Número de Documento -->
      <el-form-item label="Número de Documento" prop="numero_documento">
        <el-input
          v-model="form.numero_documento"
          maxlength="20"
          autocomplete="off"
        />
      </el-form-item>

      <!-- Nombres -->
      <el-form-item label="Primer Nombre" prop="nombre1">
        <el-input v-model="form.nombre1" maxlength="50" autocomplete="off" />
      </el-form-item>
      <el-form-item label="Segundo Nombre">
        <el-input v-model="form.nombre2" maxlength="50" autocomplete="off" />
      </el-form-item>

      <!-- Apellidos -->
      <el-form-item label="Primer Apellido" prop="apellido1">
        <el-input v-model="form.apellido1" maxlength="50" autocomplete="off" />
      </el-form-item>
      <el-form-item label="Segundo Apellido">
        <el-input v-model="form.apellido2" maxlength="50" autocomplete="off" />
      </el-form-item>

      <!-- Género -->
      <el-form-item label="Género" prop="genero_id">
        <el-select v-model="form.genero_id" placeholder="Seleccione">
          <el-option
            v-for="genero in generos"
            :key="genero.id"
            :label="genero.nombre"
            :value="genero.id"
          />
        </el-select>
      </el-form-item>

      <!-- Correo -->
      <el-form-item label="Correo Electrónico" prop="correo">
        <el-input
          v-model="form.correo"
          type="email"
          maxlength="100"
          autocomplete="off"
        />
      </el-form-item>

      <!-- Departamento y Municipio -->
      <el-form-item label="Departamento" prop="departamento_id">
        <el-select
          v-model="form.departamento_id"
          placeholder="Seleccione"
          filterable
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
        <el-select v-model="form.municipio_id" placeholder="Seleccione" filterable>
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
      <el-button type="primary" @click="submitForm" :loading="saving">Crear</el-button>
    </div>
  </el-dialog>
</template>

<script>
const API_BASE_URL = process.env.VUE_APP_API_BASE_URL;

export default {
  name: "CreatePatientModal",
  props: {
    visible: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
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
        municipio_id: null,
      },
      loading: false,
      saving: false,
      // Catálogos
      tiposDocumento: [],
      generos: [],
      departamentos: [],
      municipios: [],

      rules: {
        tipo_documento_id: [
          { required: true, message: "Requerido", trigger: "change" },
        ],
        numero_documento: [
          { required: true, message: "Requerido", trigger: "blur" },
          { max: 20, message: "Máximo 20 caracteres", trigger: "blur" },
        ],
        nombre1: [
          { required: true, message: "Requerido", trigger: "blur" },
          { max: 50, message: "Máximo 50 caracteres", trigger: "blur" },
        ],
        apellido1: [
          { required: true, message: "Requerido", trigger: "blur" },
          { max: 50, message: "Máximo 50 caracteres", trigger: "blur" },
        ],
        genero_id: [{ required: true, message: "Requerido", trigger: "change" }],
        correo: [
          { required: true, message: "Requerido", trigger: "blur" },
          { type: "email", message: "Formato de correo inválido", trigger: "blur" },
        ],
        departamento_id: [
          { required: true, message: "Requerido", trigger: "change" },
        ],
        municipio_id: [{ required: true, message: "Requerido", trigger: "change" }],
      },
    };
  },
  async mounted() {
    await this.loadCatalogs();
  },
  methods: {
    async loadCatalogs() {
      this.loading = true;
      try {
        const headers = {
          'ngrok-skip-browser-warning': 'true'
        };

        const [tiposRes, generosRes, deptosRes, muniRes] = await Promise.all([
          fetch(`${API_BASE_URL}/api/tipo-documentos`, { headers }),
          fetch(`${API_BASE_URL}/api/generos`, { headers }),
          fetch(`${API_BASE_URL}/api/departamentos`, { headers }),
          fetch(`${API_BASE_URL}/api/municipios`, { headers }),
        ]);

        if (tiposRes.ok) this.tiposDocumento = await tiposRes.json();
        if (generosRes.ok) this.generos = await generosRes.json();
        if (deptosRes.ok) this.departamentos = await deptosRes.json();
        if (muniRes.ok) this.municipios = await muniRes.json();
      } catch (err) {
        console.error("Error al cargar catálogos:", err);
        this.$message.error("Error al cargar opciones");
      } finally {
        this.loading = false;
      }
    },
    async submitForm() {
      await this.$refs.formRef.validate(async (valid) => {
        if (!valid) return;

        this.saving = true;
        try {
          const payload = {
            tipo_documento_id: this.form.tipo_documento_id,
            numero_documento: this.form.numero_documento,
            primer_nombre: this.form.nombre1,
            segundo_nombre: this.form.nombre2 || null, // opcional
            primer_apellido: this.form.apellido1,
            segundo_apellido: this.form.apellido2 || null, // opcional
            genero_id: this.form.genero_id,
            departamento_id: this.form.departamento_id,
            municipio_id: this.form.municipio_id,
            correo: this.form.correo,
          };

          const response = await fetch(`${API_BASE_URL}/api/pacientes`, {
            method: "POST",
            headers: { 
              "Content-Type": "application/json",
              "ngrok-skip-browser-warning": "true",
              "Authorization": `Bearer ${localStorage.getItem('access_token')}`
            },
            body: JSON.stringify(payload),
          });

          if (!response.ok) {
            const errorText = await response.text();
            throw new Error(errorText || "Error al crear el paciente");
          }

          this.$message.success("Paciente creado exitosamente");
          this.$emit("created");
          this.closeModal();
        } catch (err) {
          console.error("Error al crear paciente:", err);
          this.$message.error(err.message || "Error al crear el paciente");
        } finally {
          this.saving = false;
        }
      });
    },
    closeModal() {
      this.$emit("close");
    },
  },
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