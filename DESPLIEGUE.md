# 🚀 Guía de Despliegue — Cómo Hacer Funcionar la App

Esta guía explica paso a paso cómo poner en funcionamiento el Sistema de Gestión Clínica, desde cero hasta tener la aplicación corriendo en producción.

---

## Tabla de Contenidos

1. [Requisitos Previos](#1-requisitos-previos)
2. [Instalación en Desarrollo Local](#2-instalación-en-desarrollo-local)
3. [Configuración del Entorno](#3-configuración-del-entorno)
4. [Levantar la Base de Datos](#4-levantar-la-base-de-datos)
5. [Ejecutar Migraciones y Seed](#5-ejecutar-migraciones-y-seed)
6. [Iniciar en Modo Desarrollo](#6-iniciar-en-modo-desarrollo)
7. [Verificar que Todo Funciona](#7-verificar-que-todo-funciona)
8. [Compilar para Producción](#8-compilar-para-producción)
9. [Despliegue en Producción (Docker)](#9-despliegue-en-producción-docker)
10. [Configurar Nginx y HTTPS](#10-configurar-nginx-y-https)
11. [Acceso Remoto Seguro](#11-acceso-remoto-seguro)
12. [Mantenimiento y Monitoreo](#12-mantenimiento-y-monitoreo)
13. [Solución de Problemas Comunes](#13-solución-de-problemas-comunes)

---

## 1. Requisitos Previos

### Software necesario:

| Herramienta | Versión mínima | Para qué |
|-------------|---------------|----------|
| **Node.js** | 18.x o superior | Runtime del backend y frontend |
| **npm** | 10.x o superior | Gestor de paquetes (viene con Node.js) |
| **Docker Desktop** | 4.x o superior | Ejecutar PostgreSQL y Redis |
| **Git** | 2.x | Control de versiones |

### Para producción adicional:

| Herramienta | Para qué |
|-------------|----------|
| **Docker Compose** | Orquestar todos los servicios |
| **Nginx** | Reverse proxy + HTTPS |
| **Certbot** (opcional) | Certificados SSL gratuitos |

### Verificar instalaciones:

```bash
node --version       # Debe mostrar v18.x o superior
npm --version        # Debe mostrar 10.x o superior
docker --version     # Docker Desktop instalado
git --version        # Git instalado
```

### Instalar Node.js (si no lo tienes):

1. Descarga de https://nodejs.org/ (versión LTS)
2. Instala con las opciones por defecto
3. Reinicia la terminal

### Instalar Docker Desktop (si no lo tienes):

1. Descarga de https://www.docker.com/products/docker-desktop/
2. Instala y reinicia Windows
3. Abre Docker Desktop y espera a que arranque

---

## 2. Instalación en Desarrollo Local

### Paso 1: Clonar el repositorio

```bash
git clone <url-de-tu-repositorio>
cd historial_clinico
```

### Paso 2: Instalar TODAS las dependencias

```bash
npm install
```

Esto instala las dependencias de:
- La raíz del proyecto (Turborepo, TypeScript, Prettier)
- `apps/api` (NestJS, Prisma, pdf-lib, etc.)
- `apps/web` (React, Vite, Tailwind, etc.)
- `packages/shared-types`, `packages/validators`, `packages/constants`

### Paso 3: Compilar los paquetes compartidos

```bash
npm run build
```

Esto compila los paquetes en `packages/` que son dependencias de `apps/api` y `apps/web`.

---

## 3. Configuración del Entorno

### Paso 1: Crear el archivo .env

```bash
copy .env.example .env
```

### Paso 2: Editar .env con tus datos

Abre `.env` con un editor de texto y cambia como mínimo:

```env
# Contraseña de la base de datos (CAMBIA ESTO)
POSTGRES_PASSWORD=tu_contraseña_segura_aqui

# Actualiza DATABASE_URL con la misma contraseña
DATABASE_URL=postgresql://historial:tu_contraseña_segura_aqui@localhost:5432/historial_clinico

# Secretos JWT (GENERA valores aleatorios)
JWT_ACCESS_SECRET=un-string-aleatorio-de-al-menos-32-caracteres
JWT_REFRESH_SECRET=otro-string-aleatorio-diferente-de-32-caracteres

# Clave de encriptación de respaldos
BACKUP_ENCRYPTION_KEY=clave-segura-para-encriptar-respaldos
```

### Paso 3: (Opcional) Configurar OpenAI

Si quieres la funcionalidad de resumen clínico por IA:

```env
OPENAI_API_KEY=sk-proj-tu-api-key-de-openai
```

---

## 4. Levantar la Base de Datos

### Opción A: Con Docker (recomendado)

```bash
docker compose up -d postgres redis
```

Esto levanta:
- **PostgreSQL 16** en el puerto 5432
- **Redis 7** en el puerto 6379

### Verificar que están corriendo:

```bash
docker compose ps
```

Debes ver ambos servicios con estado "running" y "healthy".

### Opción B: Sin Docker (PostgreSQL y Redis instalados localmente)

Si ya tienes PostgreSQL y Redis instalados en tu máquina:
1. Crea la base de datos: `createdb historial_clinico`
2. Asegúrate de que Redis está corriendo en puerto 6379
3. Ajusta `DATABASE_URL` y `REDIS_URL` en `.env` según tu configuración

---

## 5. Ejecutar Migraciones y Seed

### Paso 1: Generar el cliente Prisma

```bash
npx prisma generate
```

### Paso 2: Aplicar todas las migraciones

```bash
npx prisma migrate dev
```

Esto crea todas las tablas, índices, triggers y permisos en PostgreSQL.

### Paso 3: Cargar datos iniciales

```bash
npx prisma db seed
```

Esto crea:
- ✅ Usuario administrador: `admin` / `Admin123!`
- ✅ Clínica demo con datos de ejemplo
- ✅ Tema por defecto (colores azules)

### Verificar con Prisma Studio:

```bash
npx prisma studio
```

Se abre http://localhost:5555 — navega por las tablas y verifica que los datos se crearon.

---

## 6. Iniciar en Modo Desarrollo

### Comando único (levanta API + Web simultáneamente):

```bash
npm run dev
```

Turborepo ejecuta ambas apps en paralelo:
- **Backend (NestJS)**: http://localhost:3000
- **Frontend (Vite + React)**: http://localhost:5173

### También puedes iniciar cada uno por separado:

```bash
# Terminal 1 — Backend
cd apps/api
npm run dev

# Terminal 2 — Frontend
cd apps/web
npm run dev
```

---

## 7. Verificar que Todo Funciona

### Checklist de verificación:

| # | Verificar | Cómo | Resultado esperado |
|---|-----------|------|--------------------|
| 1 | Backend responde | Abrir http://localhost:3000 | Respuesta del API |
| 2 | Frontend carga | Abrir http://localhost:5173 | Pantalla de login |
| 3 | Login funciona | Ingresar `admin` / `Admin123!` | Redirige al dashboard |
| 4 | BD conectada | Prisma Studio (puerto 5555) | Datos visibles |
| 5 | Redis conectado | Login + esperar 15min | Sesión se cierra por inactividad |

### Probar el login:

1. Abre http://localhost:5173
2. Ingresa:
   - Usuario: `admin`
   - Contraseña: `Admin123!`
3. Deberías ver el dashboard de administrador

### Ejecutar los tests:

```bash
# Todos los tests del proyecto
npm run test

# Solo tests del backend
cd apps/api
npx vitest run

# Un test específico
npx vitest run src/modules/auth/auth.service.test.ts
```

---

## 8. Compilar para Producción

### Paso 1: Compilar todo

```bash
npm run build
```

Esto genera:
- `apps/api/dist/` — Backend compilado a JavaScript
- `apps/web/dist/` — Frontend optimizado (HTML/CSS/JS estáticos)

### Paso 2: Probar la compilación localmente

```bash
# Backend
cd apps/api
npm run start:prod

# Frontend (servir archivos estáticos)
cd apps/web
npm run preview
```

---

## 9. Despliegue en Producción (Docker)

### Paso 1: Configurar .env para producción

```env
# Contraseñas SEGURAS
POSTGRES_PASSWORD=ContraseñaMuySegura2024!
DATABASE_URL=postgresql://historial:ContraseñaMuySegura2024!@postgres:5432/historial_clinico

# Secretos JWT SEGUROS
JWT_ACCESS_SECRET=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
JWT_REFRESH_SECRET=aW50ZXJuYWwtc2VjcmV0LWtleS1mb3ItcmVm...

# URL del frontend en producción
CORS_ORIGIN=https://tu-dominio.com

# Modo producción
NODE_ENV=production
```

### Paso 2: Levantar todos los servicios

```bash
docker compose up -d
```

Esto levanta:
1. **PostgreSQL** — Base de datos
2. **Redis** — Cache y sesiones
3. **API** — Backend NestJS (puerto 3000 interno)
4. **Nginx** — Reverse proxy (puertos 80 y 443)

### Paso 3: Ejecutar migraciones en el contenedor

```bash
docker compose exec api npx prisma migrate deploy
docker compose exec api npx prisma db seed
```

### Paso 4: Verificar

```bash
# Ver logs
docker compose logs -f api

# Verificar salud de servicios
docker compose ps
```

---

## 10. Configurar Nginx y HTTPS

### Crear la configuración de Nginx:

Crea el archivo `nginx/nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Compresión
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    # Límite de subida de archivos (200 MB)
    client_max_body_size 200M;

    server {
        listen 80;
        server_name tu-dominio.com;

        # Redirigir HTTP a HTTPS
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl;
        server_name tu-dominio.com;

        # Certificados SSL
        ssl_certificate     /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;
        ssl_protocols       TLSv1.2 TLSv1.3;

        # Headers de seguridad
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;

        # Frontend (archivos estáticos)
        location / {
            root /usr/share/nginx/html;
            try_files $uri $uri/ /index.html;
        }

        # Backend API
        location /api/ {
            proxy_pass http://api:3000/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # SSE (Server-Sent Events) para prescripciones en tiempo real
        location /api/prescriptions/events {
            proxy_pass http://api:3000/prescriptions/events;
            proxy_http_version 1.1;
            proxy_set_header Connection '';
            proxy_buffering off;
            proxy_cache off;
            chunked_transfer_encoding off;
        }
    }
}
```

### Certificados SSL:

#### Opción A: Let's Encrypt (gratuito)

```bash
# Instalar Certbot
sudo apt install certbot

# Generar certificado
sudo certbot certonly --standalone -d tu-dominio.com

# Los certificados se guardan en:
# /etc/letsencrypt/live/tu-dominio.com/fullchain.pem
# /etc/letsencrypt/live/tu-dominio.com/privkey.pem
```

#### Opción B: Certificado autofirmado (solo para desarrollo/pruebas)

```bash
mkdir -p nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/privkey.pem \
  -out nginx/ssl/fullchain.pem \
  -subj "/CN=localhost"
```

---

## 11. Acceso Remoto Seguro

Para que el doctor acceda desde fuera de la clínica de forma segura:

### Opción A: Tailscale (recomendado, fácil)

1. Instala Tailscale en el servidor: https://tailscale.com/download
2. Instala Tailscale en los dispositivos del doctor
3. Ambos se conectan a la misma red privada automáticamente
4. El doctor accede por IP de Tailscale: `https://100.x.x.x`

### Opción B: Cloudflare Tunnel (sin abrir puertos)

1. Crea una cuenta en Cloudflare
2. Instala `cloudflared` en el servidor
3. Crea un tunnel: `cloudflared tunnel create historial-clinico`
4. Configura el dominio
5. El doctor accede por el dominio: `https://clinica.tu-dominio.com`

### Opción C: VPN (WireGuard)

1. Configura WireGuard en el servidor
2. Genera claves para cada dispositivo del doctor
3. El doctor se conecta a la VPN antes de acceder

---

## 12. Mantenimiento y Monitoreo

### Ver logs en tiempo real:

```bash
# Todos los servicios
docker compose logs -f

# Solo el API
docker compose logs -f api

# Solo PostgreSQL
docker compose logs -f postgres
```

### Monitorear espacio en disco:

El sistema muestra una alerta automática en el panel de administración cuando el espacio disponible está bajo. Verifica manualmente:

```bash
# Windows
wmic logicaldisk get size,freespace,caption

# Linux
df -h
```

### Actualizar la aplicación:

```bash
# 1. Obtener cambios
git pull

# 2. Instalar nuevas dependencias
npm install

# 3. Compilar
npm run build

# 4. Aplicar migraciones nuevas
npx prisma migrate deploy

# 5. Reiniciar servicios
docker compose restart api
```

### Respaldos manuales:

```bash
# Trigger manual desde la API
curl -X POST http://localhost:3000/api/backups/trigger \
  -H "Authorization: Bearer <token-admin>"

# O dump directo de PostgreSQL
docker compose exec postgres pg_dump -U historial historial_clinico > backup_manual.sql
```

---

## 13. Solución de Problemas Comunes

### ❌ "Cannot connect to database"

**Causa**: PostgreSQL no está corriendo o las credenciales son incorrectas.

```bash
# Verificar que Docker está corriendo
docker compose ps

# Reiniciar PostgreSQL
docker compose restart postgres

# Verificar logs
docker compose logs postgres
```

### ❌ "ECONNREFUSED 127.0.0.1:6379"

**Causa**: Redis no está corriendo.

```bash
docker compose restart redis
docker compose logs redis
```

### ❌ "Prisma migrate: Database already contains data"

**Causa**: Ya existen tablas en la BD.

```bash
# Si es desarrollo y quieres empezar de cero:
npx prisma migrate reset

# ⚠️ ESTO BORRA TODOS LOS DATOS - solo en desarrollo
```

### ❌ "Module not found: @historial/shared-types"

**Causa**: Los paquetes compartidos no están compilados.

```bash
npm run build
```

### ❌ "Port 5432 already in use"

**Causa**: Otra instancia de PostgreSQL está usando el puerto.

```bash
# Detener el otro PostgreSQL o cambiar el puerto en docker-compose.yml:
ports:
  - '5433:5432'   # Usar puerto 5433 externamente

# Y actualizar DATABASE_URL:
DATABASE_URL=postgresql://historial:password@localhost:5433/historial_clinico
```

### ❌ "CORS error" en el navegador

**Causa**: La URL del frontend no coincide con `CORS_ORIGIN`.

```env
# Asegúrate de que coincida EXACTAMENTE:
CORS_ORIGIN=http://localhost:5173
# NO: http://localhost:5173/  (sin barra final)
# NO: https://localhost:5173  (http vs https)
```

### ❌ "JWT expired" o sesión se cierra inesperadamente

**Causa**: El token expiró y el refresh falló.

- Verifica que Redis está funcionando
- El refresh token tiene 7 días de vida
- La inactividad de 15 minutos cierra la sesión

### ❌ La PWA no se instala

**Causa**: Falta HTTPS o el manifest es inválido.

Requisitos para que la PWA sea instalable:
1. Servida sobre HTTPS (o localhost en desarrollo)
2. `manifest.webmanifest` válido con iconos
3. Service Worker registrado
4. Contenido visible en la primera carga

### ❌ "ENOMEM" o el servidor se queda sin memoria

```bash
# Verificar uso de memoria
docker stats

# Aumentar memoria del contenedor en docker-compose.yml:
api:
  deploy:
    resources:
      limits:
        memory: 2G
```

---

## 📋 Checklist de Despliegue Completo

Use este checklist para asegurarte de que no falta nada:

- [ ] Node.js 18+ instalado
- [ ] Docker Desktop funcionando
- [ ] `.env` configurado con secretos únicos
- [ ] PostgreSQL y Redis corriendo
- [ ] Migraciones aplicadas (`prisma migrate deploy`)
- [ ] Seed ejecutado (usuario admin creado)
- [ ] Build exitoso (`npm run build`)
- [ ] Tests pasan (`npm run test`)
- [ ] Frontend accesible en el navegador
- [ ] Login funciona con `admin` / `Admin123!`
- [ ] Logo y tema personalizados
- [ ] Membrete PDF configurado (o usando formato estándar)
- [ ] Certificado SSL configurado (HTTPS)
- [ ] Respaldos automáticos configurados
- [ ] `CORS_ORIGIN` apunta al dominio correcto
- [ ] Contraseña del admin cambiada del valor por defecto
- [ ] API Key de OpenAI configurada (si se usa IA)
- [ ] Acceso remoto configurado (Tailscale/VPN/Tunnel)
- [ ] Espacio en disco suficiente (>50 GB recomendado)
