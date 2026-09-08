# Application Components

> **Purpose:** Detailed breakdown of all components in the 9Remote system.
> **Scope:** CLI, Server, PtyDaemon, UI, and their internal modules.
> **Source:** `package/dist/cli.cjs`, `package/dist/server.cjs`, `package/dist/ptyDaemon.cjs`

## 1. Component Map

```
9Remote (npm package "9remote" v2.5.8)
│
├── CLI Entry Point (cli.cjs)
│   ├── TUI Mode (default)
│   │   └── TUI Renderer
│   │       ├── Status spinner (Preparing → Connecting → Tunneling → Ready)
│   │       ├── QR code renderer
│   │       ├── Menu system (keys, desktop, devices, settings, logs)
│   │       └── Pair Device prompt (y/n)
│   │
│   ├── UI Mode (`9remote ui`)
│   │   └── HTTP Server Launcher
│   │       ├── System Tray Manager (systray2)
│   │       ├── Cloudflare Tunnel Manager
│   │       ├── Update Checker
│   │       └── Autostart Manager
│   │
│   └── Background Mode (`--start`)
│       └── Server (detached process)
│
├── Server (server.cjs)
│   ├── HTTP Router (access control: localhost + CF-Connecting-IP)
│   ├── SSE Streamer (/api/ui/events)
│   ├── Socket.IO Server (port 2208)
│   ├── Tunnel Manager
│   │   ├── cloudflared Quick Tunnel
│   │   ├── URL rotation detection
│   │   ├── Health check (90s timeout)
│   │   ├── Auto-restart (on network change, sleep/wake)
│   │   ├── Edge probe (connection count check)
│   │   └── LocalFirstAdapter (LAN vs tunnel racing)
│   ├── PtyDaemon Client
│   │   ├── Unix Socket client
│   │   ├── Request/response protocol
│   │   ├── Auto-spawn if daemon not running
│   │   └── Version verification
│   ├── Desktop Manager
│   │   ├── WebRTC via node-datachannel
│   │   ├── Screen capture via node-screenshots
│   │   ├── Input control via @hurdlegroup/robotjs
│   │   ├── JPEG encoding via @julusian/jpeg-turbo
│   │   └── GPU acceleration (Windows OpenCL, macOS vImage)
│   ├── System Tray
│   │   ├── systray2 native binary
│   │   ├── Tray icons per platform
│   │   ├── Menu: Open Web UI, Shutdown
│   │   └── Windows PowerShell tray bridge
│   ├── Sleep Inhibitor
│   │   ├── macOS: caffeinate
│   │   ├── Linux: systemd-inhibit
│   │   └── Windows: PowerCreateRequest API
│   ├── Autostart Manager
│   │   ├── macOS: LaunchAgent plist
│   │   ├── Linux: systemd user service / XDG autostart
│   │   └── Windows: Startup VBS script
│   ├── Notification Manager
│   │   └── web-push for push notifications
│   ├── Update Manager
│   │   ├── Version check from npm registry
│   │   ├── SHA256 verification
│   │   ├── Cross-platform update scripts
│   │   └── Rollback on failure
│   ├── Clipboard Manager
│   │   ├── macOS: pbpaste/pbcopy, osascript
│   │   ├── Linux: xclip
│   │   └── Windows: PowerShell
│   ├── Permission Manager (macOS)
│   │   ├── Screen Recording detection
│   │   ├── Accessibility detection
│   │   └── Permission polling
│   ├── Log Manager
│   │   ├── File-based logging (~/.9remote/logs/agent.log)
│   │   ├── 2KB rotation, 7-day cleanup
│   │   └── In-memory tail (last 60 lines)
│   ├── State Persistence
│   │   ├── state.json (UI state, step, tunnel info)
│   │   ├── settings.json (theme, preferences)
│   │   ├── keys.json (machine ID, permanent key)
│   │   └── cmd.json (command channel file)
│   └── Local Proxy Manager
│       ├── Detects running localhost ports
│       ├── Proxies through tunnel
│       └── Path: /proxy/{port}/
│
├── PtyDaemon (ptyDaemon.cjs)
│   ├── Socket server (Unix socket / named pipe)
│   ├── PTY Session Manager
│   │   ├── Session pool (Map<string, SessionInfo>)
│   │   ├── node-pty spawn
│   │   ├── Shell detection (bash, zsh, sh, cmd, powershell, pwsh)
│   │   ├── OSC 7 CWD tracking
│   │   ├── ZSHRC injection (for OSC 7 support)
│   │   └── Prompt command injection (PowerShell)
│   ├── Command Protocol
│   │   ├── JSON-over-socket, newline-delimited
│   │   ├── Protocol version: "46"
│   │   ├── Ping/pong keepalive
│   │   └── Request/response correlation via requestId
│   ├── Output Buffer Management
│   │   ├── Buffer concatenation
│   │   ├── ANSI escape sequence filtering
│   │   ├── Flush scheduling (setImmediate)
│   │   └── Buffer size limit (truncation when > threshold)
│   └── Client Connection Manager
│       ├── Multiple clients (server connections)
│       ├── Message routing to session outputs
│       └── Graceful shutdown (SIGTERM/SIGINT)
│
└── Web UI (ui/)
    ├── index.html
    │   ├── Preact mount point
    │   ├── Theme loader (localStorage)
    │   └── Google Fonts (Sora, JetBrains Mono, Material Symbols)
    ├── assets/index-H_1CAVCP.js (bundled Preact SPA)
    │   ├── Terminal view (xterm.js via Socket.IO)
    │   ├── File Explorer (tree view, upload/download)
    │   ├── Code Editor (syntax highlighting)
    │   ├── Git panel (status, commit, push)
    │   ├── Remote Desktop viewer (WebRTC video)
    │   ├── Device management (Pair Device, approval list)
    │   ├── Settings panel (theme, autostart, sleep, permissions)
    │   ├── QR code scanner (camera or paste key)
    │   └── AI agent integration (claude, codex, gemini, opencode icons)
    └── assets/index-CcS6AXZF.css (Tailwind CSS)
```

