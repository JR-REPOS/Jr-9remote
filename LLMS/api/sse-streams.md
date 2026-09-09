# Server-Sent Events (SSE) Reference

> **Purpose:** Document all Server-Sent Events streams and event types.
> **Scope:** `GET /api/ui/events` SSE endpoint in `package/dist/server.cjs`

## 1. Overview

The 9Remote server provides a **Server-Sent Events (SSE)** stream at `/api/ui/events` for real-time state broadcasting. This is used by the embedded web UI to keep the dashboard in sync with server-side state without polling. SSE is unidirectional (server to client) over HTTP, with automatic reconnection handled by the browser's `EventSource` API.

## 2. Endpoint Details

### `GET /api/ui/events`

**Headers:**
```
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

**SSE Event Format:**
```
data: { "type": "state", ... }

data: { "type": "connections", ... }

```

Each event is separated by a blank line. A single logical SSE message may contain multiple `data:` lines (for buffered output), but state events send one JSON object per `data:` line.

## 3. Event Types

### `state`
Broadcast whenever UI state changes (step transitions, tunnel URL updates, key generation).

**Payload:**
```json
{
  "type": "state",
  "permanentKey": "sk-xxx-xxx-xxxxxx",
  "step": 5,
  "stepDesc": "",
  "tunnelUrl": "https://abcd1234.trycloudflare.com",
  "theme": "dark",
  "oneTimeKey": "temp-key-here",
  "oneTimeKeyExpiresAt": "2024-01-01T00:30:00.000Z",
  "qrUrl": "https://9remote.cc/login?k=temp-key-here",
  "tunnelRetry": null
}
```

**Field Reference:**
| Field | Type | Description |
|---|---|---|
| `permanentKey` | `string` | Machine-derived permanent API key (`sk-{8}-{4}-{6}`) |
| `step` | `number` | State enum: 0=STOPPED, 1=PREPARING, 2=CONNECTING, 3=TUNNELING, 4=VERIFYING, 5=READY |
| `stepDesc` | `string` | Human-readable step description |
| `tunnelUrl` | `string` | Cloudflare tunnel URL (empty if tunnel is down) |
| `theme` | `string` | UI theme (`dark` or `light`) |
| `oneTimeKey` | `string` | Temporary QR login key (30-min expiry) |
| `oneTimeKeyExpiresAt` | `string` | ISO timestamp for one-time key expiry |
| `qrUrl` | `string` | Full QR code login URL |
| `tunnelRetry` | `object\|null` | Retry attempt metadata |

### `connections`
Broadcast on connection state changes (new device connects/disconnects).

**Payload:**
```json
{
  "type": "connections",
  "connections": [
    {
      "socketId": "socket-id",
      "ip": "192.168.1.1",
      "deviceId": "device-uuid",
      "type": "ws",
      "connectedAt": 1700000000000
    }
  ]
}
```

### `permissions`
Broadcast when macOS permissions change.

**Payload:**
```json
{
  "type": "permissions",
  "screenRecording": false,
  "accessibility": false,
  "desktopEnabled": false
}
```

### `transport`
Broadcast when transport state changes (LAN vs tunnel, RTC availability).

**Payload:**
```json
{
  "type": "transport",
  "signaling": "connected",
  "rtcDisabled": false,
  "remoteAvailable": true
}
```

**Transport states:**
| `signaling` value | Description |
|---|---|
| `connected` | Socket.IO signaling channel is active |
| `connecting` | Signaling channel connecting |
| `off` | No tunnel/remote connection |

### `updateAvailable`
Broadcast when an update check completes and a new version is detected.

**Payload:**
```json
{
  "type": "updateAvailable",
  "version": "2.5.9",
  "url": "https://registry.npmjs.org/9remote/-/9remote-2.5.9.tgz"
}
```

## 4. Reconnection Behavior

The browser's `EventSource` automatically reconnects on:
- Network disconnection
- Server restart
- HTTP errors

On reconnect, the server immediately sends the current state as a `state` event, followed by `connections`, `permissions`, and `transport` events if applicable.

## 5. SSE vs Socket.IO Comparison

| Feature | SSE (`/api/ui/events`) | Socket.IO |
|---|---|---|
| Direction | Server → Client only | Bidirectional |
| Transport | HTTP streaming | WebSocket + polling |
| Reconnection | Automatic (browser) | Automatic (library) |
| Use case | UI state sync, dashboard updates | Terminal I/O, real-time events |
| Message format | `data: {JSON}` | Event-based JSON |

The web UI uses **both**: SSE for dashboard state and Socket.IO for terminal/control operations.
