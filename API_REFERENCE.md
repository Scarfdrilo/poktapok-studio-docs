# API Reference - Poktapok Studio

## Endpoint

```
POST /api/studio/session
GET  /api/studio/session?action=history&visitorId=xxx
```

## POST Actions

### Start Session

Starts a new development session.

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

Sends a message to the agent.

```json
{
  "action": "send",
  "visitorId": "k977cdbtxm99ajdbcxg462g6t180zqh1",
  "message": "I want to create a tasks app"
}
```

**Response:**
```json
{
  "ok": true,
  "reply": "Perfect! Let's create your tasks app...",
  "tunnelUrl": "https://xxx.trycloudflare.com",
  "repoUrl": "https://github.com/Scarfdrilo/tasks-app"
}
```

## GET Actions

### Get History

Gets conversation history.

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
      "content": "I want to create a tasks app"
    },
    {
      "role": "assistant", 
      "content": "Perfect! Let's create your app..."
    }
  ]
}
```

## OpenClaw Gateway Integration

The API route communicates with OpenClaw Gateway via `/tools/invoke`:

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