## 2. File Layout

### npm Package Structure
```
package/
├── package.json          # Package manifest, dependencies, bin entry
├── README.md             # Package-specific README (MIT license)
└── dist/
    ├── cli.cjs            # CLI entry point (TUI + UI mode launcher)
    ├── server.cjs         # HTTP + Socket.IO server
    ├── ptyDaemon.cjs      # Persistent PTY daemon
    ├── install.cjs        # Post-install script (systray2, robotjs)
    ├── ui/
    │   ├── index.html     # Embedded dashboard HTML
    │   ├── favicon.svg
    │   ├── agents/        # AI agent icons
    │   │   ├── claude.svg
    │   │   ├── codex.svg
    │   │   ├── gemini.svg
    │   │   └── opencode.svg
    │   └── assets/
    │       ├── index-H_1CAVCP.js   # Preact SPA bundle
    │       └── index-CcS6AXZF.css  # Tailwind CSS bundle
    └── assets/
        ├── tray.ps1        # Windows tray PowerShell script
        ├── trayIcon.ico    # Windows tray icon
        └── bin/
            ├── desktop-elevate.cs   # C# elevation helper
            └── desktop-bridge.cs    # C# desktop bridge
```

### Runtime File Structure (~/.9remote/)
```
~/.9remote/
├── keys.json              # Machine ID, permanent key, device name
├── autostart.log          # Autostart-related logs
├── logs/
│   ├── agent.log          # Server logs (rotated at 2KB)
│   └── agent.log.1
├── state/
│   ├── state.json         # Current UI state (step, tunnelUrl, keys)
│   ├── settings.json      # User settings (theme, etc.)
│   └── cmd.json           # Command channel (TUI → Server)
├── pids/
│   ├── cloudflared.pid    # Cloudflare tunnel process PID
│   ├── agent.pid          # Main agent process PID
│   └── ptyDaemon.pid      # PTY daemon process PID
├── daemon/                # PTY daemon workspace
├── buffers/               # Uploaded file buffers
├── runtime/               # Runtime deps (systray2 node_modules)
└── pty-daemon.sock        # Unix socket for daemon communication
                           # (Windows: \\.\pipe\9remote-pty)
```
