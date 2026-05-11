# Deployment Guide — MiniJira

Guia de despliegue a produccion del backend (Spring Boot) y el frontend (React / Vite). Lee esto completo antes de ejecutar cualquier comando en un entorno productivo.

---

## Pre-requisitos de produccion

Antes de desplegar, verificar que:

- PostgreSQL esta accesible desde el servidor del backend (host, puerto, credenciales).
- Todas las variables de entorno estan configuradas con valores reales — ningun default de desarrollo debe quedar activo.
- `JWT_SECRET` tiene al menos 32 caracteres, es unico por entorno y no esta en el repositorio.
- El origen del frontend de produccion esta incluido en `CORS_ALLOWED_ORIGINS`.
- `JPA_DDL_AUTO` esta configurado en `validate` o `none` (no `update`) para proteger el schema.

---

## Variables de entorno — produccion

Todas las variables marcadas como requeridas deben estar presentes. El backend no arranca si falta `DB_URL`, `DB_USERNAME` o `DB_PASSWORD` porque no tienen fallback valido en produccion.

| Variable | Requerida | Descripcion | Ejemplo produccion |
|---|---|---|---|
| `DB_URL` | Si | URL JDBC de PostgreSQL | `jdbc:postgresql://db.ejemplo.com:5432/minijira_prod` |
| `DB_USERNAME` | Si | Usuario de la base de datos | (valor en secrets manager) |
| `DB_PASSWORD` | Si | Contrasena de la base de datos | (valor en secrets manager) |
| `JWT_SECRET` | Si | Clave HMAC-SHA, minimo 32 caracteres | (valor en secrets manager) |
| `CORS_ALLOWED_ORIGINS` | Si | Origen del frontend en produccion | `https://app.ejemplo.com` |
| `JWT_EXPIRATION` | No | Expiracion del token en ms (default 86400000 = 24 h) | `3600000` para 1 h |
| `SERVER_PORT` | No | Puerto HTTP del servidor (default 8080) | `8080` |
| `JPA_DDL_AUTO` | Si | Estrategia DDL de Hibernate | `validate` |

**Nunca hardcodear estos valores en el codigo ni en archivos versionados.** Usar variables de entorno del sistema, un secrets manager (AWS Secrets Manager, GCP Secret Manager, Vault) o el mecanismo de secrets de la plataforma de CI/CD.

---

## Despliegue del backend

### Paso 1 — Build del JAR

```bash
cd MiniJira-BE
mvn clean package -DskipTests
```

El parametro `-DskipTests` omite los tests durante el build. Asegurate de que los tests pasaron en CI antes de llegar a este punto.

El artefacto generado queda en:

```
target/backend-0.0.1-SNAPSHOT.jar
```

### Paso 2 — Ejecutar el JAR

Las variables de entorno se pasan al proceso Java. Hay dos formas equivalentes:

**Opcion A — variables de entorno del sistema (recomendada):**

```bash
export DB_URL=jdbc:postgresql://db.ejemplo.com:5432/minijira_prod
export DB_USERNAME=usuario_prod
export DB_PASSWORD=contrasena_prod
export JWT_SECRET=clave-secreta-de-produccion-minimo-32-chars
export CORS_ALLOWED_ORIGINS=https://app.ejemplo.com
export JPA_DDL_AUTO=validate

java -jar target/backend-0.0.1-SNAPSHOT.jar
```

**Opcion B — propiedades en la linea de comando:**

```bash
java \
  -DSERVER_PORT=8080 \
  -DDB_URL=jdbc:postgresql://db.ejemplo.com:5432/minijira_prod \
  -DDB_USERNAME=usuario_prod \
  -DDB_PASSWORD=contrasena_prod \
  -DJWT_SECRET=clave-secreta-de-produccion-minimo-32-chars \
  -DCORS_ALLOWED_ORIGINS=https://app.ejemplo.com \
  -DJPA_DDL_AUTO=validate \
  -jar target/backend-0.0.1-SNAPSHOT.jar
```

### Paso 3 — Verificacion

Una vez arrancado, verificar que el servidor responde:

```bash
# Si el endpoint de tareas requiere autenticacion, un 401 confirma que el servidor esta activo
curl -i http://localhost:8080/api/tasks

# Esperado: HTTP/1.1 403 o HTTP/1.1 401 (Spring Security activo)
# Si ves 200 sin token, revisar la configuracion de SecurityConfig
```

Si Spring Actuator esta habilitado en el proyecto:

```bash
curl http://localhost:8080/actuator/health
# Esperado: {"status":"UP"}
```

Nota: Actuator no esta incluido en el `pom.xml` actual. Si se agrega en el futuro, proteger `/actuator` con autenticacion.

---

## Despliegue del frontend

### Paso 1 — Actualizar la URL del backend

La URL base del backend esta definida como constante en `src/api.ts`:

```ts
const BASE_URL = 'http://localhost:8080/api';
```

Antes del build de produccion, cambiar este valor a la URL real del backend:

```ts
const BASE_URL = 'https://api.ejemplo.com/api';
```

Alternativa recomendada a largo plazo: usar una variable de entorno de Vite (`import.meta.env.VITE_API_URL`) para no tener que tocar el codigo en cada deploy.

### Paso 2 — Build de produccion

```bash
cd MiniJira-FE
npm install
npm run build
```

El script `build` corre TypeScript (`tsc --noEmit`) y luego Vite. El output queda en:

```
dist/
├── index.html
└── assets/
    └── index-<hash>.js
```

### Paso 3 — Servir los archivos estaticos

La carpeta `dist/` contiene una SPA (Single Page Application). Cualquier static host o servidor web puede servirla. Opciones comunes:

**nginx:**

```nginx
server {
    listen 80;
    server_name app.ejemplo.com;
    root /var/www/minijira/dist;
    index index.html;

    # Necesario para routing de SPA (React Router)
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**Vercel / Netlify:**
Apuntar el directorio de publish a `dist/`. Ambas plataformas detectan Vite automaticamente y configuran el rewrite de SPA.

**Servidor estatico rapido (para validacion):**

```bash
npx serve dist
```

---

## Riesgos conocidos

### `JPA_DDL_AUTO=update` en produccion

El default de desarrollo es `update`, que permite a Hibernate modificar el schema de la base de datos al iniciar la aplicacion. En produccion esto es peligroso: puede alterar tablas existentes de forma involuntaria.

**Accion requerida:** configurar `JPA_DDL_AUTO=validate` en produccion. Con `validate`, Hibernate verifica que el schema coincide con las entidades pero no lo modifica. Si hay diferencias, el servidor no arranca y el error queda en el log — mucho mejor que una migracion silenciosa.

Para migraciones controladas de schema en produccion, evaluar incorporar Flyway o Liquibase.

### Campo `password` en la entidad User

La entidad `User` almacena el hash BCrypt de la contrasena. La clase `UserResponse` (DTO de salida) debe excluir este campo — verificar que no se expose en `GET /api/users` ni en `GET /api/users/{id}`.

### URL del backend hardcodeada en el frontend

`src/api.ts` tiene `http://localhost:8080/api` como constante. Si se hace un build sin cambiar esta linea, el frontend de produccion intentara conectar al localhost del usuario. Verificar siempre antes del build.

### JWT_SECRET debil o predecible

El default de desarrollo (`cambiar-esta-clave-en-produccion-minimo-32-chars`) es un string conocido publicamente. En produccion, cualquier atacante que lo conozca puede forjar tokens validos. Generar una clave aleatoria de al menos 32 caracteres:

```bash
openssl rand -base64 48
```

---

## Rollback

El proceso de rollback depende de como este configurado el entorno. En lineas generales:

**Backend:**
1. Detener el proceso Java.
2. Desplegar el JAR de la version anterior.
3. Si hubo migracion de schema, restaurar backup de base de datos antes de arrancar.

**Frontend:**
1. Restaurar el `dist/` de la version anterior en el static host.
2. En Vercel/Netlify: usar la funcion de "Promote to Production" sobre el deployment anterior.

**Consejo:** etiquetar cada build con el commit SHA para poder identificar rapidamente que version esta corriendo:

```bash
# incluir el SHA en el nombre del JAR o en una variable de entorno de la app
git rev-parse --short HEAD
```
