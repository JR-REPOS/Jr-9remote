# API Endpoints Reference

> **Purpose:** Complete reference for all HTTP API endpoints exposed by the 9Remote server.
> **Scope:** All routes defined in `package/dist/server.cjs`
> **Default port:** 2208 (configurable via `PORT` env var)

## 1. Overview

The 9Remote server exposes a set of HTTP endpoints that serve both the embedded web UI and provide real-time state management capabilities. All state-changing endpoints are behind a security check that verifies either localhost access or the presence of the `CF-Connecting-IP` header (Cloudflare tunnel traffic).

**Base URL:** `http://localhost:2208` (local) or `https://<tunnel-url>` (remote)

## 2. Static Assets

### `GET /`
Returns the embedded Preact web dashboard (`ui/index.html`).

### `GET /ui/*`
Serves static UI files:
- `GET /ui/` — `index.html`
- `GET /ui/favicon.svg`
- `GET /ui/assets/index-CcS6AXZF.css`
- `GET /ui/assets/index-H_1CAVCP.js`
- `GET /ui/agents/{claude,codex,gemini,opencode}.svg`

### `GET /* (fallback)`
Falls through to the router fallback handler if no route matches.

## 3. Health Check

### `GET /api/health`
Checks if the server is responsive.

**Response (200):**
```json
{}
```

## 4. UI State Management

### `GET /api/ui/state`
Returns the current UI state including step, tunnel URL, keys, and theme.

**Response (200):**
```json
{
  "permanentKey": "string",
  "step": 0,
  "stepDesc": "string",
  "tunnelUrl": "string",
  "theme": "dark|light",
  "tunnelRetry": { "attempt": 0, "delay": 0, "at": 0 } | null,
  "oneTimeKey": "string",
  "oneTimeKeyExpiresAt": "ISO date string",
  "qrUrl": "string"
}
```

**State step enum (`STEP`):**
| Value | Name | Description |
|-------|------|-------------|
| 0 | `STOPPED` | Tunnel is not running |
| 1 | `PREPARING` | Checking dependencies / preparing |
| 2 | `CONNECTING` | Creating session with worker |
| 3 | `TUNNELING` | Spawning Cloudflare tunnel |
| 4 | `VERIFYING` | Health check on tunnel URL |
| 5 | `READY` | Tunnel is live and verified |

### `POST /api/ui/state`
Updates UI state in the server's in-memory state and persisted state file.

**Request Body:**
```json
{
  "permanentKey": "string",
  "step": 1,
  "stepDesc": "string",
  "tunnelUrl": "string",
  "theme": "dark",
  "tunnelRetry": null,
  "oneTimeKey": "string",
  "oneTimeKeyExpiresAt": "string",
  "qrUrl": "string"
}
```

**Response (200):** Echoes updated state.

## 5. Real-Time Event Stream (SSE)

### `GET /api/ui/events`
Server-Sent Events (SSE) stream that broadcasts real-time state changes. Clients receive JSON events with the following `type` values:

| Event Type | Payload |
|------------|---------|
| `state` | `{ permanentKey, step, stepDesc, tunnelUrl, theme, oneTimeKey, oneTimeKeyExpiresAt, qrUrl, tunnelRetry }` |
| `connections` | `{ connections: [{ socketId, ip, deviceId, type, connectedAt }] }` |
| `permissions` | `{ screenRecording, accessibility, desktopEnabled }` |
| `transport` | `{ signaling, rtcDisabled, remoteAvailable }` |
| `updateAvailable` | `{ version, url, ... }` (when update check result available) |

**SSE Format:**
```
data: { "type": "state", ... }

data: { "type": "connections", ... }

```

## 6. Device Auto-Approval

### `GET /api/device/auto-approve`
Returns whether new device connections are auto-approved.

**Response (200):**
```json
{ "enabled": false }
```

### `POST /api/device/auto-approve`
Sets auto-approval for new device connections.

**Request Body:**
```json
{ "enabled": true }
```

**Response (200):** The updated setting.

## 7. Autostart Management

### `GET /api/autostart`
Returns whether 9Remote is configured to auto-start on system boot.

**Response (200):**
```json
{ "enabled": false }
```

### `POST /api/autostart`
Enables or disables autostart on boot.

**Request Body:**
```json
{ "enabled": true }
```

**Response (200):** `{ "ok": true, "enabled": true }`

