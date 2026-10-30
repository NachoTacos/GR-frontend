# GR-frontend — Grupo Ricartti

Sitio público (landing, galería de proyectos, blog, formularios) y panel de administración de Grupo Ricartti, hechos con **Angular** y **Tailwind CSS**.

El sistema está repartido en **dos repositorios**:

| Repositorio | Contenido |
|---|---|
| **GR-backend** | API, base de datos, Docker Compose y documentación de arquitectura. **Contiene la guía principal de instalación.** |
| **GR-frontend** (este) | Aplicación Angular: sitio público y panel `/admin` |

> **¿Quieres levantar el sistema completo?** Sigue el [README de GR-backend](../GR-backend/README.md) (en GitHub: https://github.com/NachoTacos/GR-backend). Ahí están la instalación, las credenciales de prueba y el checklist de verificación. Este documento cubre solo el trabajo en el frontend.

---

## 1. Ubicación de los repositorios

Clona los dos repositorios **uno al lado del otro**, con estos nombres exactos:

```
grupo-ricartti/
├── GR-backend/
└── GR-frontend/     ← este repositorio
```

Docker Compose, que vive en GR-backend, construye este frontend desde `../GR-frontend`.

```bash
mkdir grupo-ricartti
cd grupo-ricartti
git clone https://github.com/NachoTacos/GR-backend.git  GR-backend
git clone https://github.com/NachoTacos/GR-frontend.git GR-frontend
```

---

## 2. Requisitos

- **Node.js 22 LTS o superior** y npm. En Windows: `winget install -e --id OpenJS.NodeJS.LTS`. En macOS: `brew install node`.
- **La API de GR-backend en marcha en `http://localhost:5080`**, de una de estas dos formas:

| Forma | Comando (en `GR-backend`) | Necesitas |
|---|---|---|
| **Con Docker** (sin instalar .NET) | `docker compose up --build db api` | Docker Desktop |
| **Local** | `dotnet run --project src/Ricartti.Api` | .NET 10 SDK y PostgreSQL ([ver guía](../GR-backend/README.md#5-opción-b--instalación-local-sin-docker)) |

En los dos casos, antes debes crear `GR-backend/.env` a partir de `.env.example`, como explica la guía de GR-backend.

---

## 3. Ejecutar en desarrollo

Los comandos de esta guía son iguales en PowerShell, **Git Bash**, macOS y Linux.

```bash
cd GR-frontend
npm install
npm start
```

Abre http://localhost:4200. El panel está en http://localhost:4200/admin/login y las credenciales de prueba están en el [README de GR-backend](../GR-backend/README.md#3-credenciales-de-prueba).

**Cómo se conecta con la API:**

- `npm start` usa `proxy.conf.json`, que reenvía `/api` y `/uploads` a `http://localhost:5080`. Para el navegador, sitio y API comparten el mismo origen. Por eso funcionan la cookie de sesión `SameSite=Strict` y la protección XSRF de Angular, y no hace falta CORS.
- **El frontend no tiene `.env` ni secretos.** Todo lo que contiene es público y se entrega al navegador.
- Si cambias `API_PORT` en GR-backend, actualiza el destino en `proxy.conf.json`.

---

## 4. Scripts

| Comando | Qué hace |
|---|---|
| `npm start` | Servidor de desarrollo con recarga automática y proxy a la API |
| `npm run build` | Compilación de producción en `dist/` |
| `npm test` | Pruebas unitarias |

---

## 5. Docker

Este repositorio incluye:

- **`Dockerfile`**: compila la aplicación con Node y la sirve con Nginx.
- **`nginx.conf`**: sirve el build, reenvía `/api` y `/uploads` al contenedor de la API y añade las cabeceras de seguridad (`Content-Security-Policy`, `X-Content-Type-Options`, `Referrer-Policy`).

No se ejecuta por separado: lo construye `docker compose up --build` desde GR-backend. El sitio queda en http://localhost:8080.

---

## 6. Estructura

```
src/app/
├── core/          # Servicios HTTP, autenticación (guard), interceptores, modelos
├── features/
│   ├── public/    # Landing, galería, detalle de proyecto, blog, formularios
│   └── admin/     # Panel: login, menú lateral, proyectos, blog, mensajes
└── shared/ui/     # Componentes reutilizables
```

| Ruta | Vista |
|---|---|
| `/` | Landing |
| `/proyectos/:slug` | Detalle de proyecto |
| `/blog` · `/blog/:slug` | Blog |
| `/admin/login` | Inicio de sesión |
| `/admin/proyectos` · `/admin/blog` · `/admin/mensajes` | Secciones del panel |

El detalle completo (rutas, menú del panel, decisiones) está en la [arquitectura del sistema](../GR-backend/docs/ARCHITECTURE.md#13-frontend), dentro de GR-backend.

---

## 7. Reglas de seguridad del frontend

- **Prohibido `bypassSecurityTrust*`.**
- **Prohibido `[innerHTML]`**, con una única excepción: el componente `blog-content`, que muestra HTML ya saneado por la API. Angular lo sanea de nuevo.
- Toda la interpolación usa `{{ }}`, que Angular escapa automáticamente.
- Las peticiones van siempre a **URL relativas** (`/api/...`). Así Angular añade la cabecera `X-XSRF-TOKEN` y el navegador envía la cookie de sesión.
- El guard de `/admin` solo mejora la experiencia de uso: **la autorización real la hace la API**.
- Las fuentes tipográficas se alojan en el propio proyecto (`src/assets/fonts`) para cumplir la CSP.

---

## 8. Solución de problemas

| Problema | Solución |
|---|---|
| La página carga pero no hay datos (errores 504 o de proxy en la terminal) | La API no está en marcha en `:5080`. Levántala desde GR-backend (ver sección 2). |
| `npm` no se reconoce | Cierra y vuelve a abrir la terminal después de instalar Node.js. |
| Inicio sesión y el panel me devuelve al login | Revisa `AUTH_COOKIE_SECURE` en `GR-backend/.env` (ver la solución de problemas de GR-backend). |
| `docker compose` no encuentra el frontend | Este repositorio debe estar en `../GR-frontend` respecto a GR-backend, o configurado en `FRONTEND_PATH`. |
