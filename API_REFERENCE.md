# API Reference - Poktapok Studio

## Endpoint

```
POST /api/studio/session
GET  /api/studio/session?action=history&visitorId=xxx
```

## POST Actions

### Start Session

Inicia una nueva sesión de desarrollo.

```json
{
  "action": "start",
  "visitorId": "k977cdbtxm99ajdbcxg462g6t180zqh1"
}
```

**Response:**
```json
{
  "ok": true,
  "sessionKey": "agent:main:subagent:xxx",
  "message": "Session started"
}
```

### Send Message

Envía un mensaje al agente.

```json
{
  "action": "send",
  "visitorId": "k977cdbtxm99ajdbcxg462g6t180zqh1",
  "message": "Quiero crear una app de tareas"
}
```

**Response:**
```json
{
  "ok": true,
  "reply": "¡Perfecto! Vamos a crear tu app de tareas...",
  "tunnelUrl": "https://xxx.trycloudflare.com",
  "repoUrl": "https://github.com/Scarfdrilo/tareas-app"
}
```

## GET Actions

### Get History

Obtiene el historial de conversación.

```
GET /api/studio/session?action=history&visitorId=xxx
```

**Response:**
```json
{
  "ok": true,
  "messages": [
    {
      "role": "user",
      "content": "Quiero crear una app de tareas"
    },
    {
      "role": "assistant", 
      "content": "¡Perfecto! Vamos a crear tu app..."
    }
  ]
}
```

## OpenClaw Gateway Integration

El API route se comunica con OpenClaw Gateway via `/tools/invoke`:

```typescript
const response = await fetch(`${OPENCLAW_GATEWAY_URL}/tools/invoke`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${OPENCLAW_GATEWAY_TOKEN}`,
    "ngrok-skip-browser-warning": "true",
  },
  body: JSON.stringify({
    tool: "sessions_spawn", // or sessions_send
    args: { ... }
  }),
});
```

## Port Assignment Algorithm

```typescript
function hashCode(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash;
  }
  return hash;
}

// Ports 3000-3008 (9 concurrent students max)
const portOffset = Math.abs(hashCode(visitorId)) % 9;
const userPort = 3000 + portOffset;
```