**Platform mechanisms:**
- **macOS:** LaunchAgent plist at `~/Library/LaunchAgents/cc.9remote.agent.plist`
- **Linux:** systemd user service at `~/.config/systemd/user/cc.9remote.agent.service` or XDG autostart `~/.config/autostart/cc.9remote.agent.desktop`
- **Windows:** VBS script in `APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\9Remote.vbs`

## 8. Sleep Inhibition

### `GET /api/sleep-inhibit`
Returns current sleep inhibition settings.

**Response (200):**
```json
{
  "mode": "never|always|active",
  "active": false,
  "presets": [{ "mode": "never", "label": "..." }, ...]
}
```

### `POST /api/sleep-inhibit`
Sets sleep inhibition mode.

**Request Body:**
```json
{ "mode": "always" }
```

**Response (200):** `{ "ok": true, "mode": "always" }`

**Platform mechanisms:**
- **macOS:** `caffeinate -imsd -w <pid>`
- **Linux:** `systemd-inhibit --what=idle:sleep:handle-lid-switch --who=9remote --why=remote-active sleep infinity`
- **Windows:** `PowerCreateRequest` / `PowerSetRequest` API via PowerShell

## 9. Permissions (macOS)

### `GET /api/permissions`
Returns current macOS permission status for screen recording and accessibility.

**Response (200):**
```json
{
  "screenRecording": false,
  "accessibility": false,
  "desktopEnabled": false
}
```

### `POST /api/permissions`
Triggers macOS system preferences for a specific permission type.

**Request Body:**
```json
{ "type": "screenRecording|accessibility" }
```

Opens:
- `x-apple.systempreferences:com.apple.preference.security?Privacy_ScreenCapture`
- `x-apple.systempreferences:com.apple.preference.security?Privacy_Accessibility`

### `POST /api/desktop`
Toggles remote desktop feature on/off.

**Request Body:**
```json
{ "enabled": true }
```

**Response (200):** `{ "ok": true, "enabled": true }`

## 10. Transport & Connectivity

### `GET /api/transport`
Returns current transport state.

**Response (200):**
```json
{
  "signaling": "connected|connecting|off",
  "rtcDisabled": false,
  "remoteAvailable": false
}
```

### `POST /api/transport`
Disables or enables WebRTC RTC.

**Request Body:**
```json
{ "rtcDisabled": true }
```

**Response (200):** Echoes `{ ok: true, ...transportState }`

## 11. Connections

### `GET /api/connections`
Returns active connections with their socket IDs, IPs, and device IDs.

**Response (200):**
```json
{
  "connections": [
    {
      "socketId": "string",
      "ip": "string",
      "deviceId": "string",
      "type": "ws",
      "connectedAt": 1234567890
    }
  ]
}
```

## 12. Logs

### `GET /api/logs?lines=N`
Returns the last N log entries (default: 60, max: 200).

**Response (200):**
```json
{ "logs": ["2024-01-01 00:00:00.000 [info] message", ...] }
```

### `DELETE /api/logs`
Clears the agent log file.

**Response (200):** `{ "ok": true }`

## 13. Update Management

### `POST /api/update`
Triggers an in-place update of the 9Remote npm package.

**Response (200):** `{ "ok": true }`

**Flow:** Downloads latest npm package, verifies SHA256 checksums, spawns a background process that:
1. Kills stale cloudflared/agent processes
2. Runs `npm install -g 9remote@latest`
3. Verifies the installed version matches
4. Restarts the agent

### `GET /api/update` (via SSE)
The `updateAvailable` SSE event is emitted when a new version is detected during SSE stream.

## 14. Cloudflare Worker API (`/api/session/create`)

> Note: This endpoint is called on the **9remote.cc** Cloudflare Worker, not the local server.

### `POST https://9remote.cc/api/session/create`
Creates a session on the Cloudflare Worker for tunnel coordination.

**Request Body:**
```json
{ "apiKey": "string" }
```

**Response (200):**
```json
{ "ok": true, "apiKey": "string" }
```

## 15. Security Model

### Access Control
The router enforces these rules:
- Requests from `127.0.0.1`, `::1`, or `::ffff:127.0.0.1` are always allowed (local)
- Requests with `CF-Connecting-IP` header are allowed (Cloudflare tunnel)
- All other requests return **403 Forbidden**

### Origin Check
For certain sensitive endpoints, the `origin` header is validated against an allowlist of known origins.
