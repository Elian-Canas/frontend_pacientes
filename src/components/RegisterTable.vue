<template>
  <div class="patients-container">
    <h1>Pacientes</h1>

    <!-- Contenedor principal de filtros y acciones -->
    <div class="filters-actions-container">
      <!-- Filtros -->
      <div class="filters-bar">
        <div class="block">
          <el-select
            v-model="filterField"
            placeholder="Seleccione un Filtro"
            @change="handleChange"
            class="filter-select"
          >
            <el-option
              v-for="option in filterOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </div>

        <el-input
          v-if="filterField && !isDateFilter"
          type="text"
          placeholder="Ingrese el valor a buscar..."
          v-model="searchQuery"
          size="small"
          class="filter-input"
        />

        <el-button @click="search" size="small" class="action-btn">
          Buscar
        </el-button>
        <el-button @click="clearFilters" size="small" type="info" class="action-btn">
          Limpiar
        </el-button>
        <el-button
          @click="exportToExcel"
          size="small"
          icon="el-icon-document"
          type="success"
          class="action-btn"
        >
          Generar Excel
        </el-button>
      </div>

      <!-- Botones de acción: Logout + Crear Paciente -->
      <div class="action-buttons">
        <el-button type="primary" icon="el-icon-plus" @click="openCreatePatientModal">
          Crear Paciente
        </el-button>

        <el-button
          type="danger"
          icon="el-icon-switch-button"
          @click="logout"
          size="small"
          class="logout-btn"
        >
          Cerrar Sesión
        </el-button>

      </div>
    </div>

    <!-- Tabla de pacientes -->
    <el-table :data="patients" style="width: 100%; margin-bottom: 24px">
      <el-table-column prop="numero_documento" label="Documento" min-width="120" />
      <el-table-column prop="primer_nombre" label="Primer Nombre" min-width="120" />
      <el-table-column prop="segundo_nombre" label="Segundo Nombre" min-width="120" />
      <el-table-column prop="primer_apellido" label="Primer Apellido" min-width="120" />
      <el-table-column prop="segundo_apellido" label="Segundo Apellido" min-width="120" />
      <el-table-column prop="correo" label="Correo Electrónico" min-width="150" />
      <el-table-column label="Acciones" min-width="100">
        <template #default="scope">
          <el-button
            icon="el-icon-view"
            size="mini"
            @click="viewPatient(scope.row)"
            title="Ver paciente"
          ></el-button>
          <el-button
            icon="el-icon-edit"
            size="mini"
            type="primary"
            @click="editPatient(scope.row)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- Paginación -->
    <div class="pagination-container">
      <el-pagination
        background
        layout="prev, pager, next"
        :total="totalCount"
        :page-size="limit"
        :current-page="currentPage"
        @current-change="handlePageChange"
        :hide-on-single-page="true"
      />
    </div>

    <!-- Modales -->
    <create-patient-modal
      :visible="isCreatePatientModalVisible"
      :mode.sync="modalMode"
      :patient="selectedPatient"
      @close="isCreatePatientModalVisible = false"
      @created="handlePatientCreated"
    />
    <view-patient-modal
      v-if="isViewPatientModalVisible"
      :visible.sync="isViewPatientModalVisible"
      :patient="selectedPatient"
      @close="isViewPatientModalVisible = false"
      ref="viewPatientModal"
      @edit="openEditModal"
    />
    <edit-patient-modal
      v-if="isEditPatientModalVisible"
      ref="editPatientModal"
      :visible.sync="isEditPatientModalVisible"
      :patient-id="selectedPatientId"
    />
  </div>
</template>

<script>
import CreatePatientModal from "./CreatePatientModal.vue";
import ViewPatientModal from "./ViewPatientModal.vue";
import EditPatientModal from "./EditPatientModal.vue";

const API_BASE_URL = process.env.VUE_APP_API_BASE_URL;

