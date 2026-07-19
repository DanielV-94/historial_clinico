# 📋 Guía de Configuración y Personalización

Esta guía explica paso a paso cómo configurar, personalizar y llenar los datos correspondientes a cada sección del Sistema de Gestión Clínica.

---

## Tabla de Contenidos

1. [Variables de Entorno](#1-variables-de-entorno)
2. [Configuración de la Base de Datos](#2-configuración-de-la-base-de-datos)
3. [Configuración de Usuarios y Roles](#3-configuración-de-usuarios-y-roles)
4. [Personalización White-Label (Tema)](#4-personalización-white-label-tema)
5. [Configuración del Membrete PDF](#5-configuración-del-membrete-pdf)
6. [Configuración de Archivos y Almacenamiento](#6-configuración-de-archivos-y-almacenamiento)
7. [Configuración de IA (OpenAI)](#7-configuración-de-ia-openai)
8. [Configuración de Respaldos](#8-configuración-de-respaldos)
9. [Configuración de Seguridad y JWT](#9-configuración-de-seguridad-y-jwt)
10. [Aviso de Privacidad (LFPDPPP)](#10-aviso-de-privacidad-lfpdppp)
11. [Configuración del Modo Kiosco](#11-configuración-del-modo-kiosco)
12. [Datos de la Clínica](#12-datos-de-la-clínica)

---

## 1. Variables de Entorno

El archivo `.env` en la raíz del proyecto controla toda la configuración del sistema. Copia `.env.example` como base:

```bash
copy .env.example .env
```

### Descripción de cada variable:

```env
# ═══════════════════════════════════════════════════
# BASE DE DATOS
# ═══════════════════════════════════════════════════

# Credenciales de PostgreSQL
POSTGRES_USER=historial              # Usuario de la BD
POSTGRES_PASSWORD=tu_contraseña_segura   # ⚠️ CAMBIA ESTO en producción
POSTGRES_DB=historial_clinico        # Nombre de la base de datos

# URL completa de conexión (Prisma la usa directamente)
DATABASE_URL=postgresql://historial:tu_contraseña_segura@localhost:5432/historial_clinico

# ═══════════════════════════════════════════════════
# REDIS (Sesiones y Cache)
# ═══════════════════════════════════════════════════

REDIS_URL=redis://localhost:6379     # URL de conexión a Redis

# ═══════════════════════════════════════════════════
# API
# ═══════════════════════════════════════════════════

PORT=3000                            # Puerto del backend NestJS
CORS_ORIGIN=http://localhost:5173    # URL del frontend (para CORS)

# ═══════════════════════════════════════════════════
# JWT (Autenticación)
# ═══════════════════════════════════════════════════

JWT_ACCESS_SECRET=cambia-este-secreto-access     # ⚠️ GENERA un string aleatorio largo
JWT_REFRESH_SECRET=cambia-este-secreto-refresh   # ⚠️ GENERA un string aleatorio largo
JWT_ACCESS_EXPIRATION=15m            # Duración del token de acceso
JWT_REFRESH_EXPIRATION=7d            # Duración del refresh token

# ═══════════════════════════════════════════════════
# ALMACENAMIENTO DE ARCHIVOS
# ═══════════════════════════════════════════════════

UPLOAD_DIR=./uploads                 # Ruta donde se guardan los archivos
MAX_FILE_SIZE=209715200              # Tamaño máximo en bytes (200 MB)

# ═══════════════════════════════════════════════════
# OPENAI (IA - Resumen Clínico)
# ═══════════════════════════════════════════════════

OPENAI_API_KEY=sk-...                # Tu API key de OpenAI (opcional)
OPENAI_MODEL=gpt-4o-mini            # Modelo a usar

# ═══════════════════════════════════════════════════
# RESPALDOS
# ═══════════════════════════════════════════════════

BACKUP_DIR=./backups                 # Directorio de respaldos
BACKUP_ENCRYPTION_KEY=cambia-esta-clave-de-encriptacion  # ⚠️ CAMBIA ESTO
BACKUP_RETENTION=12                  # Número de respaldos a retener
```

### Generar secretos seguros

Para producción, genera secretos aleatorios:

```bash
# En PowerShell:
[System.Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }) -as [byte[]])

# O con Node.js:
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

---

## 2. Configuración de la Base de Datos

### Migración inicial

```bash
# Generar el cliente Prisma y aplicar migraciones
npx prisma migrate dev

# Cargar datos iniciales (admin, clínica demo, tema)
npx prisma db seed
```

### Datos que se crean automáticamente con el seed:

| Dato | Valor por defecto |
|------|-------------------|
| Usuario admin | `admin` / `Admin123!` |
| Clínica | "Clínica Demo" |
| Tema | Colores azules, fuente Inter |

### Explorar la base de datos visualmente

```bash
npx prisma studio
```

Esto abre una interfaz web en http://localhost:5555 donde puedes ver y editar todos los registros.

---

## 3. Configuración de Usuarios y Roles

### Crear un usuario Doctor

Desde Prisma Studio o con un script SQL:

```sql
INSERT INTO users (id, username, password_hash, role, full_name, email, is_active, failed_login_attempts, created_at, updated_at)
VALUES (
  gen_random_uuid(),
  'dr_garcia',                -- Nombre de usuario para login
  '$2b$12$...',               -- Hash bcrypt de la contraseña (usa el seed como referencia)
  'doctor',                   -- Rol: doctor, assistant, admin, kiosk
  'Dr. Carlos García López',  -- Nombre completo que aparecerá en prescripciones
  'dr.garcia@clinica.com',    -- Email
  true,                       -- Activo
  0,                          -- Intentos fallidos
  NOW(), NOW()
);
```

### Crear usuario desde la API (como admin logueado)

También puedes crear usuarios programáticamente desde el módulo de administración una vez desplegado.

### Roles disponibles:

| Rol | Qué puede hacer |
|-----|----------------|
| `doctor` | Ver/editar pacientes, crear notas, prescripciones, usar IA, ver dashboard doctor |
| `assistant` | Ver pacientes (solo lectura), bandeja prescripciones, imprimir, enviar recordatorios |
| `admin` | Todo lo anterior + configurar tema, ver auditoría, gestionar respaldos, crear usuarios |
| `kiosk` | Solo el formulario de registro de pacientes nuevos |

---

## 4. Personalización White-Label (Tema)

El tema controla la apariencia visual de toda la aplicación. Se configura desde el endpoint `PUT /api/theme/config` o editando directamente en la BD.

### Campos configurables:

```json
{
  "clinicName": "Mi Clínica Estética",
  "primaryColor": "#2563EB",
  "secondaryColor": "#1E40AF",
  "accentColor": "#3B82F6",
  "fontFamily": "Inter",
  "logoPath": "/theme/logo.png"
}
```

### Descripción de cada campo:

| Campo | Formato | Descripción |
|-------|---------|-------------|
| `clinicName` | Texto | Nombre que aparece en encabezados, PDFs y notificaciones |
| `primaryColor` | Hex `#RRGGBB` | Color principal (botones, links, barras) |
| `secondaryColor` | Hex `#RRGGBB` | Color secundario (fondos, acentos sutiles) |
| `accentColor` | Hex `#RRGGBB` | Color de acento (badges, alertas, hover) |
| `fontFamily` | Nombre CSS | Fuente tipográfica (Inter, Poppins, Roboto, etc.) |
| `logoPath` | Ruta relativa | Logo de la clínica (PNG o SVG, máximo 2 MB) |

### Cómo subir el logo:

1. Coloca tu logo en la carpeta `data/clinic-files/theme/logo.png`
2. Actualiza el campo `logoPath` con la ruta relativa: `/theme/logo.png`
3. Formatos aceptados: PNG, SVG
4. Tamaño máximo: 2 MB
5. Recomendado: fondo transparente, mínimo 200x200px

### Paleta de colores de ejemplo:

```json
// Azul profesional (por defecto)
{ "primaryColor": "#2563EB", "secondaryColor": "#1E40AF", "accentColor": "#3B82F6" }

// Verde médico
{ "primaryColor": "#059669", "secondaryColor": "#047857", "accentColor": "#10B981" }

// Morado elegante
{ "primaryColor": "#7C3AED", "secondaryColor": "#6D28D9", "accentColor": "#8B5CF6" }

// Rosa estético
{ "primaryColor": "#EC4899", "secondaryColor": "#DB2777", "accentColor": "#F472B6" }
```

### Hot-reload

Los cambios de tema se aplican automáticamente en ~30 segundos (el frontend hace polling cada 30s). No requiere reiniciar el servidor ni recargar la página.

---

## 5. Configuración del Membrete PDF

El membrete es la plantilla PDF con diseño corporativo donde se inyectan los datos de las prescripciones.

### Paso 1: Crear tu plantilla PDF

1. Diseña un PDF tamaño carta (612x792 pts) con tu membrete corporativo
2. Deja espacios en blanco donde irán los datos inyectados
3. Guárdalo en: `data/clinic-files/templates/letterhead.pdf`

### Paso 2: Configurar las coordenadas

Edita la configuración `letterheadConfig` de la clínica en la BD:

```json
{
  "pageSize": "letter",
  "margins": { "top": 80, "right": 40, "bottom": 60, "left": 40 },
  "fields": {
    "patientName": {
      "x": 120,       
      "y": 680,       
      "fontSize": 12,
      "font": "Helvetica-Bold"
    },
    "date": {
      "x": 420,
      "y": 680,
      "fontSize": 10,
      "font": "Helvetica"
    },
    "content": {
      "x": 40,
      "y": 620,
      "fontSize": 11,
      "font": "Helvetica",
      "maxWidth": 520,
      "lineHeight": 14
    },
    "doctorSignature": {
      "x": 200,
      "y": 100,
      "fontSize": 10,
      "font": "Helvetica"
    },
    "footer": {
      "x": 40,
      "y": 40,
      "fontSize": 8,
      "font": "Helvetica"
    }
  },
  "templatePdfPath": "/templates/letterhead.pdf"
}
```

### Referencia de coordenadas:

- **x** = distancia desde el borde izquierdo (en puntos PDF, 1 punto = 1/72 pulgada)
- **y** = distancia desde el borde inferior (y=0 es la parte de abajo)
- El punto (0,0) está en la esquina inferior izquierda
- Tamaño carta: 612 pts de ancho × 792 pts de alto

### Sin membrete (fallback)

Si no configuras un membrete, el sistema genera un formato estándar limpio con el nombre de la clínica en el encabezado.

---

## 6. Configuración de Archivos y Almacenamiento

### Estructura de carpetas

Los archivos se almacenan localmente en el servidor:

```
{UPLOAD_DIR}/
├── patients/
│   └── {patient-uuid}/
│       ├── documents/       # PDFs de estudios
│       ├── gallery/
│       │   ├── images/      # Fotos (con watermark si aplica)
│       │   └── videos/      # Videos
│       ├── signatures/      # Firma digital
│       └── profile/         # Foto de perfil
├── templates/
│   └── letterhead.pdf       # Membrete de la clínica
├── backups/                 # Respaldos encriptados
└── theme/
    └── logo.png             # Logo white-label
```

### Límites de tamaño por tipo:

| Tipo | Formatos | Tamaño máximo |
|------|----------|---------------|
| PDF | application/pdf | 20 MB |
| Imagen | JPEG, PNG, HEIC | 50 MB |
| Video | MP4, MOV | 200 MB |
| Logo | PNG, SVG | 2 MB |
| Foto de perfil | JPEG, PNG | 5 MB |

### Marca de agua (Watermark)

Se aplica automáticamente sobre fotos y videos si está habilitada. Incluye:
- Logo de la clínica (esquina)
- ID del paciente (texto semi-transparente)

Para habilitar/deshabilitar, configura el campo correspondiente en la clínica.

---

## 7. Configuración de IA (OpenAI)

El Motor IA genera resúmenes clínicos automatizados del historial del paciente.

### Paso 1: Obtener API Key

1. Ve a https://platform.openai.com/api-keys
2. Crea una nueva API key
3. Cópiala en tu `.env`:

```env
OPENAI_API_KEY=sk-proj-tu-api-key-aqui
OPENAI_MODEL=gpt-4o-mini
```

### Paso 2: Verificar funcionamiento

El resumen IA solo funciona cuando:
- El paciente tiene al menos 1 nota de evolución
- La API key es válida
- Hay conectividad a internet

### Sin API Key (modo offline)

Si no configuras la API key de OpenAI:
- El botón "Generar Resumen Clínico" mostrará un mensaje indicando que el servicio no está disponible
- El sistema funciona completamente sin esta función (es opcional)
- Se usa un fallback local si el LLM no responde

### Timeouts y reintentos

- Timeout máximo: 30 segundos
- Reintentos: 2 (con backoff exponencial: 2s, 4s)
- Si falla, se notifica al doctor con opción de reintentar

---

## 8. Configuración de Respaldos

### Ejecución automática

Los respaldos se ejecutan automáticamente:
- **Cuándo**: Día 1 de cada mes a las 02:00 AM (hora del servidor)
- **Qué respalda**: Dump completo de PostgreSQL + carpeta de archivos multimedia
- **Formato**: tar.gz comprimido + encriptado con AES-256-CBC
- **Retención**: Últimos 12 respaldos (se eliminan los más antiguos)

### Variables de configuración:

```env
BACKUP_DIR=./backups                              # Dónde se guardan
BACKUP_ENCRYPTION_KEY=una-clave-muy-segura-de-32-caracteres  # Clave de encriptación
BACKUP_RETENTION=12                               # Cuántos respaldos retener
```

### Ejecutar respaldo manual

```bash
# Desde la API (como admin):
curl -X POST http://localhost:3000/api/backups/trigger \
  -H "Authorization: Bearer <tu-token>"
```

### Restaurar un respaldo

1. Desencriptar el archivo con la misma clave
2. Descomprimir el tar.gz
3. Restaurar el dump de PostgreSQL con `pg_restore`
4. Copiar la carpeta de archivos multimedia

---

## 9. Configuración de Seguridad y JWT

### Contraseñas

Requisitos mínimos para todas las contraseñas:
- Mínimo 8 caracteres
- Al menos 1 letra mayúscula
- Al menos 1 letra minúscula
- Al menos 1 número

### Sesiones

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| Token de acceso | 15 min | Se renueva automáticamente |
| Refresh token | 7 días | Almacenado en Redis |
| Inactividad | 15 min | Cierre automático de sesión |
| Intentos fallidos | 5 | Máximo antes de bloquear cuenta |
| Bloqueo temporal | 15 min | Duración del bloqueo |

### CORS

```env
# Producción — solo tu dominio:
CORS_ORIGIN=https://tu-dominio.com

# Desarrollo — frontend local:
CORS_ORIGIN=http://localhost:5173
```

---

## 10. Aviso de Privacidad (LFPDPPP)

El aviso de privacidad se muestra en el modo kiosco cuando el paciente se registra.

### Configurar el texto:

Edita el campo `privacyNotice` de la tabla `clinics` en la BD:

```json
{
  "title": "Aviso de Privacidad Integral",
  "content": "En cumplimiento con la Ley Federal de Protección de Datos Personales en Posesión de los Particulares...\n\nSu contenido legal completo aquí...\n\nDerechos ARCO: Puede ejercer sus derechos de Acceso, Rectificación, Cancelación y Oposición contactándonos en...",
  "version": "2.0",
  "lastUpdated": "2024-01-15T00:00:00Z"
}
```

### Información requerida por LFPDPPP:

1. Identidad del responsable (nombre de la clínica y dirección)
2. Datos personales que se recaban
3. Finalidades del tratamiento de datos
4. Mecanismos para ejercer derechos ARCO
5. Transferencias de datos (si las hay)
6. Procedimiento para notificar cambios al aviso

---

## 11. Configuración del Modo Kiosco

El modo kiosco es una interfaz tipo wizard para iPads donde los pacientes se registran solos.

### Configurar el iPad:

1. Abre Safari en el iPad
2. Navega a `https://tu-dominio.com/kiosk`
3. Toca "Compartir" → "Añadir a pantalla de inicio"
4. La app se ejecutará en modo standalone (sin barra de Safari)

### Meta tags Apple (ya incluidos):

```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```

### Timeout de inactividad

Si el paciente no interactúa durante 3 minutos, el kiosco:
1. Descarta todos los datos ingresados
2. Regresa a la pantalla de bienvenida
3. Queda listo para el siguiente paciente

---

## 12. Datos de la Clínica

### Campos principales de la clínica:

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| `name` | Nombre oficial | "Clínica Estética Dermaplus" |
| `address` | Dirección completa | "Av. Reforma 123, Col. Centro, CDMX" |
| `phone` | Teléfono principal | "5551234567" |
| `privacyNotice` | Aviso de privacidad (JSON) | Ver sección 10 |
| `letterheadConfig` | Coordenadas del membrete PDF (JSON) | Ver sección 5 |

### Actualizar desde Prisma Studio:

```bash
npx prisma studio
```

Navega a la tabla `Clinic` y edita los campos directamente.

---

## 🔧 Resumen de Archivos a Personalizar

| Qué personalizar | Dónde |
|-----------------|-------|
| Variables generales | `.env` |
| Logo de la clínica | `data/clinic-files/theme/logo.png` |
| Membrete PDF | `data/clinic-files/templates/letterhead.pdf` |
| Colores y fuente | `PUT /api/theme/config` o tabla `theme_configs` |
| Nombre de la clínica | Tabla `clinics` |
| Aviso de privacidad | Tabla `clinics.privacyNotice` |
| Coordenadas del membrete | Tabla `clinics.letterheadConfig` |
| Usuarios del sistema | Tabla `users` |
| Clave de encriptación | `.env` → `BACKUP_ENCRYPTION_KEY` |
| API Key de IA | `.env` → `OPENAI_API_KEY` |
