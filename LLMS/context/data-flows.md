# Data Flows

> **Purpose:** Document the key data flows through the 9Remote system.
> **Scope:** Terminal I/O, desktop streaming, file transfer, authentication, tunneling.
> **Source:** `package/dist/cli.cjs`, `package/dist/server.cjs`, `package/dist/ptyDaemon.cjs`

## 1. Overview

9Remote handles multiple concurrent data flows across different transport mechanisms. This document describes each flow from end-to-end, including the protocols, channels, and components involved.

## 2. Terminal I/O Flow

The terminal flow enables remote shell access from a phone or browser.

```
┌─────────────┐         ┌──────────────────┐    Unix Socket    ┌──────────────┐
│  Phone/Browser │     │   Server (port 2208)   │◄──────►│  PtyDaemon    │
│  (xterm.js)   │     │   (Socket.IO)        │  JSON    │  (node-pty)   │
└──────┬────────┘       └──────────┬─────────┘           └───────┬──────┘
       │ Socket.IO WS                │                           │
       │                             │ forward input/resize     │
       │ [input]                     │ ───────►                  │ spawn shell
       │ ◄───────────────────────────┤                           │
       │ [resize]                    │ ───────►                  │
       │                             │                           │
       │ [output]                    │ ◄────────────────────────  │
       │ ◄───────────────────────────┤   emit to client           │
       │                             │                           │
       │ [sessionClosed]             │ ◄────────────────────────  │
       │ ◄───────────────────────────┤                           │
       │ [cwdChange]                 │ ◄────────────────────────  │
       └──────────────┬─────────────┴───────────────────────────┘
                      │
               ┌──────▼──────┐
               │ Cloudflare  │  HTTPS/WSS (tunnel)
               │ Tunnel      │
               └──────┬──────┘
                      │
               ┌──────▼──────┐
               │ 9remote.cc  │  Session coordination
               │ Worker      │
               └─────────────┘
```

### Flow Steps:
1. **Session Creation:** Phone WebSocket → Server → forward `createSession` → PtyDaemon spawns `node-pty` shell
2. **Session Join:** Phone sends `joinSession` with `sessionId` — PtyDaemon attaches to existing session
3. **Input:** Phone keystrokes → Socket.IO `input` event → Server → PtyDaemon `input` message → `pty.write()`
4. **Output:** Shell output → `pty.onData()` → PtyDaemon `output` message → Server → Socket.IO broadcast → Phone browser renders in xterm.js
5. **Resize:** Phone sends `resize` → Server → PtyDaemon → `pty.resize(cols, rows)`
6. **Session Close:** Shell exits → `pty.onExit()` → PtyDaemon sends `sessionClosed` → Server → Phone receives event

### Data Encoding:
- **Terminal output:** Base64-encoded (`enc: "b64"`) for binary-safe transport
- **Input:** UTF-8 strings
- **CWD changes:** Parsed from OSC 7 escape sequences in shell output

## 3. Desktop Streaming Flow

Desktop streaming uses WebRTC for low-latency screen sharing.

```
┌─────────────────────────────────────────────────────────┐
│  Phone/Browser (WebRTC Client)                          │
│  ┌─────────────────────────────────────┐                │
│  │ WebRTC PeerConnection               │                │
│  │ - Video track (received)            │                │
│  │ - DataChannel (input events)        │                │
│  └──────────────┬──────────────────────┘                │
│                 │ WebRTC over Cloudflare Tunnel        │
│                 ▼                                        │
│  ┌─────────────────────────────────────┐                │
│  │ Server (port 2208)                   │                │
│  │ - node-datachannel (WebRTC)         │                │
│  │ - node-screenshots (capture)        │                │
│  │ - @hurdlegroup/robotjs (input)      │                │
│  └──────────────┬──────────────────────┘                │
│                 │                                        │
│                 │ Screen capture loop                    │
│                 │ - Capture screen → encode JPEG        │
│                 │ - Send via DataChannel or RTP          │
│                 ▼                                        │
│  ┌─────────────────────────────────────┐                │
│  │ Host: macOS/Linux/Windows          │                │
│  └─────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────┘
```

### Flow Steps:
1. **Signaling:** WebRTC signaling channel negotiated via Socket.IO
2. **Capture:** Server captures screen via `node-screenshots` (30-60ms intervals)
3. **Encoding:** Frames encoded as JPEG via `@julusian/jpeg-turbo` (or GPU-accelerated on Windows)
4. **Transmission:** Frames sent over WebRTC DataChannel (or RTP for video track)
5. **Input:** Mouse/keyboard events sent from phone → WebRTC DataChannel → `robotjs` → host input injection
6. **Rendering:** Phone renders video frames in `<video>` or `<canvas>` element

### Optimization Strategies:
- **Tile-based diff:** Only changed screen regions are sent
- **Adaptive framerate:** 60ms active / 400ms idle
- **GPU acceleration:** OpenCL on Windows for fast scaling

## 4. File Transfer Flow

```
┌─────────────┐  ┌────────────────┐  ┌─────────────┐
│  Phone      │  │  Server        │  │  Host FS    │
│  Browser    │  │  (port 2208)   │  │             │
└──────┬──────┘  └───────┬────────┘  └──────┬──────┘
       │                  │                  │
       │ upload-file     │                  │
       │ (base64 content)│                  │
       ├─────────────────┼──────────────────►
       │                  │  fs.writeFileSync  │
       │                  │  to temp location │
       │                  │◄──────────────────┤
       │                  │                  │
       │ (terminal writes   │                  │
       │  temp file path)   │                  │
       ◄──────────────────┤──────────────────┤
       │                  │  pty.write(path)  │
       │                  │                   │
```

