# TUI Menu & Navigation

> **Purpose:** Detailed reference of the TUI menu structure and navigation.
> **Scope:** Interactive menu items, keyboard shortcuts, and their actions.
> **Source:** `package/dist/cli.cjs` (menu functions)

## 1. Overview

The TUI (Terminal User Interface) mode is the default entry point when running `9remote` without arguments. It provides an interactive, keyboard-navigable interface for managing the 9Remote agent.

## 2. TUI Layout

```
┌─────────────────────────────────────────────────────────────┐
│  🚀 9Remote v2.5.8 — Terminal in Your Pocket                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  🔑 Scan QR to connect:                                    ││
│  │                                                           ││
│  │  ┌───┬───┬───┐                                       □ ││
│  │  │ ░ │ █ │ ░ │   QR Code (auto-refreshing)            ││
│  │  ├───┼───┼───┤                                          ││
│  │  │ █ │ ░ │ █ │                                          ││
│  │  └───┴───┴───┘                                          ││
│  │                                                          ││
│  └─────────────────────────────────────────────────────────┘│
│                                                             │
│  QR expires in 30 minutes (one-time use)                   │
│  ─────────────────────────────────────────                 │
│  App URL:  https://9remote.cc/login                         │
│  One-Time Key:  sk-abcd1234-wxyz-abcdef                      │
│  Key:  sk-abcd1234-ijkl-mnopqr                               │
│  Tunnel:  https://abcd1234.trycloudflare.com                │
│  ─────────────────────────────────────────                 │
│                                                             │
│  [K] Keys   [D] Desktop   [V] Devices   [S] Settings      │
│  [L] Logs   [U] Update    [Shift+Q] Shutdown                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 3. Menu Items

The TUI menu uses single-key shortcuts for navigation:

### 3.1 Keys (`K` or `1`)

Manages pairing keys:

- **View permanent key:** Displays the machine-derived permanent API key
- **View one-time key:** Displays the current 30-minute QR key
- **Regenerate one-time key:** Generates a new temp key (old one invalidated)
- **Copy to clipboard:** Copies the selected key

**State:** Keys are read from `~/.9remote/keys.json` and SSE state.

### 3.2 Desktop (`D` or `2`)

Manages remote desktop:

- **Toggle on/off:** Enables/disables desktop streaming
- **Permission check:** On macOS, verifies Screen Recording + Accessibility permissions
- **Status:** Shows if `robotjs` is available

**macOS permissions dialog:**
```
Remote Desktop requires:
  ✓ Screen Recording
  ✓ Accessibility

Grant permissions in System Settings → Privacy & Security
```

### 3.3 Devices (`V` or `3`)

Manages connected and paired devices:

- **Connected:** Shows active Socket.IO connections (IP, device ID, connection time)
- **Paired:** Shows approved device identifiers
- **Pending:** Shows devices awaiting approval
- **Auto-approve toggle:** Automatically approve new connections
- **Revoke device:** Remove device from approved list

**Pair Device prompt (when new device connects):**
```
🔔 New Device Connection

  Device:  ABCD12...
  IP:      192.168.1.100

  Allow this device?  (y/N)
```

### 3.4 Settings (`S` or `4`)

Configuration settings:

- **Theme:** Dark / Light mode (persisted to localStorage + state.json)
- **Autostart:** Enable/disable auto-start on system boot
- **Sleep Inhibition:**
  - `Never` — System sleep is not inhibited
  - `Always` — Sleep is always prevented while 9Remote is running
  - `Active` — Sleep is prevented only during active sessions
- **Connection Mode:**
  - `Tunnel preferred` — Use Cloudflare tunnel by default
  - `Tunnel only` — Skip LAN detection, always use tunnel
  - `LAN preferred` — Try LAN first, fall back to tunnel

### 3.5 Logs (`L` or `5`)

Server log viewer:

- Shows last 200 log entries (from `~/.9remote/logs/agent.log`)
- Log levels: info, warn, error, debug
- Timestamp format: `HH:mm:ss.mmm [level] message`
- Clear button to empty log file

### 3.6 Update (`U` or `6`)

Update management:

- **Check:** Fetches latest version from npm registry
- **Download:** Downloads and verifies SHA256 checksum
- **Install:** Spawns background installer process
- **Rollback:** Automatic rollback if new version is broken

**Status indicators:**
```
Checking for updates...    (spinner)
Update available: 2.5.9    (if newer version found)
Up to date: 2.5.8          (if current is latest)
Updating...                (during install)
```

### 3.7 Shutdown (`Shift+Q` or `7`)

Graceful shutdown:

1. Stop Cloudflare tunnel
2. Clear PID files
3. Stop PtyDaemon (sessions persist if daemon is separate)
4. Exit the process

## 4. Keyboard Shortcuts

### Global Shortcuts

| Key | Action |
|-----|--------|
| `↑` / `↓` | Navigate menu items |
| `Enter` | Select current menu item |
| `Esc` | Cancel / go back |
| `Ctrl+C` | Exit (SIGINT) |
| `Ctrl+L` | Refresh screen |

### Menu Navigation

| Key | Action |
|-----|--------|
| `K` | Open Keys menu |
| `D` | Open Desktop menu |
| `V` | Open Devices menu |
| `S` | Open Settings menu |
| `L` | Open Logs menu |
| `U` | Open Update menu |
| `Shift+Q` | Shutdown |

## 5. Status Indicators

### 5.1 Connection Status

| Icon | State | Description |
|------|-------|-------------|
| 🔴 | `STOPPED` | Tunnel is not running |
| 🟡 | `PREPARING` | Checking dependencies |
| 🟡 | `CONNECTING` | Creating session with worker |
| 🟡 | `TUNNELING` | Spawning Cloudflare tunnel |
| 🟡 | `VERIFYING` | Health check on tunnel URL |
| 🟢 | `READY` | Tunnel is live and verified |

### 5.2 Transport Status

| Mode | Description |
|------|-------------|
| `Tunnel` | Connected via Cloudflare Tunnel |
| `LAN` | Connected via local network (LocalFirst) |
| `Offline` | No active connection |

## 6. Command File Channel

The TUI communicates with the background server via a command file (`~/.9remote/state/cmd.json`):

```
TUI writes command → Server polls every 1s → Server executes command
```

| Command | TUI Source | Server Action |
|---------|-----------|---------------|
| `stop-tunnel` | Menu → Keys → Stop | Stop Cloudflare tunnel, clear tunnelUrl |
| `restart-tunnel` | Menu → Settings → Restart | Stop then start tunnel |
| `start-tunnel` | Startup | Start Cloudflare tunnel |
| `regenerate-key` | Menu → Keys → Regenerate | Generate new one-time key |
| `shutdown` | Menu → Shutdown | Stop tunnel + exit server |
| `update` | Menu → Update | Trigger update flow |
| `restart` | Menu → Settings → Restart | Restart agent process |
