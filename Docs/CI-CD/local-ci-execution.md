# Ejecución local del CI

Antes de hacer push a `test-05-cicd_v01`, podés correr el mismo proceso que corre GitHub Actions en tu máquina. Esto te ahorra tiempo de feedback: detectas errores en segundos en vez de esperar el runner remoto.

## Prerequisitos

| Herramienta | Versión requerida | Cómo verificar |
|---|---|---|
| Node.js | 20.x | `node --version` |
| npm | >= 10 (viene con Node 20) | `npm --version` |
| Java (JDK) | 17 | `java -version` |
| Maven | >= 3.8 | `mvn --version` |
| Docker | cualquier versión reciente | `docker --version` |
| Docker Compose | v2 (plugin, no standalone) | `docker compose version` |

Si no tenés Java 17 instalado, la forma más simple en Linux/Mac es con SDKMAN:
```bash
sdk install java 17.0.11-tem
```

## Frontend — script local

El script `scripts/run-ci-frontend-local.sh` replica los 5 pasos del pipeline en tu máquina.

```bash
# Desde la raíz de MiniJira-FE
bash scripts/run-ci-frontend-local.sh
```

El script ejecuta en orden:
1. `npm ci` — instala dependencias exactas del lockfile
2. Lint (ESLint) — solo si existe `eslint.config.js` o equivalente
3. Tests (Vitest) — solo si existen archivos `*.test.*` o `*.spec.*` en `src/`
4. `npm run build` — build de producción en `dist/`
5. `npm audit --audit-level=high` — solo si existe `package-lock.json`

Al final imprime un resumen:
```
======================================
  RESUMEN CI LOCAL
======================================
  Pasaron : 5
  Fallaron: 0
  Saltados: 0 (pendientes documentados)
======================================
  Estado: OK — seguro para push.
```

Si el script termina con estado `FALLO`, corregí antes de hacer push. El pipeline en GitHub Actions fallará por la misma razón.

## Backend — script local

El script `scripts/run-ci-backend-local.sh` replica los 3 pasos del pipeline backend.

```bash
# Desde la raíz de MiniJira-BE
bash scripts/run-ci-backend-local.sh
```

El script ejecuta en orden:
1. `mvn clean test --no-transfer-progress` — tests unitarios JUnit/Mockito
2. `mvn verify --no-transfer-progress` — verificación integrada
3. `mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7 --no-transfer-progress` — escaneo de vulnerabilidades

Aviso importante sobre el paso 3: la primera vez que corras el OWASP check, Maven descarga la base de datos NVD del NIST (~300 MB). Esto puede tardar 5-10 minutos. Las ejecuciones siguientes usan la base de datos cacheada en `~/.m2/repository/org/owasp/`.

## Frontend — Docker

El `Dockerfile` del frontend usa multi-stage build: compila con Node 20 y sirve con nginx.

```bash
# Desde la raíz de MiniJira-FE
docker compose up --build
```

El frontend queda disponible en `http://localhost:3000`.

Para detener:
```bash
docker compose down
```

El `docker-compose.yml` del frontend solo levanta el servicio `frontend` (puerto 3000 -> 80 del container nginx).

## Backend — Docker Compose

El backend requiere PostgreSQL. El `docker-compose.yml` del backend levanta dos servicios: `postgres` y `backend`.

**Paso previo obligatorio — configurar variables de entorno:**

```bash
# Desde la raíz de MiniJira-BE
cp .env.example .env
```

Editá `.env` y completá las variables si querés cambiar los valores por defecto. Los valores del `.env.example` son funcionales para desarrollo local. No commitees el archivo `.env`.

Variables del `.env.example`:

```
POSTGRES_DB=minijira
POSTGRES_USER=minijira_user
POSTGRES_PASSWORD=changeme_local

SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/minijira
SPRING_DATASOURCE_USERNAME=minijira_user
SPRING_DATASOURCE_PASSWORD=changeme_local

JWT_SECRET=changeme_jwt_secret_minimo_32_caracteres
JWT_EXPIRATION_MS=86400000
```

Una vez configurado el `.env`:

```bash
# Desde la raíz de MiniJira-BE
docker compose up --build
```

Servicios levantados:
- `postgres` en puerto `5432` (Postgres 16 Alpine)
- `backend` (Spring Boot) en puerto `8080`

El backend espera a que Postgres esté healthy antes de arrancar (healthcheck configurado con `pg_isready`).

Para detener y remover volúmenes:
```bash
docker compose down -v
```

El flag `-v` elimina el volumen `postgres_data`. Omitilo si querés conservar los datos entre reinicios.
