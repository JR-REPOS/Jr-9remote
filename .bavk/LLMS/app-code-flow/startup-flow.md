# Application Startup Flow

> **Purpose:** Detailed sequence of how 9Remote starts up in different modes.
> **Scope:** CLI entry point through to server/tunnel readiness.
> **Source:** `package/dist/cli.cjs` (main entry, `Xt()`, `Vs()`, `Zs()`)

## 1. Overview

9Remote has three startup paths, determined by CLI arguments and flags:

| Mode | Command | Entry Function | Description |
|------|--------|---------------|-------------|
| TUI Mode | `9remote` (no args) | `Xt()` | Interactive terminal UI with QR code |
| Web UI Mode | `9remote ui` | `Vs()` | Starts HTTP server, opens browser dashboard |
| Background Mode | `9remote --start` | `Zs()` or `sl()` | Headless server for tray/autostart |

## 2. TUI Mode Startup Flow

```
┌─────────────────────────────────────────────────────────────┐
│  Terminal: 9remote                                            │
│                                                             │
│  Step 1: Preparing                                          │
│  - Check cloudflared binary exists                          │
│  - Verify dependencies                                      │
│                                                             │
│  Step 2: Connecting                                         │
│  - Generate permanent key (machine ID + salt)              │
│  - POST to https://9remote.cc/api/session/create           │
│  - Verify session created                                  │
│                                                             │
│  Step 3: Tunneling                                          │
│  - Spawn cloudflared tunnel                                │
│  - Wait for tunnel URL (90s timeout)                       │
│                                                             │
│  Step 4: Verifying                                          │
│  - HTTP health check on tunnel URL                         │
│                                                             │
│  Step 5: Ready                                              │
│  - Display QR code + one-time key                          │
│  - Wait for device connection                              │
│  - Show interactive menu                                   │
│                                                             │
│  Menu options:                                              │
│  [K] Keys      [D] Desktop     [V] Devices                 │
│  [S] Settings  [L] Logs        [U] Update                  │
└─────────────────────────────────────────────────────────────┘
```

### Detailed Sequence

```mermaid
sequenceDiagram
    participant User as User Terminal
    participant CLI as cli.cjs (Xt)
    participant Tunnel as cloudflared
    participant Worker as 9remote.cc Worker
    participant Server as server.cjs

    User->>CLI: Run `9remote`
    CLI->>CLI: Read/write log header
    CLI->>CLI: Initialize logging (agent.log)

    Note over CLI: Step PREPARING (1)
    CLI->>CLI: Check cloudflared exists (kill stale)
    CLI->>CLI: Show spinner: "Preparing"

    Note over CLI: Step CONNECTING (2)
    CLI->>CLI: Generate permanent key (xt function)
    CLI->>Worker: POST /api/session/create {apiKey}
    Worker-->>CLI: 200 {ok: true}
    CLI->>CLI: Show spinner: "Connecting"

    Note over CLI: Step TUNNELING (3)
    CLI->>Tunnel: Spawn cloudflared (Fe function)
    Tunnel-->>CLI: Tunnel URL (stdout)
    CLI->>CLI: Show spinner: "Starting tunnel"

    Note over CLI: Step VERIFYING (4)
    CLI->>Tunnel: HTTP health check on tunnel URL
    Note over CLI: 90s timeout, retries

    Note over CLI: Step READY (5)
    CLI->>CLI: Generate one-time key
    CLI->>CLI: Display QR code + keys
    CLI->>CLI: Show TUI menu
    CLI->>CLI: Start command poll (st function)
    CLI->>CLI: Wait for keypress (menu navigation)
```

## 3. Web UI Mode Startup Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│  Terminal: 9remote ui                                               │
│                                                                     │
│  1. Generate permanent key                                          │
│  2. Start HTTP server (port 2208)                                   │
│  3. Load persisted state (state.json)                              │
│  4. Start system tray (systray2)                                    │
│  5. Show notification: "9Remote is running"                        │
│  6. Open browser to http://localhost:2208                          │
│  7. Display QR code in browser                                     │
│  8. Wait for --start flag or exit                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Detailed Sequence

