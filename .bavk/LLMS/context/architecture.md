# System Architecture

> **Purpose:** High-level architecture overview of the 9Remote system.
> **Scope:** All major components and their interactions.
> **Source:** `README.md`, `package/dist/cli.cjs`, `package/dist/server.cjs`, `package/dist/ptyDaemon.cjs`

## 1. Overview

9Remote is an all-in-one remote access tool that provides **terminal access**, **remote desktop**, **file explorer**, **code editor**, and **Git integration** through a single CLI. It uses **Cloudflare Tunnel** for zero-config networking (no port forwarding needed) and **Socket.IO** for real-time terminal I/O.

The system is designed around three core processes that communicate via different channels:

1. **CLI** — User-facing entry point (TUI or Web UI mode)
2. **Server** — HTTP + Socket.IO server, tunnel management, desktop capture
3. **PtyDaemon** — Persistent PTY daemon (survives server restarts)

## 2. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        USER'S MACHINE                                    │
│                                                                         │
│  ┌──────────────┐       ┌──────────────────────────┐                   │
│  │  CLI (TUI)   │──────►│  Server (HTTP + Socket.IO)│                   │
│  │  or `ui`     │       │  Port 2208                 │                   │
│  └──────────────┘       └──────┬──────────┬─────────┘                   │
│                               │ Socket.IO│ Unix Socket                  │
│  ┌────────────────────────┐  │          │                              │
│  │  Web UI (Preact)        │◄─┼─ SSE     │                              │
│  │  Embedded dashboard     │  │          │                              │
│  └────────────────────────┘  │          │                              │
│                               │          ▼                              │
│  ┌────────────────────────┐  │  ┌──────────────┐                        │
│  │ Cloudflare Tunnel        │  │  │  PtyDaemon   │                        │
│  │ (cloudflared)            │  │  │  (Persistent)│                        │
│  │ Outbound only            │  │  └──────┬───────┘                        │
│  └──────────┬───────────────┘  │         │ node-pty                       │
│             │                   │         ▼                               │
│             │  ┌────────────────┐│  ┌──────────┐                         │
│             │  │ WebRTC Desktop ││  │  Shell    │                         │
│             │  │(node-datachannel││  │ (bash/   │                         │
│             │  │ + robotjs)     ││  │ zsh/...)  │                         │
│             │  └────────────────┘│  └──────────┘                         │
│             ▼                      │                                     │
│  ┌──────────────────────┐         │                                     │
│  │  Internet             │         │                                     │
│  └──────────┬───────────┘         │                                     │
│             ▼                      │                                     │
│  ┌──────────────────────┐         │                                     │
│  │  9remote.cc Worker   │         │                                     │
│  │  (Cloudflare)        │         │                                     │
│  │  /api/session/create │         │                                     │
│  └──────────────────────┘         │                                     │
└─────────────────────────────────────────────────────────────────────────┘
         │                                                                 │
         │  HTTPS (Cloudflare Tunnel)                            ┌─────────┴──────────┐
         └────────────────────────────────────────────┬─────────│  PHONE / BROWSER  │
                                                        │       └────────────────────┘
                                                        ▼
                                              ┌──────────────────┐
                                              │ Web Client UI    │
                                              │ (React/Next.js)  │
                                              │ 9remote.cc       │
                                              └──────────────────┘
```

## 3. Component Responsibilities

### 3.1 CLI (`cli.cjs`)

The CLI is the entry point (`package.json` bin: `./dist/cli.cjs`). It supports two modes:

- **Default (TUI mode):** Renders an interactive terminal UI with QR code, status spinner, and menu navigation. Manages the full lifecycle: tunnel spawning, session creation, key generation, upgrade checks.

- **`ui` mode:** Starts the HTTP server in the foreground. Opens the embedded web UI in the default browser. Uses system tray (systray2) for background management on macOS/Windows/Linux.

**Key functions:**
- `Xt()` — Main TUI entry point
- `Vs()` — UI mode server entry point
- `Zs()` — Background mode server entry point (for `--start` flag)
- `sl(e)` — Server mode tunnel startup

### 3.2 Server (`server.cjs`)

The server provides:
1. **HTTP server** (port 2208) serving the embedded Preact dashboard
2. **Router** with access control (localhost or Cloudflare tunnel only)
3. **API endpoints** for state management (see `api/api-endpoints.md`)
4. **SSE stream** for real-time state `/api/ui/events`
5. **Socket.IO** server for terminal I/O
6. **Cloudflare tunnel management** — spawns `cloudflared`, monitors health, handles URL rotation/restart
7. **PtyDaemon client** — forwards Socket.IO events to the daemon via Unix socket
8. **Desktop capture** — WebRTC streaming via `node-datachannel` + screen capture via `node-screenshots`
9. **System tray** — systray2 integration for background operation
10. **Sleep inhibition** — prevents system sleep during active sessions
11. **Autostart** — registers with OS autostart (LaunchAgent/systemd/VBS)
12. **Push notifications** — web-push for build/deploy alerts
13. **Clipboard integration** — cross-platform clipboard access
14. **Local site proxy** — proxies localhost dev servers through the tunnel

### 3.3 PtyDaemon (`ptyDaemon.cjs`)

A persistent daemon process that:
- Runs independently of the main server
- Manages PTY sessions via `node-pty`
- Survives server restarts (sessions persist)
- Communicates with server via Unix socket (`~/.9remote/pty-daemon.sock`)
- Uses a custom JSON-over-socket protocol (version "46")
- Handles session lifecycle: create, join, delete, resize, input

### 3.4 Web UI (`ui/`)

A Preact single-page application (SPA) served by the server:
- **Dashboard** — shows QR code, tunnel status, device management
- **Terminal** — xterm.js-based terminal emulator via Socket.IO
- **File Explorer** — browse, upload, download files
- **Code Editor** — syntax-highlighted editor
- **Git Integration** — visual git status, commit/push
- **Remote Desktop** — WebRTC screen streaming viewer
- **Settings** — theme, autostart, sleep inhibition, permissions

## 4. Communication Channels

| Channel | Type | Purpose |
|---|---|---|
| CLI ↔ Server | Unix Socket / HTTP | TUI control commands |
| Server ↔ Web UI | HTTP + SSE | State management, real-time updates |
| Web UI ↔ Server | Socket.IO (WebSocket) | Terminal I/O, desktop streaming |
| Server ↔ PtyDaemon | Unix Socket (JSON) | PTY session management |
| Server ↔ cloudflared | stdio pipes | Tunnel process management |
| Server ↔ 9remote.cc Worker | HTTPS | Session coordination |
| Web UI ↔ Phone Browser | WebRTC | Direct desktop streaming |

## 5. Key Design Principles

1. **Zero-configuration:** Cloudflare Quick Tunnel eliminates port forwarding
2. **Persistent sessions:** PtyDaemon ensures PTY sessions survive restarts
3. **Local-first:** LocalFirstAdapter races LAN vs tunnel for lowest latency
4. **Security-first:** No open ports, no data collection, Pair Device approval
5. **Cross-platform:** macOS, Linux, Windows support at the host level
