# Sistema de Gestión de Licitaciones

Sistema fullstack para la gestión de pacientes, desarrollado conforme a los requisitos de la prueba técnica de Sinergia 2025. Implementado la interfaz grafica con Vue.js 2.6+. El sistema permite crear, editar, listar y visualizar pacientes con validaciones.

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
├── frontend/             # Aplicación Vue.js 2.6
│   ├── src/              # Componentes, vistas, store, servicios
│   ├── public/           # Assets públicos
│   ├── package.json      # Dependencias npm
│   ├── .env              # Variables de entorno
│   └── vue.config.js     # Configuración de Vue CLI
│
├── nginx/                # Configuración de servidor web reverse proxy
│   └── default.conf      # Configuración de Nginx
│
├── docker-compose.yml    # Orquestación de contenedores
└── README.md             # Documentación del proyecto
``` 

## 🔧 Instalación y Configuración

### 1. Clonar el repositorio

```bash
git clone https://github.com/Elian-Canas/frontend_pacientes.git
cd frontend_pacientes
```

### 2. Configurar variables de entorno

De acuerdo al archivo de ejemplo y configurar las variables de entorno principalmente indicando la IP de su equipo *Frontend*:

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
  --name vue_front \
  -p 8080:80 \
  pacientes-vue
```

La aplicación estará disponible en `http://localhost:8080`


## 🔌 Integración con Backend

El frontend se comunica con el backend PHP a través de los siguientes endpoints:

- `GET /ofertas` – Obtener listado paginado de ofertas con filtros opcionales
- `GET /ofertas/{id}` – Obtener los detalles completos de una oferta específica
- `GET /ofertas/export` – Exportar listado de ofertas filtrado a Excel
- `POST /ofertas` – Crear una nueva oferta
- `PUT /ofertas/{id}` – Actualizar una oferta existente
- `POST /documentos` – Subir documentos asociados a una oferta (solo en edición)
- `GET /actividades` – Obtener listado completo de actividades (maestra UNSPSC)
- `GET /actividades/buscar` – Buscar actividades por nombre o código
- `GET /actividades/{id}` – Obtener detalles de una actividad específica

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