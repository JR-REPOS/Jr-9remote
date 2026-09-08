# Configuration Files and Paths

> **Purpose:** Document all configuration files, paths, and storage locations used by 9Remote.
> **Scope:** `~/.9remote/` directory structure, file formats, and persistence.
> **Source:** `package/dist/cli.cjs`, `package/dist/server.cjs`

## 1. Overview

9Remote stores all runtime configuration, state, logs, and process information in the user's home directory under `~/.9remote/`.

| Environment | Root Path |
|------------|-----------|
| All platforms | `~/.9remote/` |
| macOS | `/Users/<username>/.9remote/` |
| Linux | `/home/<username>/.9remote/` |
| Windows | `%USERPROFILE%\.9remote\` |

## 2. Directory Structure

```
~/.9remote/
├── keys.json                  # Permanent key (machine ID derived)
├── autostart.log              # Autostart setup logs
├── logs/
│   ├── agent.log              # Server log (max 2KB, rotated)
│   └── agent.log.1            # Rotated log
├── state/
│   ├── state.json             # Current UI state
│   ├── settings.json          # User settings (theme, etc.)
│   └── cmd.json               # Command channel (TUI → Server)
├── pids/
│   ├── cloudflared.pid        # Cloudflare tunnel PID
│   ├── agent.pid              # Main agent process PID
│   └── ptyDaemon.pid          # PTY daemon PID
├── daemon/                    # PTY daemon workspace
├── buffers/                   # Uploaded file buffers
├── runtime/                   # Runtime dependencies (systray2)
│   ├── package.json
│   └── node_modules/
│       └── systray2/
│           └── traybin/
│               ├── tray_darwin_release
│               ├── tray_windows_release.exe
│               └── tray_linux_release
└── pty-daemon.sock            # Unix socket (Windows: \\.\pipe\9remote-pty)
```

## 3. Configuration Files

### 3.1 `keys.json`

Stores the permanent key and machine identity.

```json
{
  "machineId": "9f3a2b1c4d5e6f7a",
  "key": "sk-9f3a2b1c-d4e5-f6a7b8c9d0e1f2",
  "name": "Default",
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `machineId` | `string` | 16-char SHA256 hash of machine ID + salt |
| `key` | `string` | Permanent key (`sk-` format) |
| `name` | `string` | Device name (default: "Default") |
| `createdAt` | `string` | ISO timestamp of key creation |

**Generation:** `xt()` function in `cli.cjs`
```javascript
const machineId = machineIdSync(); // platform-specific
const salted = sha256(machineId + salt);
const keyId = salted.substring(0, 16);
// Then: generate key with random + HMAC
```

### 3.2 `state/state.json`

Stores the current UI state. Written by the server via `fs.writeFileSync`.

```json
{
  "permanentKey": "sk-abcd1234-ijkl-mnopqr",
  "step": 5,
  "stepDesc": "",
  "tunnelUrl": "https://abcd1234.trycloudflare.com",
  "theme": "dark",
  "oneTimeKey": "",
  "oneTimeKeyExpiresAt": null,
  "qrUrl": "",
  "tunnelRetry": null
}
```

**State step enum:**
| Value | Name | Description |
|-------|------|-------------|
| 0 | `STOPPED` | Tunnel stopped |
| 1 | `PREPARING` | Checking dependencies |
| 2 | `CONNECTING` | Creating session |
| 3 | `TUNNELING` | Spawning tunnel |
| 4 | `VERIFYING` | Health check |
| 5`READY | Tunnel live | Tunnel live and verified |

### 3.3 `state/settings.json`

User settings persisted across restarts.

```json
{
  "theme": "dark",
  "autoApprove": false,
  "sleepInhibitMode": "never"
}
```

### 3.4 `state/cmd.json`

Command channel file used by TUI to send commands to the server.

```json
"stop-tunnel"
```

**Valid commands:**
| Command | Action |
|---------|--------|
| `stop-tunnel` | Stop Cloudflare tunnel |
| `restart-tunnel` | Restart tunnel |
| `start-tunnel` | Start tunnel |
| `regenerate-key` | Generate new one-time key |
| `shutdown` | Stop everything and exit |
| `update` | Trigger update |
| `restart` | Restart agent process |

**Polling:** Server checks this file every 1 second (1000ms).

### 3.5 `pids/cloudflared.pid`

Contains the PID of the running `cloudflared` process (plain text).

### 3.6 `pids/agent.pid`

Contains the PID of the main agent/server process (plain text).

### 3.7 `pids/ptyDaemon.pid`

Contains the PID of the PtyDaemon process (plain text).

### 3.8 `pty-daemon.sock`

Unix domain socket for communication between the server and PtyDaemon.

- **Unix:** `~/.9remote/pty-daemon.sock` (permissions: 0600)
- **Windows:** `\\.\pipe\9remote-pty`

## 4. Log Files

### 4.1 `logs/agent.log`

- **Max size:** 2KB (rotated when exceeded)
- **Rotation:** Renamed to `agent.log.1`, new file created
- **Cleanup:** Files older than 7 days are deleted
- **Log level:** `process.env.NODE_ENV === "development"` or `AGENT_DEBUG=1` enables debug logs
- **Format:** `{HH:mm:ss.SSS [loggerName] message}`

**Example log entries:**
```
=== SESSION 2024-01-01T00:00:00.000Z pid=12345 argv=start --tray ===
2024-01-01 00:00:00.000 [info] Tunnel ready: https://abcd.trycloudflare.com
2024-01-01 00:00:01.000 [tunnel] URL rotated: https://newabc.trycloudflare.com
2024-01-01 00:00:02.000 [network] Network change detected
```

### 4.2 `logs/agent.log.1`

- Previous rotated log file
- Overwritten on next rotation

### 4.3 `autostart.log`

- Logs from autostart setup/teardown
- Written by the server process

## 5. Runtime Dependencies

### 5.1 Runtime Directory (`runtime/`)

- Created by `install.cjs` post-install script
- Contains `systray2` for system tray support
- Installed via `npm install systray2@2.1.4` in this directory
- Native tray binaries stored in `runtime/node_modules/systray2/traybin/`

### 5.2 Buffers Directory (`buffers/`)

- Temporary storage for file uploads
- Files named `{timestamp}_{sanitized_filename}`
- Cleaned up after upload processing

## 6. State Persistence Functions

### 6.1 State Save (server.cjs)

```javascript
// fn(state) - writes to state.json
function saveState(state) {
  mkdirSync(re.STATE, { recursive: true });
  writeFileSync(path.join(re.STATE, 'state.json'), JSON.stringify(state, null, 2), { mode: 0o600 });
}
```

### 6.2 Command File (server.cjs)

```javascript
// Reads cmd.json every 1 second
// st() function - command poll
setInterval(async () => {
  const cmd = readCmdFile();
  if (cmd) {
    switch(cmd) {
      case 'stop-tunnel': await stopTunnel(); break;
      case 'restart-tunnel': await restartTunnel(); break;
      // ...
    }
  }
}, 1000);
```

### 6.3 Keys Save (cli.cjs)

```javascript
// ut(keys) - writes to keys.json
function saveKeys(keys) {
  mkdirSync(re.ROOT, { recursive: true });
  writeFileSync(path.join(re.ROOT, 'keys.json'), JSON.stringify({ machineId, key, name, createdAt }, null, 2));
}
```

## 7. File Permissions

| File | Permission | Rationale |
|------|-----------|-----------|
| `pty-daemon.sock` | 0600 | Unix socket for daemon comms |
| `state/state.json` | 0600 | Contains permanent key |
| `keys.json` | default | Contains permanent key |
| `pids/*.pid` | 0600 | Process management |
| `runtime/` | 0755 | Systray native binaries |
| `logs/agent.log` | default | Log file |
