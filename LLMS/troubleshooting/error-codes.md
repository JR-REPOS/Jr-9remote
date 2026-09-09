# Error Codes & Messages

> **Purpose:** Reference of all error codes, error messages, and their meanings.
> **Scope:** All error conditions and their troubleshooting steps.
> **Source:** `package/dist/cli.cjs`, `package/dist/server.cjs`, `package/dist/ptyDaemon.cjs`

## 1. HTTP Status Codes

### API Responses

| Status | Message | Description |
|--------|---------|-------------|
| 200 | OK | Request successful |
| 400 | Invalid JSON | Request body is not valid JSON |
| 400 | Bad request | Malformed URL or query parameters |
| 403 | Forbidden | Request from non-local/non-Cloudflare source |
| 403 | Forbidden origin | `origin` header not in allowlist |
| 404 | Not found | Route does not exist |
| 500 | Internal Server Error | Unhandled server exception |

### Worker API Responses

| Status | Message | Description |
|--------|---------|-------------|
| 200 | OK | Session created or validated |
| 400 | Missing API key | `apiKey` field not provided |
| 401 | Invalid API key | HMAC verification failed |
| 500 | Worker error | Internal worker error |

## 2. PtyDaemon Messages

### 2.1 Error Messages

| Message | Description | Cause |
|---------|-------------|-------|
| `Session already exists` | Attempted to create a session with an existing ID | Duplicate session name |
| `Session not found` | Join/delete requested for non-existent session | Session was closed or daemon restarted |
| `Invalid message` | JSON parse error in socket protocol | Corrupted message over socket |
| `Client error` | Socket connection error | Client disconnected abruptly |
| `Server error` | Socket server bind error | Port/socket already in use |
| `Socket already in use, exiting` | Unix socket is already bound | Stale socket file or running daemon |

### 2.2 Daemon Protocol Errors

| Message | Code Path |
|---------|-----------|
| `Failed to create session` | `fe()` function in ptyDaemon |
| `pipe timeout` | Socket handshake timeout |
| `Invalid message` | `JSON.parse()` failure in `ue()` handler |

## 3. CLI/Server Error Messages

### 3.1 Tunnel Errors

| Message | Function | Meaning |
|---------|----------|---------|
| `Tunnel spawn failed: {error} — QR RTC-only, bg retry` | `Fe()` / `Gs()` | cloudflared failed to start; UI continues with RTC-only mode |
| `Quick tunnel timed out after 90s` | `Fe()` | cloudflared didn't produce a URL in 90s |
| `cloudflared exited (code={code})` | `Fe()` | cloudflared process terminated |
| `Tunnel health check timed out — bg reconnect` | `rt()` / `Gs()` | Health check to tunnel URL failed |
| `tunnel unreachable but cloudflared alive — edge lost` | `Nc()` | Cloudflare edge lost the tunnel |
| `edge probe failing but cloudflared holds {n} connection(s)` | `Nc()` | Edge shows cloudflared but no edge connections |

### 3.2 Session Errors

| Message | Function | Meaning |
|---------|----------|---------|
| `Session create failed: {status}` | `Gs()` / `sl()` | Worker API returned error status |
| `Failed to start: {error}` | `Zs()` | General startup failure in TUI mode |
| `Failed to connect: {error}` | `Zs()` | Session creation threw exception |

### 3.3 Update Errors

| Message | Function | Meaning |
|---------|----------|---------|
| `Verify failed (got {version}, want {expected})` | Update script | Installed version doesn't match expected |
| `Integrity check failed: SHA256 mismatch` | `wc()` | Downloaded package checksum mismatch |
| `Install failed` | Update script | npm install failed |
| `Background server failed to start` | `Zs()` | Updated server didn't come online |

### 3.4 Permission Errors (macOS)

| Message | Function | Meaning |
|---------|----------|---------|
| `permissions_required` | `/api/desktop` endpoint | Screen Recording or Accessibility permission not granted |

### 3.5 Process Errors

| Message | Function | Meaning |
|---------|----------|---------|
| `killing stale cloudflared pid={pid}` | `Fe()` | Old tunnel process detected and killed |
| `Agent v{version} != v{expected}, restarting` | `VN()` | PtyDaemon version mismatch, restarting daemon |

## 4. Exit Codes

| Code | Signal | Description |
|------|--------|-------------|
| 0 | — | Success / graceful shutdown |
| 1 | — | General error (startup failure, session error) |
| 128 | SIGINT | Interrupt signal (Ctrl+C) |
| 143 | SIGTERM | Termination signal |

## 5. Logger Names

The server uses named loggers for filtering. Log format: `{timestamp} [{loggerName}] {message}`

| Logger | Module | Describes |
|--------|--------|-----------|
| `cmd` | cli.cjs | Command polling from cmd.json |
| `mode` | cli.cjs | UI mode operations |
| `crash` | Both | Uncaught exceptions / unhandled rejections |
| `tunnel` | server.cjs | cloudflared lifecycle |
| `network` | server.cjs | Network changes, sleep/wake |
| `desktop-bridge` | server.cjs | Desktop streaming operations |
| `install` | install.cjs | Post-install setup |
| `worker` | server.cjs | Cloudflare Worker API calls |
| `sleep` | server.cjs | Sleep inhibition |
| `autostart` | server.cjs | Autostart setup/teardown |
| `update` | cli.cjs | Update checks and installation |
| `permissions` | server.cjs | macOS permission status |

## 6. Error Context

### 6.1 Common Root Causes

| Symptom | Likely Cause | Check |
|---------|-------------|-------|
| Can't connect from phone | Tunnel not ready | TUI status = `READY`? |
| Tunnel stuck at VERIFYING | Health check failing | Is `localhost:2208/api/health` accessible? |
| Black desktop | Permissions (macOS) | Screen Recording + Accessibility granted? |
| robotjs not found | Optional dependency missing | `npm install -g 9remote` (clean install) |
| Systray missing | systray2 install failed | Check `~/.9remote/runtime/node_modules/systray2/` |
| Update fails | Network/npm issue | Try `npm install -g 9remote@latest` manually |
| PTY not spawning | node-pty not available | Check optional dependencies |

### 6.2 Debugging Checklist

1. **Check process status:**
   ```bash
   ps aux | grep 9remote      # Agent process
   ps aux | grep cloudflared  # Tunnel process
   ps aux | grep ptyDaemon    # Daemon process
   ```

2. **Check PID files:**
   ```bash
   cat ~/.9remote/pids/*.pid
   ```

3. **Check logs:**
   ```bash
   NODE_ENV=development 9remote  # Verbose output
   ```

4. **Check health endpoint:**
   ```bash
   curl http://localhost:2208/api/health
   ```

5. **Check API state:**
   ```bash
   curl http://localhost:2208/api/ui/state | jq .
   ```
