# 🏥 Sistema de Gestión Clínica — Historial Clínico

Sistema de Gestión Clínica de nivel premium (Expediente Médico Electrónico + Gestión de Práctica Médica/Estética). Aplicación web desplegada On-Premise con acceso remoto seguro, optimizada como PWA para iPad, smartphones y desktop.

## ✨ Características Principales

- **Expediente Clínico Electrónico** — Perfil completo del paciente, antecedentes, alergias, cirugías previas
- **Expediente Digital** — Subida y consulta de estudios en PDF con vista previa
- **Galería Multimedia** — Fotos y videos con timeline cronológica y comparador antes/después
- **Notas Clínicas** — Notas de evolución con historial completo
- **Prescripciones** — Generación y envío automático al asistente con notificación en tiempo real
- **Dashboard Doctor** — Citas del día, próximo paciente, búsqueda global
- **Dashboard Asistente** — Citas con materiales, bandeja de prescripciones, impresión con membrete
- **Modo Kiosco iPad** — Registro de pacientes con firma digital
- **Resumen Clínico IA** — Generación automática de resúmenes usando OpenAI
- **Generación PDF** — Prescripciones con membrete personalizable de la clínica
- **Recordatorios WhatsApp** — Enlace wa.me con mensaje pre-llenado
- **Respaldos Automatizados** — Cron mensual con encriptación AES-256
- **Tema White-Label** — Logo y colores personalizables por clínica (hot-reload)
- **PWA Offline** — Consulta en modo lectura sin conectividad
- **Auditoría Inmutable** — Registro de todas las acciones (NOM-004-SSA3-2012)
- **Modo Oscuro/Claro** — Con persistencia de preferencia del usuario

## 🛠️ Stack Tecnológico

| Capa | Tecnología |
|------|-----------|
| Frontend | React 18, Vite, TypeScript, Tailwind CSS, Framer Motion, Zustand |
| Backend | Node.js, NestJS, TypeScript |
| Base de Datos | PostgreSQL 16 |
| Cache/Sesiones | Redis 7 |
| ORM | Prisma |
| Autenticación | JWT (RS256) con refresh tokens |
| PDF | pdf-lib + sharp |
| IA | OpenAI API (GPT-4o-mini, configurable) |
| PWA | Workbox |
| Testing | Vitest + fast-check (Property-Based Testing) |
| Monorepo | Turborepo + npm workspaces |
| Contenedores | Docker Compose |

## 📁 Estructura del Proyecto

```
historial_clinico/
├── apps/
│   ├── api/                    # Backend NestJS
│   │   └── src/
│   │       ├── modules/        # Módulos por dominio (auth, patient, file, etc.)
│   │       ├── common/         # Guards, decoradores, interceptors
│   │       └── database/       # Prisma service
│   └── web/                    # Frontend React PWA
│       └── src/
│           ├── modules/        # Módulos UI (auth, patient, dashboard, kiosk)
│           ├── shared/         # Componentes compartidos
│           ├── stores/         # Estado global (Zustand)
│           └── services/       # Capa de API client
├── packages/
│   ├── shared-types/           # Interfaces TypeScript compartidas
│   ├── validators/             # Schemas Zod (frontend + backend)
│   └── constants/              # Límites, formatos, roles
├── prisma/
│   ├── schema.prisma           # Modelo de datos
│   ├── migrations/             # Migraciones de BD
│   └── seed.ts                 # Datos iniciales
├── docker-compose.yml          # PostgreSQL + Redis + API + Nginx
├── turbo.json                  # Configuración de Turborepo
└── .env.example                # Variables de entorno de referencia
```

## 👥 Roles del Sistema

| Rol | Acceso |
|-----|--------|
| **Doctor** | Pacientes (lectura/escritura), Dashboard Doctor, Motor IA |
| **Asistente** | Dashboard Asistente, Pacientes (solo lectura), Bandeja prescripciones |
| **Administrador** | Todos los módulos, configuración white-label, auditoría, respaldos |
| **Kiosco** | Solo módulo de registro de pacientes |

## 🚀 Inicio Rápido

```bash
# 1. Clonar el repositorio
git clone <url-del-repo>
cd historial_clinico

# 2. Instalar dependencias
npm install

# 3. Copiar variables de entorno
copy .env.example .env

# 4. Levantar PostgreSQL y Redis con Docker
docker compose up -d postgres redis

# 5. Ejecutar migraciones y seed
npx prisma migrate dev
npx prisma db seed

# 6. Iniciar en modo desarrollo
npm run dev
```

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:3000
- **Credenciales admin por defecto**: `admin` / `Admin123!`

## 📖 Documentación Adicional

- 📋 [Guía de Configuración y Personalización](./CONFIGURACION.md) — Cómo personalizar cada sección del sistema
- 🚀 [Guía de Despliegue Paso a Paso](./DESPLIEGUE.md) — Cómo poner la app en funcionamiento en producción

## 🧪 Testing

```bash
# Ejecutar todos los tests
npm run test

# Solo tests del API
cd apps/api && npx vitest run

# Solo tests del frontend
cd apps/web && npx vitest run
```

El proyecto utiliza **Property-Based Testing** con fast-check para validar propiedades formales de correctitud (validaciones, formatos, límites, etc.).

## 📜 Cumplimiento Normativo

- **NOM-004-SSA3-2012** — Expediente clínico electrónico
- **LFPDPPP** — Ley Federal de Protección de Datos Personales en Posesión de los Particulares
- Auditoría inmutable con retención mínima de 5 años
- Consentimiento documentado con firma digital

## 📄 Licencia

Proyecto privado — Todos los derechos reservados.
