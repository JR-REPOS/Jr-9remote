# Socket.IO Events Reference

> **Purpose:** Document all Socket.IO real-time events used in the 9Remote system.
> **Scope:** Client-server Socket.IO communication between the web UI and the server.
> **Source:** `package/dist/server.cjs` and `package/dist/ui/assets/index-H_1CAVCP.js`

## 1. Overview

The 9Remote server uses **Socket.IO v4** as its real-time communication layer. Socket.IO provides bidirectional event-based communication over WebSocket with fallback to HTTP long-polling. The events facilitate terminal I/O, session management, desktop screen sharing, file transfers, and clipboard operations.

## 2. Connection Details

- **Namespace:** Default namespace (`/`)
- **Transport:** WebSocket (primary), polling (fallback)
- **Authentication:** Pair Device approval system; connection metadata includes `deviceId`

## 3. Server-to-Client Events (Server Emits)

### `output`
Emitted when terminal output is received from a PTY session.

**Payload:**
```json
{
  "type": "output",
  "sessionId": "string",
  "enc": "string|b64",
  "replay": false,
  "data": "string (base64 if enc='b64')"
}
```

### `sessionClosed`
Emitted when a PTY session has been terminated.

**Payload:**
```json
{
  "type": "sessionClosed",
  "sessionId": "string"
}
```

### `cwdChange`
Emitted when the working directory changes in an active session.

**Payload:**
```json
{
  "type": "cwdChange",
  "sessionId": "string",
  "cwd": "/current/working/directory"
}
```

### `sessionList`
Response to `listSessions` request (used by ptyDaemon protocol).

**Payload:**
```json
{
  "type": "sessionList",
  "sessions": [
    {
      "id": "string",
      "name": "string",
      "createdAt": 1234567890,
      "shellId": "bash|zsh|pwsh|...",
      "shellLabel": "Bash|Zsh|PowerShell",
      "cwd": "/path"
    }
  ]
}
```

### `createResult`
Response to `createSession` request.

**Payload:**
```json
{
  "type": "createResult",
  "success": true,
  "sessionId": "string",
  "cwd": "/path",
  "shellId": "bash",
  "shellLabel": "Bash"
}
```

### `joinResult`
Response to `joinSession` request.

**Payload:**
```json
{
  "type": "joinResult",
  "success": true,
  "cwd": "/path",
  "shellId": "bash",
  "shellLabel": "Bash"
}
```

### `pong`
Response to `ping` request (keepalive).

**Payload:**
```json
{
  "type": "pong",
  "version": "46",
  "requestId": "string"
}
```

## 4. Client-to-Server Events (Client Can Emit)

### `input`
Sends terminal input (keystrokes) to a PTY session.

**Payload:**
```json
{
  "sessionId": "string",
  "data": "keystroke data (string)"
}
```

### `resize`
Resizes a PTY session's terminal dimensions.

**Payload:**
```json
{
  "sessionId": "string",
  "cols": 80,
  "rows": 24
}
```

### `session-resume`
Requests to resume a previously active PTY session (e.g., for Claude Code `--resume`).

**Payload:**
```json
{
  "sessionId": "string"
}
```

### `upload-file`
Uploads a file to the host machine via drag-and-drop or file picker.

**Payload:**
```json
{
  "sessionId": "string",
  "filename": "example.txt",
  "size": 1024,
  "content": "base64-encoded file content"
}
```

## 5. PtyDaemon Protocol (Unix Socket)

The server communicates with the persistent PTY daemon via a **Unix domain socket** (`~/.9remote/pty-daemon.sock` on Unix, `\\.\pipe\9remote-pty` on Windows). The protocol uses newline-delimited JSON messages.

### Daemon Version
Protocol version string: `"46"`

### Client-to-Daemon Messages

| Message Type | Fields | Description |
|---|---|---|
| `ping` | `requestId` | Keepalive check; daemon responds with `pong` |
| `listSessions` | `requestId` | List all active sessions |
| `createSession` | `requestId`, `sessionId`, `name`, `cols`, `rows`, `shellId`, `cwd` | Create a new PTY session |
| `joinSession` | `requestId`, `sessionId` | Join an existing session |
| `input` | `requestId`, `sessionId`, `data` | Send input to session |
| `resize` | `requestId`, `sessionId`, `cols`, `rows` | Resize terminal |
| `deleteSession` | `requestId`, `sessionId` | Kill/delete a session |
| `getCwd` | `requestId`, `sessionId` | Get current working directory |
| `requestHistory` | `requestId`, `sessionId`, `have` | Request command history buffer |

### Daemon-to-Client Messages

| Message Type | Fields | Description |
|---|---|---|
| `pong` | `requestId`, `version` | Keepalive response |
| `sessionList` | `requestId`, `sessions[]` | List of sessions |
| `createResult` | `requestId`, `success`, `sessionId`, `cwd`, `shellId`, `shellLabel` | Session creation result |
| `joinResult` | `requestId`, `success`, `cwd`, `shellId`, `shellLabel` | Session join result |
| `output` | `sessionId`, `enc`, `replay`, `data` | Terminal output data |
| `sessionClosed` | `sessionId` | Session was terminated |
| `cwdChange` | `sessionId`, `cwd` | Working directory changed |

## 6. Shell Types Supported

| Platform | Shell ID | Path | Args |
|---|---|---|---|
| macOS / Linux | `bash` | `/bin/bash` | `["-l"]` |
| macOS / Linux | `zsh` | `/bin/zsh` | `["-l"]` |
| macOS / Linux | `sh` | `/bin/sh` | `["-l"]` |
| macOS / Linux | `SHELL` | `$SHELL` | `["-l"]` (fallback) |
| Windows | `cmd` | `cmd.exe` | `[]` |
| Windows | `powershell` | `powershell.exe` | `["-NoLogo", "-NoExit", "-Command", promptFunc]` |
| Windows | `pwsh` | `pwsh.exe` | `["-NoLogo", "-NoExit", "-Command", promptFunc]` |

## 7. Event Flow Summary

```
┌─────────────┐    Socket.IO WS    ┌──────────┐    Unix Socket    ┌───────────┐
│   Web UI    │  ◄──────────────►  │  Server  │  ◄──────────────► │ PtyDaemon │
│ (Preact)    │                    │ (Node.js)│                    │ (Node.js) │
└─────────────┘                    └──────────┘                    └───────────┘
     │                                    │                                │
     │ input/resize                       │ forward input/resize           │
     ├──────────────►                     ├────────────────────────────────►
     │                                    │                                │
     │ output/sessionClosed/cwdChange     │ receive output                 │
     │ ◄──────────────────                │ ◄────────────────────────────────
     │                                    │ emit to client                 │
     │ output                             │ ◄──────────────────
     └────────────────────────────────────┘
```
