# Mini Jira

Sistema de gestión de tareas estilo Kanban, con backend Spring Boot y frontend React. Permite crear proyectos, asignar tareas a usuarios, y mover tarjetas entre columnas TODO / IN_PROGRESS / DONE.

---

## Requisitos previos

| Herramienta | Version minima |
|-------------|---------------|
| Java        | 17            |
| Maven       | 3.8+          |
| PostgreSQL  | 14+           |
| Node.js     | 18+           |
| npm         | 9+            |

---

## Setup backend

```bash
# 1. Clonar el repositorio del backend
git clone -b feat/test-01-fullstack https://github.com/Alitocoin/MiniJira-BE.git
cd MiniJira-BE

# 2. Crear la base de datos en PostgreSQL
psql -U postgres -c "CREATE DATABASE minijira;"

# 3. Configurar credenciales — editar src/main/resources/application.properties
#    spring.datasource.url=jdbc:postgresql://localhost:5432/minijira
#    spring.datasource.username=<tu_usuario>
#    spring.datasource.password=<DB_PASSWORD>

# 4. Compilar y levantar
mvn spring-boot:run
```

El servidor queda escuchando en `http://localhost:8080`.

Para correr los tests (usa H2 en memoria, no necesita PostgreSQL):

```bash
mvn test
```

---

## Setup frontend

```bash
# 1. Clonar el repositorio del frontend
git clone -b feat/test-01-fullstack https://github.com/Alitocoin/MiniJira-FE.git
cd MiniJira-FE

# 2. Instalar dependencias
npm install

# 3. Levantar en modo desarrollo
npm run dev
```

La SPA queda disponible en `http://localhost:5173` y conecta automáticamente al backend en `http://localhost:8080`.

---

## URLs de acceso

| Servicio          | URL                          |
|-------------------|------------------------------|
| Frontend (Vite)   | http://localhost:5173         |
| Backend API       | http://localhost:8080/api     |
| H2 Console (test) | http://localhost:8080/h2-console |

---

## Estructura del proyecto

```
MiniJira-BE/                          # Backend Spring Boot
├── src/main/java/com/minijira/backend/
│   ├── controller/                   # Endpoints REST
│   ├── service/                      # Logica de negocio
│   ├── repository/                   # Acceso a datos (JPA)
│   ├── model/                        # Entidades JPA (User, Task, Project)
│   └── dto/                          # Data Transfer Objects
├── src/main/resources/
│   └── application.properties        # Config datasource, JPA
└── src/test/                         # Tests con H2

MiniJira-FE/                          # Frontend React
├── src/
│   ├── components/
│   │   ├── BoardColumn.tsx            # Columna Kanban (TODO/IN_PROGRESS/DONE)
│   │   ├── TaskCard.tsx               # Tarjeta de tarea
│   │   └── CreateTaskForm.tsx         # Formulario de nueva tarea
│   ├── App.tsx                        # Componente raiz, estado global
│   └── main.tsx                       # Entry point
└── vite.config.ts
```

---

## Documentacion adicional

| Documento                        | Contenido                                      |
|----------------------------------|------------------------------------------------|
| [docs/api-reference.md](docs/api-reference.md)   | Referencia completa de la API REST |
| [docs/architecture.md](docs/architecture.md)     | Diagramas C4 y decisiones tecnicas |
| [docs/test-01-metrics.md](docs/test-01-metrics.md) | Metricas de evaluacion Test 01   |
