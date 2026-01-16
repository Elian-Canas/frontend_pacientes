<template>
  <el-dialog
    title="Ver Paciente"
    :visible="visible"
    width="50%"
    @close="closeModal"
    append-to-body
  >
    <el-form :model="form" label-width="160px">
      <!-- Tipo de Documento -->
      <el-form-item label="Tipo de Documento">
        <el-input v-model="form.tipo_documento_nombre" disabled />
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
          this.form = {
            tipo_documento_nombre: newPatient.tipo_documento?.nombre || '—',
            numero_documento: newPatient.numero_documento || '—',
            primer_nombre: newPatient.primer_nombre || '—',
            segundo_nombre: newPatient.segundo_nombre || '',
            primer_apellido: newPatient.primer_apellido || '—',
            segundo_apellido: newPatient.segundo_apellido || '',
            genero_nombre: newPatient.genero?.nombre || '—',
            correo: newPatient.correo || '—',
            departamento_nombre: newPatient.departamento?.nombre || '—',
            municipio_nombre: newPatient.municipio?.nombre || '—',
          };
        } else {
          this.form = {
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
          };
        }
      },
    },
  },
  methods: {
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