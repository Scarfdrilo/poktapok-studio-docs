# Deployment Guide - Poktapok Studio

## Requirements

### AWS EC2 (Gateway Host)
- OpenClaw installed and configured
- GitHub CLI (`gh`) authenticated
- Node.js + bun
- cloudflared installed
- ngrok installed

### Vercel (Frontend)
- Next.js project deployed
- Environment variables configured

## Setup

### 1. Expose Gateway with ngrok

```bash
# On AWS server
screen -S ngrok
ngrok http 18789
# Ctrl+A, D to detach
```

Note the generated URL: `https://xxx.ngrok-free.dev`

### 2. Configure Variables in Vercel

```
OPENCLAW_GATEWAY_URL=https://xxx.ngrok-free.dev
OPENCLAW_GATEWAY_TOKEN=<your-token>
```

Get token:
```bash
cat ~/.openclaw/config.json | grep token
```

### 3. Verify Connection

```bash
curl -X POST "https://xxx.ngrok-free.dev/tools/invoke" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -H "ngrok-skip-browser-warning: true" \
  -d '{"tool": "session_status", "args": {}}'
```

## Maintenance

### Check Active Sessions

```bash
openclaw sessions list
```

### Clean Orphan Processes

```bash
# Kill old tunnels
pkill -f "cloudflared tunnel"

# Kill old dev servers
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

### "500 Error on /api/studio/session"
- Verify ngrok is running
- Verify OPENCLAW_GATEWAY_URL in Vercel
- Verify token not expired

### "Tunnel not responding"
- Port might be occupied
- Restart cloudflared
- Verify curl to localhost:PORT

### "Repo already exists on GitHub"
- Agent should use unique name
- Delete old repo or use different name