```mermaid
sequenceDiagram
    participant User as User
    participant CLI as cli.cjs (Vs)
    participant Server as server.cjs
    participant Tunnel as cloudflared
    participant Worker as 9remote.cc Worker
    participant Tray as System Tray
    participant Browser as Web Browser

    User->>CLI: Run `9remote ui`
    CLI->>CLI: Write agent PID file
    CLI->>CLI: Enable autostart (li function)
    CLI->>CLI: Generate permanent key (qe function)
    CLI->>CLI: Load persisted state from state.json
    CLI->>CLI: Start command poll (st function)

    Note over CLI: Tunnel check
    CLI->>Server: Check /api/health
    alt Server not running
        CLI->>Server: Start HTTP server (tt function)
        CLI->>Server: Wait for health (5s timeout)
    end

    CLI->>Tunnel: Start/verify tunnel (Gs function)
    Tunnel-->>CLI: Tunnel URL
    CLI->>Worker: POST /api/session/create
    Worker-->>CLI: Session OK

    Note over CLI: System Tray
    CLI->>Tray: Initialize systray2
    Tray-->>CLI: Tray ready

    CLI->>CLI: Show push notification
    CLI->>Browser: Open http://localhost:2208
    Browser->>Server: GET / (serves UI)
    Browser->>Server: GET /api/ui/events (SSE)
    Browser->>Server: WebSocket (Socket.IO)
    CLI->>CLI: Wait indefinitely (new Promise)
```

## 4. Background Mode Startup Flow (`--start` flag)

Background mode is used for autostart and tray-launched operation.

```mermaid
sequenceDiagram
    participant Launcher as Tray/Autostart
    participant CLI as cli.cjs (Zs)
    participant Server as server.cjs
    participant Tunnel as cloudflared
    participant Worker as 9remote.cc Worker

    Launcher->>CLI: Run `9remote --start`
    CLI->>CLI: Write agent PID file
    CLI->>CLI: Generate permanent key
    CLI->>Server: Start HTTP server
    CLI->>Server: Check /api/health
    CLI->>Tunnel: Start tunnel
    Tunnel-->>CLI: Tunnel URL
    CLI->>Worker: POST /api/session/create
    CLI->>CLI: Enter wait loop (new Promise)
```

## 5. First Run Detection

On **every** startup, the CLI:

1. **Checks for existing state:** Reads `~/.9remote/keys.json`
   - If exists and valid → load permanent key
   - If missing → generate new permanent key from machine ID

2. **Checks for cloudflared:** Verifies `cloudflared` binary is available
   - Auto-downloads on first run if missing (handled by `In()` function)
   - Verifies SHA256 checksum of downloaded binary

3. **Checks for systray2:** Post-install script (`install.cjs`) installs systray2
   - Installed in `~/.9remote/runtime/node_modules/systray2/`
   - If missing, disabled gracefully (tray not available)

4. **Checks for robotjs:** Verifies remote desktop support
   - If `@hurdlegroup/robotjs` is installed → desktop available
   - If missing → remote desktop feature disabled

## 6. Port Configuration

| Component | Default Port | Override |
|-----------|-------------|----------|
| HTTP Server | 2208 | `PORT` env var |
| Vite Dev Server | 5173 | `vite.config.js` |
| PtyDaemon socket | (Unix socket) | N/A |

## 7. Error Paths

### Server not running at startup
- In UI mode, CLI starts the server if `/api/health` fails
- Background mode waits up to 15s for server to be ready

### cloudflared not found
- CLI attempts to download and cache the binary
- If download fails, QR is shown with tunnel URL as empty (RTC-only mode)
- Background retry every ~2 minutes with exponential backoff

### Session creation fails
- CLI retries up to 3 times with exponential backoff (base 1s, max 60s)
- After failure, shows error and exits TUI

### Port already in use
- Error message: "Port 2208 already in use"
- Suggested fix: `pkill -f 9remote` or use `PORT=3308 9remote`
