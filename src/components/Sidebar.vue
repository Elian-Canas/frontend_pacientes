<template>
  <el-container class="sidebar-layout">
    <el-header class="custom-header">
      <div class="header-content">
        <img src="https://img.icons8.com/fluency/48/000000/medical-doctor.png" class="logo" alt="Logo" />
        <span class="dashboard-title">Gestión de Pacientes</span>
      </div>
    </el-header>
    <el-container>
      <el-aside width="220px" class="custom-aside">
        <el-menu
          default-active="1"
          class="el-menu-vertical-demo custom-menu"
          @select="handleSelect"
          background-color="#22304a"
          text-color="#fff"
          active-text-color="#ffd04b"
        >
          <el-menu-item index="1">
            <i class="el-icon-user"></i>
            <span slot="title">Pacientes</span>
          </el-menu-item>
        </el-menu>
      </el-aside>
      <el-main class="main-content">
        <RegisterTable v-if="currentView === '1'" />
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
import RegisterTable from "./RegisterTable.vue";

export default {
  name: "SidebarPacientes",
  components: {
    RegisterTable,
  },
  data() {
    return {
      currentView: "1",
    };
  },
  methods: {
    handleSelect(index) {
      this.currentView = index;
    },
  },
};
</script>

<style lang="scss" scoped>
.sidebar-layout {
  min-height: 100vh;
  background: linear-gradient(135deg, #e0e7ef 0%, #f5f7fa 100%);
  margin: 0;
  padding: 0;
}

.custom-header {
  background: linear-gradient(90deg, #22304a 60%, #2d3a4b 100%);
  color: #fff;
  font-size: 28px;
  height: 70px;
  display: flex;
  align-items: center;
  box-shadow: 0 2px 8px rgba(34, 48, 74, 0.08);
  padding: 0 32px;
}
.header-content {
  display: flex;
  align-items: center;
  gap: 16px;
}
.logo {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 2px 8px rgba(34, 48, 74, 0.12);
}
.dashboard-title {
  font-weight: 600;
  font-size: 1.5rem;
  letter-spacing: 1px;
}
.custom-aside {
  background: linear-gradient(180deg, #22304a 80%, #2d3a4b 100%);
  color: #fff;
  min-height: calc(100vh - 70px);
  box-shadow: 2px 0 8px rgba(34, 48, 74, 0.08);
  padding-top: 0;
}
.custom-menu {
  border-right: none;
  background: transparent;
  font-size: 18px;
  padding-top: 24px;
}
.custom-menu .el-menu-item {
  border-radius: 8px;
  margin: 8px 12px;
  transition: background 0.2s;
}
.custom-menu .el-menu-item.is-active {
  background: linear-gradient(90deg, #ffd04b 60%, #ffe9a7 100%);
  color: #22304a !important;
  font-weight: bold;
}
.main-content {
  background: #f5f7fa;
  min-height: calc(100vh - 70px);
  padding: 32px 24px 24px 24px;
  border-radius: 16px;
  margin: 16px 0 16px 0;
  box-shadow: 0 2px 12px rgba(34, 48, 74, 0.06);
}
@media (max-width: 900px) {
  .custom-aside {
    width: 60px !important;
    min-width: 60px !important;
    padding: 0;
  }
  .custom-menu .el-menu-item span {
    display: none;
  }
  .main-content {
    padding: 12px 4px;
    margin: 0;
    border-radius: 0;
  }
}
</style>