### Flow Steps:
1. User drags/drops a file in the web UI
2. File content is base64-encoded and sent via Socket.IO `upload-file` event
3. Server writes the file to `~/.9remote/buffers/{timestamp}_{sanitized_name}`
4. Server sends the temp file path to the PTY session as terminal input
5. The shell can then process the file (e.g., `cat file.txt`)

## 5. Authentication & Key Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Key Generation (First Run)               │
│                                                             │
│  1. machine-id ──► SHA256(machineId + salt) ──► 16-char key │
│  2. Generate permanent key: sk-{8chars}-{4chars}-{6hmac}   │
│  3. Store in ~/.9remote/keys.json                          │
│                                                             │
│  4. Generate one-time key: same format, 30-min expiry      │
│  5. QR code: https://9remote.cc/login?k={tempKey}          │
└─────────────────────────────────────────────────────────────┘
```

### Key Format:
```
sk-{first-8-of-permanent}-{4-random-chars}-{6-char-hmac}
```
- `sk-` prefix
- 8-char slice of permanent key
- 4 random chars from `abcdefghijklmnopqrstuvwxyz0123456789`
- 6-char HMAC-SHA256 truncated from (keyId + apiKeySecret + apiKeySecret)

### Authentication Flow:
1. CLI generates permanent key on first run
2. CLI creates one-time key for session
3. CLI starts tunnel, displays QR code / keys
4. Phone scans QR or enters one-time key
5. Phone connects via WebSocket to tunnel URL
6. Server validates connection via Socket.IO handshake

## 6. Tunnel Flow

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────┐
│  Server     │     │ cloudflared     │     │ Cloudflare   │
│  (port 2208)│───►│ Quick Tunnel    │───►│ Edge Network   │
│             │     │ (outbound)     │     │               │
└─────────────┘     └────────┬────────┘     └──────┬───────┘
                             │                      │
                             │ Tunnel URL            │ HTTPS/WSS
                             ▼                      ▼
                    ┌────────────────────────────────────┐
                    │  Phone/Browser                     │
                    │  wss://<tunnel>.trycloudflare.com   │
                    └────────────────────────────────────┘
```

### Flow Steps:
1. **Tunnel Spawn:** Server spawns `cloudflared tunnel --url http://localhost:2208`
2. **URL Detection:** cloudflared outputs URL (e.g., `https://abcd.trycloudflare.com`)
3. **URL Rotation:** If URL changes, server updates state and notifies clients
4. **Health Check:** Server verifies tunnel URL responds (90s timeout)
5. **Session Creation:** Server POSTs to `https://9remote.cc/api/session/create` with API key
6. **LocalFirst:** If phone is on same LAN, traffic goes local (bypasses tunnel)
7. **Recovery:** Tunnel monitored every 30s; auto-restarts on network change

## 7. Update Flow

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  CLI        │     │ npm Registry │     │ Background   │
│  (agent)    │───►│  (HTTPS)     │───►│ Process      │
└──────┬──────┘     └──────────────┘     └──────┬───────┘
       │                                         │
       │ Check latest version                   │
       │ Download tarball + SHA256              │
       │ Verify integrity                       │
       │ Spawn update script                    │
       │                                        │
       │ 1. Kill stale processes (cloudflared)  │
       │ 2. npm install -g 9remote@latest      │
       │ 3. Verify version match                │
       │ 4. Restart agent                       │
       │                                        │
       │ ◄──── "Update complete"               │
       ▼                                        ▼
```

### Flow Steps:
1. CLI fetches latest version from `https://registry.npmjs.org/9remote/latest`
2. CLI downloads tarball and SHA256 manifest
3. CLI verifies SHA256 checksum of downloaded package
4. CLI spawns a detached background process that:
   - Kills old cloudflared and agent processes
   - Runs `npm install -g 9remote@latest`
   - Verifies the installed version matches expected
   - Rolls back on failure
   - Restarts the agent with `--tray --skip-update --start`
5. CLI exits after spawning the update process

## 8. State Persistence

| State | Storage Location | Format | Persisted By |
|---|---|---|---|
| UI State (step, tunnelUrl, keys) | `~/.9remote/state/state.json` | JSON | Server (`fs`) |
| Settings (theme, etc.) | `~/.9remote/state/settings.json` | JSON | Server (`fs`) |
| Command file | `~/.9remote/state/cmd.json` | JSON | TUI → Server |
| Keys (machineId, key, name) | `~/.9remote/keys.json` | JSON | CLI (`fs`) |
| PTY sessions | In-memory (daemon) | N/A | PtyDaemon |
| Agent logs | `~/.9remote/logs/agent.log` | Text | Server (2KB rotation) |
| PID files | `~/.9remote/pids/{name}.pid` | Text | Various |
| cloudflared PID | `~/.9remote/pids/cloudflared.pid` | Text | Server |
| Agent PID | `~/.9remote/pids/agent.pid` | Text | CLI/Server |

## 9. LocalFirst Adapter

9Remote implements a **LocalFirstAdapter** that races LAN vs tunnel connections:

```
┌─────────────────────────────────────────────────────┐
│  Phone tries both paths simultaneously:             │
│                                                     │
│  LAN path: ws://192.168.x.x:2208 (direct)          │
│  Tunnel path: wss://abcd.trycloudflare.com (relay)  │
│                                                     │
│  Whichever connects first wins → other is discarded │
│  → Local when on same WiFi, tunnel when remote      │
│  → Latency: <5ms local vs ~50ms tunnel              │
└─────────────────────────────────────────────────────┘
```