export default {
  name: "PatientsTable",
  components: {
    CreatePatientModal,
    ViewPatientModal,
    EditPatientModal,
  },
  computed: {
    isDateFilter() {
      return this.filterField === "date";
    },
  },
  data() {
    return {
      patients: [],
      totalCount: 0,
      limit: 15,
      currentPage: 1,
      filterField: "",
      searchQuery: "",
      filterOptions: [
        { value: "numero_documento", label: "Número de Documento" },
        { value: "nombre1", label: "Primer Nombre" },
        { value: "apellido1", label: "Primer Apellido" },
        { value: "correo", label: "Correo Electrónico" },
      ],
      isCreatePatientModalVisible: false,
      isViewPatientModalVisible: false,
      isEditPatientModalVisible: false,
      selectedPatientId: null,
      modalMode: "create",
      selectedPatient: null,
      exporting: false,
    };
  },
  methods: {
    async fetchPatients(page = 1) {
      this.loading = true;
      try {
        const token = localStorage.getItem('access_token');
        if (!token) {
          this.$message?.warning("Sesión expirada. Por favor inicia sesión nuevamente.");
          this.patients = [];
          this.totalCount = 0;
          return;
        }

        const url = `${API_BASE_URL}/api/pacientes?page=${page}&limit=${this.limit}`;
        const res = await fetch(url, {
          headers: {
            'Authorization': `Bearer ${token}`,
            'ngrok-skip-browser-warning': 'true' // 👈 solo si usas ngrok
          }
        });

        if (!res.ok) {
          if (res.status === 401) {
            this.$message?.error("Sesión no válida. Por favor inicia sesión nuevamente.");
            localStorage.removeItem('access_token');
            localStorage.removeItem('token_type');
            window.location.reload();
            return;
          }
          throw new Error("Error al obtener pacientes");
        }

        const data = await res.json();
        this.patients = data.data || [];
        this.totalCount = data.meta?.total || 0;
        this.currentPage = data.meta?.current_page || 1;
      } catch (err) {
        console.error("Error al obtener pacientes:", err);
        this.$message?.error("Error al cargar los pacientes");
        this.patients = [];
        this.totalCount = 0;
      } finally {
        this.loading = false;
      }
    },
    handlePageChange(page) {
      this.currentPage = page;
      this.fetchPatients(page);
    },
    async search() {
      if (!this.filterField) {
        this.$message?.warning("Por favor seleccione un filtro");
        return;
      }
      if (!this.isDateFilter && (!this.searchQuery?.trim())) {
        this.$message?.warning("Por favor ingrese un valor para buscar");
        return;
      }

      this.loading = true;
      try {
        const params = new URLSearchParams({
          page: 1,
          limit: this.limit,
        });
        if (!this.isDateFilter) {
          params.append(this.filterField, this.searchQuery.trim());
        }

        const url = `${API_BASE_URL}/pacientes?${params.toString()}`;
        const res = await fetch(url);
        if (!res.ok) throw new Error("Error al filtrar pacientes");

        const data = await res.json();
        this.patients = data.data || [];
        this.totalCount = data.meta?.total || 0;
        this.currentPage = data.meta?.current_page || 1;
      } catch (err) {
        console.error("Error al filtrar pacientes:", err);
        this.$message?.error("Error al aplicar el filtro");
        this.patients = [];
        this.totalCount = 0;
      } finally {
        this.loading = false;
      }
    },
    handleChange() {
      this.searchQuery = "";
    },
    clearFilters() {
      this.filterField = "";
      this.searchQuery = "";
      this.currentPage = 1;
      this.fetchPatients(1);
    },
    openCreatePatientModal() {
      this.isCreatePatientModalVisible = true;
      this.modalMode = "create";
      this.selectedPatient = null;
    },
    handlePatientCreated() {
      this.fetchPatients(this.currentPage);
    },
    viewPatient(patient) {
      this.selectedPatient = patient;
      this.isViewPatientModalVisible = true;
      this.$nextTick(() => {
        if (this.$refs.viewPatientModal) {
          this.$refs.viewPatientModal.fetchPatientDetails(patient.id);
        }
      });
    },
    editPatient(patient) {
      this.selectedPatientId = patient.id;
      this.isEditPatientModalVisible = true;
      this.$nextTick(() => {
        if (this.$refs.editPatientModal) {
          this.$refs.editPatientModal.fetchPatientDetails(patient.id);
        }
      });
    },
    openEditModal(patientId) {
      if (!patientId) {
        console.error("ID de paciente no proporcionado");
        return;
      }
      this.selectedPatientId = patientId;
      this.isViewPatientModalVisible = false;
      this.isEditPatientModalVisible = true;
    },
    exportToExcel() {
      this.exporting = true;
      const params = new URLSearchParams();
      if (this.filterField && !this.isDateFilter && this.searchQuery?.trim()) {
        params.append(this.filterField, this.searchQuery.trim());
      }
      const url = `${API_BASE_URL}/pacientes/export?${params.toString()}`;
      window.open(url, "_blank");
      setTimeout(() => {
        this.exporting = false;
      }, 800);
    },
    async logout() {
      try {
        const token = localStorage.getItem('access_token');
        if (!token) {
          this.$message.warning('No hay sesión activa');
          return;
        }

        const response = await fetch(`${API_BASE_URL}/api/auth/logout`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
          }
        });

        if (!response.ok) {
          throw new Error('Error al cerrar sesión');
        }

        localStorage.removeItem('access_token');
        localStorage.removeItem('token_type');

        // Volver al login
        window.location.reload();
      } catch (err) {
        console.error('Error al cerrar sesión:', err);
        this.$message.error('No se pudo cerrar sesión. Inténtalo nuevamente.');
      }
    }
  },
  mounted() {
    this.fetchPatients(1);
  },
};
</script>

<style lang="scss" scoped>
.patients-container {
  padding: 20px;
}

.filters-actions-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  margin-bottom: 16px;

  @media (max-width: 768px) {
    flex-direction: column;
    align-items: stretch;
  }
}

.filters-bar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  flex: 1;
  min-width: 300px;

  @media (max-width: 768px) {
    width: 100%;
  }
}

.filter-input {
  min-width: 140px;
  max-width: 200px;
  flex: 1 1 140px;

  @media (max-width: 600px) {
    min-width: 100%;
    max-width: none;
  }
}

.filter-select {
  min-width: 180px;

  @media (max-width: 600px) {
    min-width: 100%;
  }
}

.action-btn {
  white-space: nowrap;
  margin-right: 8px;
}

.action-buttons {
  display: flex;
  gap: 8px;
  align-items: center;

  @media (max-width: 768px) {
    width: 100%;
    justify-content: flex-end;
  }

  @media (max-width: 480px) {
    flex-direction: column;
    width: 100%;
    align-items: stretch;
  }
}

.logout-btn {
  @media (max-width: 480px) {
    text-align: center;
  }
}

.pagination-container {
  display: flex;
  justify-content: flex-end;
  padding: 16px 0;

  @media (max-width: 768px) {
    justify-content: center;
  }
}

/* Ajustes para la tabla en móviles */
.el-table {
  font-size: 14px;

  .el-table__header-wrapper th,
  .el-table__body-wrapper td {
    word-break: break-word;
    white-space: normal;
    padding: 8px 12px;

    @media (max-width: 768px) {
      font-size: 12px;
      padding: 6px 8px;
    }
  }
}
</style>