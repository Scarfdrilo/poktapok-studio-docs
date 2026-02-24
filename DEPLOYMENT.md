# Deployment Guide - Poktapok Studio

## Requisitos

### AWS EC2 (Gateway Host)
- OpenClaw instalado y configurado
- GitHub CLI (`gh`) autenticado
- Node.js + bun
- cloudflared instalado
- ngrok instalado

### Vercel (Frontend)
- Proyecto Next.js deployado
- Variables de entorno configuradas

## Setup

### 1. Exponer Gateway con ngrok

```bash
# En el servidor AWS
screen -S ngrok
ngrok http 18789
# Ctrl+A, D para detach
```

Toma nota del URL generado: `https://xxx.ngrok-free.dev`

### 2. Configurar Variables en Vercel

```
OPENCLAW_GATEWAY_URL=https://xxx.ngrok-free.dev
OPENCLAW_GATEWAY_TOKEN=<tu-token>
```

Obtener token:
```bash
cat ~/.openclaw/config.json | grep token
```

### 3. Verificar Conexión

```bash
curl -X POST "https://xxx.ngrok-free.dev/tools/invoke" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -H "ngrok-skip-browser-warning: true" \
  -d '{"tool": "session_status", "args": {}}'
```

## Mantenimiento

### Revisar Sesiones Activas

```bash
openclaw sessions list
```

### Limpiar Procesos Huérfanos

```bash
# Matar tunnels viejos
pkill -f "cloudflared tunnel"

# Matar servidores de desarrollo viejos
pkill -f "bun run dev"
```

### Logs

```bash
# Gateway logs
journalctl -u openclaw -f

# Tunnel logs
cat /tmp/tunnel-*.log
```

## Troubleshooting

### "500 Error en /api/studio/session"
- Verificar que ngrok está corriendo
- Verificar OPENCLAW_GATEWAY_URL en Vercel
- Verificar token no expirado

### "Tunnel no responde"
- Puerto puede estar ocupado
- Reiniciar cloudflared
- Verificar curl al localhost:PORT

### "Repo ya existe en GitHub"
- El agente debe usar nombre único
- Borrar repo viejo o usar otro nombre
