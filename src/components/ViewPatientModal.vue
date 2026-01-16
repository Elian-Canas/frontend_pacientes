<template>
  <el-dialog
    title="Ver Paciente"
    :visible="visible"
    width="50%"
    @close="closeModal"
    append-to-body
  >
    <el-form :model="form" label-width="160px" ref="formRef">
      <!-- Tipo de Documento -->
      <el-form-item label="Tipo de Documento">
        <el-input
          v-model="form.tipo_documento_nombre"
          disabled
        />
      </el-form-item>

      <!-- Número de Documento -->
      <el-form-item label="Número de Documento">
        <el-input v-model="form.numero_documento" disabled />
      </el-form-item>

      <!-- Nombres -->
      <el-form-item label="Primer Nombre">
        <el-input v-model="form.primer_nombre" disabled />
      </el-form-item>
      <el-form-item label="Segundo Nombre">
        <el-input v-model="form.segundo_nombre" disabled />
      </el-form-item>

      <!-- Apellidos -->
      <el-form-item label="Primer Apellido">
        <el-input v-model="form.primer_apellido" disabled />
      </el-form-item>
      <el-form-item label="Segundo Apellido">
        <el-input v-model="form.segundo_apellido" disabled />
      </el-form-item>

      <!-- Género -->
      <el-form-item label="Género">
        <el-input v-model="form.genero_nombre" disabled />
      </el-form-item>

      <!-- Correo -->
      <el-form-item label="Correo Electrónico">
        <el-input v-model="form.correo" disabled />
      </el-form-item>

      <!-- Ubicación -->
      <el-form-item label="Departamento">
        <el-input v-model="form.departamento_nombre" disabled />
      </el-form-item>
      <el-form-item label="Municipio">
        <el-input v-model="form.municipio_nombre" disabled />
      </el-form-item>
    </el-form>

    <div slot="footer" class="dialog-footer">
      <el-button @click="closeModal">Cerrar</el-button>
      <el-button type="primary" @click="editPatient">Editar</el-button>
    </div>
  </el-dialog>
</template>

<script>
const API_BASE_URL = process.env.VUE_APP_API_BASE_URL;

export default {
  name: "ViewPatientModal",
  props: {
    visible: {
      type: Boolean,
      required: true,
    },
    patient: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      form: {
        tipo_documento_nombre: "",
        numero_documento: "",
        primer_nombre: "",
        segundo_nombre: "",
        primer_apellido: "",
        segundo_apellido: "",
        genero_nombre: "",
        correo: "",
        departamento_nombre: "",
        municipio_nombre: "",
      },
    };
  },
  watch: {
    patient: {
      immediate: true,
      handler(newPatient) {
        if (newPatient) {
          // Si ya vienen los nombres anidados (ej. desde la tabla), úsalos
          this.form = {
            tipo_documento_nombre: newPatient.tipo_documento?.nombre || newPatient.tipo_documento_nombre || "",
            numero_documento: newPatient.numero_documento || "",
            primer_nombre: newPatient.primer_nombre || "",
            segundo_nombre: newPatient.segundo_nombre || "",
            primer_apellido: newPatient.primer_apellido || "",
            segundo_apellido: newPatient.segundo_apellido || "",
            genero_nombre: newPatient.genero?.nombre || newPatient.genero_nombre || "",
            correo: newPatient.correo || "",
            departamento_nombre: newPatient.departamento?.nombre || newPatient.departamento_nombre || "",
            municipio_nombre: newPatient.municipio?.nombre || newPatient.municipio_nombre || "",
          };
        }
      },
    },
  },
  methods: {
    async fetchPatientDetails(id) {
      try {
        const res = await fetch(`${API_BASE_URL}/api/pacientes/${id}`, {
          headers: {
            'ngrok-skip-browser-warning': 'true'
          }
        });
        if (!res.ok) throw new Error("Error al cargar el paciente");
        const data = await res.json();

        // Asignar nombres de relaciones si no vienen en la tabla
        this.form = {
          tipo_documento_nombre: data.tipo_documento?.nombre || "",
          numero_documento: data.numero_documento || "",
          primer_nombre: data.primer_nombre || "",
          segundo_nombre: data.segundo_nombre || "",
          primer_apellido: data.primer_apellido || "",
          segundo_apellido: data.segundo_apellido || "",
          genero_nombre: data.genero?.nombre || "",
          correo: data.correo || "",
          departamento_nombre: data.departamento?.nombre || "",
          municipio_nombre: data.municipio?.nombre || "",
        };
      } catch (err) {
        console.error("Error al cargar paciente:", err);
        this.$message.error("Error al cargar los datos del paciente");
      }
    },
    editPatient() {
      if (!this.patient?.id) {
        console.error("No se puede editar: paciente sin ID");
        return;
      }
      this.$emit("edit", this.patient.id);
      this.closeModal();
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
.el-form-item .el-input {
  width: 100%;
}
</style>