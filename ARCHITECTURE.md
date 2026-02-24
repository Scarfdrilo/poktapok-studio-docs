# Poktapok Studio - VibeCoding Bootcamp Architecture

## 📊 Diagrams

Diagrams are in `.drawio` format (editable at [app.diagrams.net](https://app.diagrams.net)):

- **[main-architecture.drawio](./diagrams/main-architecture.drawio)** - Main system architecture
- **[user-flow.drawio](./diagrams/user-flow.drawio)** - Complete user flow

> 💡 To view diagrams: download the file and open it at [app.diagrams.net](https://app.diagrams.net) or in the draw.io desktop app

## Overview

Poktapok Studio is an "AI-powered vibe coding" platform where students with no experience can create complete web projects by chatting with an AI agent.

## Flow Diagram

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

## Components

### 1. Frontend (Next.js)

**Location:** `/src/app/bootcamp/studio/`

- `studio-layout.tsx` - Main layout with chat and preview
- Chat component to interact with the agent
- Preview iframe to display the live project
- "View my deploy" button to publish on Vercel

### 2. API Route

**Location:** `/src/app/api/studio/session/route.ts`

**Actions:**
- `start` - Spawn new subagent session
- `send` - Send message to agent
- `history` - Get conversation history

**Port Assignment:**
```typescript
// Each student gets a unique port (3000-3008)
const portOffset = Math.abs(hashCode(visitorId)) % 9;
const userPort = 3000 + portOffset;
```

### 3. OpenClaw Gateway

**Exposure:** ngrok tunnel for access from Vercel
**Authentication:** Bearer token

**Tools used:**
- `sessions_spawn` - Create new session with specific task
- `sessions_send` - Send message to existing session
- `sessions_history` - Get history

### 4. Subagent Session

Each student has their own isolated session with the prompt:
- Create Next.js project with bun
- Upload to GitHub
- Create tunnel with cloudflared
- Give feedback with explicit links
- Limit to ONE project per student

### 5. Convex (Database)

**Tables:**
- `studioProjects` - Deployed projects
- `studioSessions` - Active sessions
- `studioChatHistory` - Persisted chat history

## User Flow

```
1. Student enters frutero.club/bootcamp/studio
2. Unique visitorId is generated
3. Writes: "I want to create a tasks app"
4. Frontend → POST /api/studio/session {action: "send", message: "..."}
5. API → OpenClaw Gateway (sessions_send with label)
6. Gateway → Spawn/find subagent session
7. Subagent executes:
   - bunx create-next-app@latest tasks-app
   - git init && gh repo create
   - bun run dev --port 3005
   - cloudflared tunnel
8. Subagent responds with links
9. Student sees preview in iframe
10. Student can request changes
11. When done, can deploy to Vercel
```

## Required Configuration

### Environment Variables (Vercel)
```
OPENCLAW_GATEWAY_URL=https://nonvenously-unvoluble-yolando.ngrok-free.dev
OPENCLAW_GATEWAY_TOKEN=<token>
```

### OpenClaw (AWS)
- Gateway running in `screen` session
- Active ngrok tunnel
- Authenticated GitHub CLI access

## Current Limitations

1. **One project per student** - Focused on iterating on one
2. **9 ports maximum** - Limits concurrent students with preview
3. **Temporary tunnels** - cloudflare tunnels expire
4. **No student authentication** - Based on browser visitorId

## Related Repos

- **Poktapok:** https://github.com/Scarfdrilo/poktapok (private)
- **Student projects:** https://github.com/Scarfdrilo/<project-name>
