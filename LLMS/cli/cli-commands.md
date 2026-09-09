# CLI Commands & Reference

> **Purpose:** Complete reference for the 9Remote CLI commands, flags, and exit codes.
> **Scope:** All command-line entry points and their options.
> **Source:** `package/package.json` (bin), `package/dist/cli.cjs`

## 1. Installation

### 1.1 Global Installation (Recommended)

```bash
npm install -g 9remote
```

This installs the `9remote` binary globally, linked to `dist/cli.cjs`.

### 1.2 Post-Install

The `postinstall` script (`dist/install.cjs`) automatically:
1. Checks for `@hurdlegroup/robotjs` (enables remote desktop)
2. Installs `systray2` v2.1.4 in `~/.9remote/runtime/` (for system tray)

```
⏳ Installing system tray (first run)...
✅ System tray ready
✅ robotjs installed (Remote desktop available)
```

Or if robotjs is not available:
```
ℹ️  robotjs not available (Remote desktop disabled)
```

## 2. CLI Commands

### 2.1 `9remote` (TUI Mode — Default)

Starts 9Remote in **Terminal User Interface (TUI) mode**. Displays an interactive terminal UI with:

- QR code for device pairing
- Status spinner (Preparing → Connecting → Tunneling → Verifying → Ready)
- Menu navigation for configuration

**Usage:**
```bash
9remote
```

**Flags:**
| Flag | Description |
|------|-------------|
| (none) | Default: Interactive TUI with QR code |
| `--theme=dark\|light` | Force a specific theme |
| `--start` | Start in background mode (headless) |
| `--tray` | Enable system tray |
| `--skip-update` | Skip version update check |

### 2.2 `9remote ui` (Web UI Mode)

Starts 9Remote in **Web UI mode**. Launches the HTTP server on port 2208 and opens the embedded web dashboard in the default browser. Also starts the system tray for background management.

**Usage:**
```bash
9remote ui
```

**Flags:**
| Flag | Description |
|------|-------------|
| `--start` | Start in background mode (for autostart) |
| `--tray` | Enable system tray (default in ui mode) |
| `--skip-update` | Skip version update check |
| `--theme=dark\|light` | Force a specific theme |

**Output:**
```
📡 9Remote running at http://localhost:2208 (PID: 12345)
💡 Log: ~/.9remote/logs/agent.log
🔔 9Remote is running
```

### 2.3 `9remote --version`

Prints the current version and exits.

```bash
9remote --version
# 9remote/2.5.8 linux-x64 node-v20.11.1
```

### 2.4 `9remote --help`

Displays help information.

## 3. TUI Menu Structure

When in TUI mode (or via the web UI dashboard), the following menu is available:

```
════════════════════════════════════════════
 🚀 9Remote — Terminal in Your Pocket
════════════════════════════════════════════

  [1] Keys        🔑 Manage pairing keys (permanent + one-time)
  [2] Desktop     🖥️  Toggle remote desktop (permissions required)
  [3] Devices     📱 Manage connected & paired devices
  [4] Settings    ⚙️  Theme, autostart, sleep inhibition
  [5] Logs        📜 View server logs
  [6] Update      🆕 Check for updates
  [7] Shutdown    ⏻ Stop server, tunnel, and quit

  Use ↑/↓ arrows to navigate, Enter to select, Esc to exit
```

### Menu Item Details

#### 1. Keys
- View permanent key and one-time key
- Regenerate one-time key (30-minute TTL)
- Copy keys to clipboard

#### 2. Desktop
- Toggle remote desktop on/off
- macOS: Requests Screen Recording + Accessibility permissions
- Checks `robotjs` availability

#### 3. Devices
- List connected devices (socket ID, IP, device ID, connection time)
- List pending/approved devices
- Revoke device access
- Toggle auto-approve for new devices

#### 4. Settings
- **Theme:** Dark / Light mode
- **Autostart:** Enable/disable auto-start on system boot
- **Sleep Inhibition:** Never / Always / During active sessions
- **Connection:** Tunnel only / LAN preferred (LocalFirst)

#### 5. Logs
- View last 200 log entries
- Filter by level (info/warn/error/debug)
- Clear logs

#### 6. Update
- Check for new version
- Download and install (background process)
- Rollback on failure

#### 7. Shutdown
- Stops Cloudflare tunnel
- Stops PtyDaemon
- Cleans up PID files
- Quits the agent

## 4. Environment Variables Reference

See [Environment Variables](config/environment-vars.md) for full details.

| Variable | Default |
|----------|---------|
| `PORT` | `2208` |
| `NREMOTE_WORKER_URL` | `https://9remote.cc` |
| `NREMOTE_REGISTRY` | `https://registry.npmjs.org/9remote/latest` |
| `MACHINE_ID_SALT` | `9remote-salt` |
| `API_KEY_SECRET` | `9remote-api-key-secret` |
| `NODE_ENV` | (unset) |
| `AGENT_DEBUG` | (unset) |

## 5. Exit Codes

| Code | Description |
|------|-------------|
| `0` | Success / graceful shutdown |
| `1` | General error (startup failure, session creation failure) |
| `128` | SIGINT (Ctrl+C during TUI) |
| `143` | SIGTERM |

## 6. Log Files

| File | Description |
|------|-------------|
| `~/.9remote/logs/agent.log` | Main agent logs (2KB rotation) |
| `~/.9remote/autostart.log` | Autostart setup logs |

**Debug mode:** Set `NODE_ENV=development` or `AGENT_DEBUG=1` for verbose output.

## 7. Troubleshooting Commands

```bash
# Port already in use
pkill -f 9remote
# Or: PORT=3308 9remote

# Check if tunnel is running
cat ~/.9remote/pids/cloudflared.pid
ps aux | grep cloudflared

# Check ptyDaemon
cat ~/.9remote/pids/ptyDaemon.pid

# View logs
cat ~/.9remote/logs/agent.log

# Check health
curl http://localhost:2208/api/health

# Manually kill stale processes
pkill -f cloudflared
pkill -f ptyDaemon
```
