# Poktapok Studio - VibeCoding Bootcamp Architecture

## 📊 Diagramas

Los diagramas están en formato `.drawio` (editable en [app.diagrams.net](https://app.diagrams.net)):

- **[main-architecture.drawio](./diagrams/main-architecture.drawio)** - Arquitectura principal del sistema
- **[user-flow.drawio](./diagrams/user-flow.drawio)** - Flujo completo del usuario

> 💡 Para ver los diagramas: descarga el archivo y ábrelo en [app.diagrams.net](https://app.diagrams.net) o en la app de escritorio de draw.io

## Overview

Poktapok Studio es una plataforma de "AI-powered vibe coding" donde estudiantes sin experiencia pueden crear proyectos web completos conversando con un agente de IA.

## Diagrama de Flujo

```
┌─────────────────────────────────────────────────────────────────────┐
│                         POKTAPOK STUDIO                              │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Frontend (Next.js + Tailwind)                                       │
│  - Chat interface for students                                       │
│  - Preview iframe                                                    │
│  - Deploy button                                                     │
│  URL: frutero.club/bootcamp/studio                                   │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼ POST /api/studio/session
┌─────────────────────────────────────────────────────────────────────┐
│  API Route (Next.js)                                                 │
│  - Receives student messages                                         │
│  - Assigns unique port per student (hash-based)                      │
│  - Forwards to OpenClaw Gateway                                      │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼ POST /tools/invoke
┌─────────────────────────────────────────────────────────────────────┐
│  OpenClaw Gateway (ngrok tunnel)                                     │
│  - Authenticates requests                                            │
│  - Routes to appropriate session                                     │
│  URL: nonvenously-unvoluble-yolando.ngrok-free.dev                   │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼ sessions_spawn / sessions_send
┌─────────────────────────────────────────────────────────────────────┐
│  Subagent Session                                                    │
│  - One per student (label: studio-{visitorId})                       │
│  - Has coding agent capabilities                                     │
│  - Creates projects, runs bun/git, creates tunnels                   │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
            ┌───────────┐  ┌───────────┐  ┌───────────┐
            │  GitHub   │  │  Preview  │  │  Convex   │
            │           │  │ (tunnel)  │  │    DB     │
            │Scarfdrilo/│  │cloudflare │  │           │
            │  <repo>   │  │  :3000-   │  │ projects  │
            │           │  │   3008    │  │ sessions  │
            └───────────┘  └───────────┘  └───────────┘
```

## Componentes

### 1. Frontend (Next.js)

**Ubicación:** `/src/app/bootcamp/studio/`

- `studio-layout.tsx` - Layout principal con chat y preview
- Chat component para interactuar con el agente
- Preview iframe para mostrar el proyecto en vivo
- Botón "Ver mi deploy" para publicar en Vercel

### 2. API Route

**Ubicación:** `/src/app/api/studio/session/route.ts`

**Acciones:**
- `start` - Spawn nueva sesión de subagent
- `send` - Enviar mensaje al agente
- `history` - Obtener historial de conversación

**Port Assignment:**
```typescript
// Cada estudiante obtiene un puerto único (3000-3008)
const portOffset = Math.abs(hashCode(visitorId)) % 9;
const userPort = 3000 + portOffset;
```

### 3. OpenClaw Gateway

**Exposición:** ngrok tunnel para acceso desde Vercel
**Autenticación:** Bearer token

**Herramientas usadas:**
- `sessions_spawn` - Crear nueva sesión con task específico
- `sessions_send` - Enviar mensaje a sesión existente
- `sessions_history` - Obtener historial

### 4. Subagent Session

Cada estudiante tiene su propia sesión aislada con el prompt:
- Crear proyecto Next.js con bun
- Subir a GitHub
- Crear tunnel con cloudflared
- Dar feedback con links explícitos
- Limitar a UN proyecto por estudiante

### 5. Convex (Database)

**Tablas:**
- `studioProjects` - Proyectos deployados
- `studioSessions` - Sesiones activas
- `studioChatHistory` - Historial de chat persistido

## Flujo de Usuario

```
1. Estudiante entra a frutero.club/bootcamp/studio
2. Se genera visitorId único
3. Escribe: "Quiero crear una app de tareas"
4. Frontend → POST /api/studio/session {action: "send", message: "..."}
5. API → OpenClaw Gateway (sessions_send con label)
6. Gateway → Spawn/find subagent session
7. Subagent ejecuta:
   - bunx create-next-app@latest tareas-app
   - git init && gh repo create
   - bun run dev --port 3005
   - cloudflared tunnel
8. Subagent responde con links
9. Estudiante ve preview en iframe
10. Estudiante puede pedir cambios
11. Al terminar, puede hacer deploy a Vercel
```

## Configuración Requerida

### Variables de Entorno (Vercel)
```
OPENCLAW_GATEWAY_URL=https://nonvenously-unvoluble-yolando.ngrok-free.dev
OPENCLAW_GATEWAY_TOKEN=<token>
```

### OpenClaw (AWS)
- Gateway corriendo en `screen` session
- ngrok tunnel activo
- Acceso a GitHub CLI autenticado

## Limitaciones Actuales

1. **Un proyecto por estudiante** - Enfocado en iterar sobre uno
2. **9 puertos máximo** - Limita estudiantes concurrentes con preview
3. **Tunnels temporales** - cloudflare tunnels expiran
4. **Sin autenticación de estudiantes** - Basado en visitorId del browser

## Repos Relacionados

- **Poktapok:** https://github.com/Scarfdrilo/poktapok (privado)
- **Proyectos de estudiantes:** https://github.com/Scarfdrilo/<nombre-proyecto>
