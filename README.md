# Sistema de Gestión de Pacientes

Sistema fullstack para la gestión de pacientes, desarrollado conforme a los requisitos de la prueba técnica de Sinergia 2025. Implementado la interfaz grafica con Vue.js 2.6+. El sistema permite crear, editar, listar y visualizar pacientes con validaciones.


### Backend
- PHP 8.2+
- Laravel 12
- JWT Auth (tymon/jwt-auth) para autenticación
- Eloquent ORM para manejo de datos
- API Resources para transformación de respuestas
- Validación de requests con Form Requests

### Base de Datos
- MySQL 8.0+
- Migraciones y seeders para gestión de esquema

## 📋 Frontend

- Vue.js 2.6+
- Axios y Fetch para consumo de API
- Element UI (Elegido en este caso por mayor dominio)
- Validaciones en tiempo real y feedback al usuario
- Formularios reactivos con estados de botones condicionales

## 📋 Infraestructura

- Docker
- Nginx como servidor web
- MySQL como base de datos

## 🏗️ Estructura del Proyecto

```
├── backend_pacientes/           # Backend Laravel API
│   ├── app/                     # Código principal de la aplicación (modelos, controladores, requests)
│   │   ├── Http/Controllers/    # Controladores de la API
│   │   ├── Models/              # Modelos Eloquent
│   │   └── ...
│   ├── config/                  # Configuración de Laravel
│   ├── database/                # Migraciones, seeders y factories
│   ├── public/                  # DocumentRoot para Nginx (index.php, assets públicos)
│   ├── resources/               # Vistas y recursos
│   ├── routes/                  # Definición de rutas (api.php, web.php)
│   ├── storage/                 # Archivos generados y logs
│   ├── tests/                   # Pruebas unitarias y funcionales
│   ├── composer.json            # Dependencias PHP
│   ├── Dockerfile               # Dockerfile para backend
│   └── ...
│
├── frontend_pacientes/          # Aplicación Vue.js 2.6
│   ├── src/                     # Componentes, vistas, servicios, utilidades
│   ├── public/                  # Assets públicos
│   ├── package.json             # Dependencias npm
│   ├── .env                     # Variables de entorno
│   ├── vue.config.js            # Configuración de Vue CLI
│   ├── Dockerfile               # Dockerfile para frontend
│   └── ...
│
├── nginx/                       # Configuración de servidor web reverse proxy
│   └── default.conf             # Configuración de Nginx
│
├── docker-compose.yml           # Orquestación de contenedores
└── README.md                    # Documentación del proyecto
```

## 🔌 Endpoints de la API

### Autenticación

| Método | Endpoint            | Descripción                        | Auth |
| ------ | ------------------- | ---------------------------------- | ---- |
| POST   | `/api/auth/login`   | Iniciar sesión y obtener token JWT | No   |
| POST   | `/api/auth/logout`  | Cerrar sesión                      | Sí   |
| POST   | `/api/auth/refresh` | Renovar token JWT                  | Sí   |

**Ejemplo de login:**

```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"password"}'
```

### Pacientes (Requiere autenticación)

| Método | Endpoint              | Descripción                     | Auth |
| ------ | --------------------- | ------------------------------- | ---- |
| GET    | `/api/pacientes`      | Listar pacientes (paginado)     | Sí   |
| GET    | `/api/pacientes/{id}` | Obtener detalles de un paciente | Sí   |
| POST   | `/api/pacientes`      | Crear nuevo paciente            | Sí   |
| PUT    | `/api/pacientes/{id}` | Actualizar paciente existente   | Sí   |
| DELETE | `/api/pacientes/{id}` | Eliminar paciente               | Sí   |

**Headers requeridos para rutas protegidas:**

```
Authorization: Bearer {token_jwt}
Content-Type: application/json
Accept: application/json
```

### Catálogos Maestros (Públicos)

| Método | Endpoint                               | Descripción                        | Auth |
| ------ | -------------------------------------- | ---------------------------------- | ---- |
| GET    | `/api/departamentos`                   | Listar departamentos               | No   |
| GET    | `/api/municipios?departamento_id={id}` | Listar municipios por departamento | No   |
| GET    | `/api/tipo-documentos`                 | Listar tipos de documento          | No   |
| GET    | `/api/generos`                         | Listar géneros                     | No   |

## 🔧 Instalación y Configuración

### 1. Clonar el repositorio

```bash
git clone https://github.com/Elian-Canas/frontend_pacientes.git
cd frontend_pacientes
```

### 2. Configurar variables de entorno

De acuerdo al archivo de ejemplo y configurar las variables de entorno principalmente indicando la IP de su equipo _Frontend_:

```bash
cp .env-production .env
Dirigirse al archivo src/config.js e indicar la IP del equipo en apiBaseUrl: "http://192.168.1.22:8080"
```

### 3. Construir e iniciar los contenedores

Previamente tener instalado en el equipo Docker y ejecutar el siguiente comando

```bash
docker build -t pacientes-vue .
```

Posteriormente ejecutar el contenedor
```bash
docker run -d \
  --name frontend_vue \
  -p 8081:80 \
  pacientes-vue
```

La aplicación estará disponible en `http://localhost:8081`


## 🚀 Despliegue

### Desarrollo Local (Sin Docker)

```bash
cd frontend
npm run serve
```

## 👨‍💻 Autor

**Elian Santiago Cañas**

## 🔗 Repositorios Relacionados

- **Backend**: [pacientes_backend](https://github.com/Elian-Canas/frontend_pacientes.git)
