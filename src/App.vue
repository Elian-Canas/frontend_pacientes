<template>
  <div id="app">
    <!-- Pantalla de login -->
    <div v-if="!isLoggedIn" class="login-background">
      <el-card class="login-card">
        <div class="login-header">
          <el-avatar :size="50" icon="el-icon-user"></el-avatar>
          <h3>Iniciar sesión</h3>
        </div>

        <el-form :model="form" :rules="rules" ref="loginForm" label-width="0px">
          <el-form-item prop="email">
            <el-input
              v-model="form.email"
              placeholder="Correo Electronico"
              prefix-icon="el-icon-user"
              clearable
            ></el-input>
          </el-form-item>

          <el-form-item prop="password">
            <el-input
              v-model="form.password"
              type="password"
              placeholder="Contraseña"
              prefix-icon="el-icon-lock"
              show-password
            ></el-input>
          </el-form-item>

          <el-form-item>
            <el-button
              type="primary"
              :loading="loading"
              @click="submitForm('loginForm')"
              style="width:100%"
            >
              {{ loading ? 'Iniciando...' : 'Iniciar sesión' }}
            </el-button>
          </el-form-item>
        </el-form>

        <p v-if="error" class="error-message">{{ error }}</p>
      </el-card>
    </div>

    <!-- Layout principal: Sidebar + Contenido -->
    <div v-else class="app-layout">
      <Sidebar class="sidebar" />
    </div>
  </div>
</template>

<script>
import Sidebar from './components/Sidebar.vue';

const API_BASE_URL = process.env.VUE_APP_API_BASE_URL;

export default {
  name: 'App',
  components: {
    Sidebar
  },
  data() {
    return {
      form: {
        email: '',
        password: ''
      },
      loading: false,
      error: '',
      isLoggedIn: !!localStorage.getItem('access_token'), // ✅ reactivo
      rules: {
        email: [
          { required: true, message: 'Por favor ingresa tu correo', trigger: 'blur' }
        ],
        password: [
          { required: true, message: 'Por favor ingresa tu contraseña', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    // Sincroniza al iniciar (por si hay sesión guardada)
    this.isLoggedIn = !!localStorage.getItem('access_token');
  },
  methods: {
    async submitForm(formName) {
      this.$refs[formName].validate(async (valid) => {
        if (!valid) return;

        this.loading = true;
        this.error = '';

        try {
          const response = await fetch(`${API_BASE_URL}/api/auth/login`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({
              email: this.form.email,
              password: this.form.password
            })
          });

          if (!response.ok) {
            const data = await response.json();
            throw new Error(data.message || 'Credenciales incorrectas');
          }

          const result = await response.json();

          localStorage.setItem('access_token', result.access_token);
          localStorage.setItem('token_type', result.token_type);

          this.isLoggedIn = true; // ✅ ¡Esto fuerza la actualización de la vista!

        } catch (err) {
          this.error = err.message || 'Error al iniciar sesión';
        } finally {
          this.loading = false;
        }
      });
    }
  }
}
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  min-height: 100vh;
  height: 100vh;
  margin: 0;
  padding: 0;
  background: linear-gradient(135deg, #e0e7ef 0%, #f5f7fa 100%);
}

body {
  margin: 0 !important;
  padding: 0 !important;
  background: transparent !important;
}

/* --- Login --- */
.login-background {
  background: linear-gradient(135deg, #4b6cb7, #182848);
  min-height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0;
  margin: 0;
}

.login-card {
  width: 400px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  padding: 30px;
}

.login-header {
  text-align: center;
  margin-bottom: 30px;
}

.login-header .el-avatar {
  background-color: #4b6cb7;
  margin-bottom: 10px;
}

.login-header h3 {
  margin: 0;
  color: #333;
  font-weight: 500;
}

.error-message {
  color: #f56c6c;
  font-size: 14px;
  text-align: center;
  margin-top: 15px;
}

/* --- Layout principal --- */
.app-layout {
  display: flex;
  height: 100vh;
  overflow: hidden;
  background: transparent;
  margin: 0;
  padding: 0;
}

.sidebar {
  flex-shrink: 0;
  height: 100%;
}

.main-content {
  flex: 1;
  overflow-y: auto;
  background-color: #f5f7fa;
  padding: 20px;
}
</style>

<style>
html, body {
  margin: 0 !important;
  padding: 0 !important;
  height: 100% !important;
  overflow: hidden !important;
}
</style>