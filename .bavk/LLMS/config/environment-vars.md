# Environment Variables

> **Purpose:** Complete reference of all environment variables used by 9Remote.
> **Scope:** All env vars read by the CLI, server, and daemon.
> **Source:** `package/dist/cli.cjs`, `package/dist/server.cjs`, `package/dist/ptyDaemon.cjs`

## 1. Overview

9Remote reads several environment variables to configure behavior, override defaults, and enable debugging. These can be set in the shell before running `9remote`, or in a `.env`-style file.

## 2. Environment Variables

### 2.1 Server Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `2208` | HTTP server port. Override if 2208 is already in use. Example: `PORT=3308 9remote` |
| `NREMOTE_WORKER_URL` | `https://9remote.cc` | Base URL for the Cloudflare Worker (session management API). Override for self-hosting. |
| `NREMOTE_REGISTRY` | `https://registry.npmjs.org/9remote/latest` | NPM registry URL for update checks. Override for private registries. |
| `NREMOTE_NPM_CLI` | (unset) | If set, tells update script to use a custom npm CLI path. |

### 2.2 Key Management

| Variable | Default | Description |
|----------|---------|-------------|
| `MACHINE_ID_SALT` | `9remote-salt` | Salt appended to machine ID when generating the permanent key. Changing this invalidates all existing device pairing. |
| `API_KEY_SECRET` | `9remote-api-key-secret` | Secret used for HMAC computation in key generation. Changing this invalidates all one-time keys. |

### 2.3 Debugging & Development

| Variable | Default | Description |
|----------|---------|-------------|
| `NODE_ENV` | (unset) | Set to `development` to enable debug logging and additional error output. |
| `AGENT_DEBUG` | (unset) | Set to `1` to enable debug-level logging (same as `NODE_ENV=development`). |

### 2.4 Display & Desktop

| Variable | Default | Description |
|----------|---------|-------------|
| `DISPLAY` | (system) | X11 display for Linux screenshot capture. Required for remote desktop on Linux. |

### 2.5 Internal / Process

| Variable | Default | Description |
|----------|---------|-------------|
| `ELECTRON_RUN_AS_NODE` | (unset) | Set to `1` when running under Electron (auto-set by `cli.cjs`). |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | `1` | Disables alternate screen buffer for Claude Code compatibility. Set by CLI before spawning shell sessions. |
| `HOME` | (system) | User home directory. Used by autostart scripts and config paths. |
| `PATH` | (system) | Used for finding `cloudflared`, `npm`, `node`, and shell binaries. Extended by autostart scripts. |

## 3. Variable Usage in Code

### 3.1 PORT (cli.cjs)

```javascript
var _ = 2208, To = process.env.PORT; // (not directly used, but server reads from state)
// Server listens on state.port (default 2208)
```

In the server, the port is managed via state:
```javascript
const port = state.port || 2208;
httpServer.listen(port);
```

### 3.2 NREMOTE_WORKER_URL (server.cjs)

```javascript
const C = process.env.NREMOTE_WORKER_URL || 'https://9remote.cc';
// Used for:
// POST {C}/api/session/create
// GET {C}/login?k={tempKey} (QR code URL)
```

### 3.3 NREMOTE_REGISTRY (cli.cjs)

```javascript
const Ar = process.env.NREMOTE_REGISTRY || `https://registry.npmjs.org/9remote/latest`;
// Used by update checker to fetch latest version
```

### 3.4 MACHINE_ID_SALT (cli.cjs)

```javascript
async function xt(salt = null) {
  const s = salt || process.env.MACHINE_ID_SALT || '9remote-salt';
  const machineId = machineIdSync();
  return createHash('sha256').update(machineId + s).digest('hex').substring(0, 16);
}
```

### 3.5 API_KEY_SECRET (cli.cjs)

```javascript
const wa = process.env.API_KEY_SECRET || '9remote-api-key-secret';
function Ea(e, n) {
  return createHmac('sha256', wa).update(e + n).digest('hex').slice(0, 6);
}
```

## 4. Setting Environment Variables

### 4.1 Temporary (single command)

```bash
NODE_ENV=development 9remote ui
PORT=3308 9remote
```

### 4.2 Permanent (shell profile)

Add to `~/.bashrc`, `~/.zshrc`, or `~/.profile`:

```bash
# Use a custom Cloudflare Worker URL
export NREMOTE_WORKER_URL="https://my-worker.example.com"

# Use a custom NPM registry
export NREMOTE_REGISTRY="https://npm.mycompany.com/9remote/latest"
```

### 4.3 .env File Support

9Remote does not have built-in `.env` file support. However, the server reads `.env`-style files from known locations:

```javascript
// Reads from:
// ~/.9remote/.env
// ~/.9remote/.env.local
// (parsed for KEY=value pairs, ignores comments starting with #)
```

## 5. Debug Mode

Enabling debug mode provides verbose logging:

```bash
# Method 1: NODE_ENV
NODE_ENV=development 9remote ui

# Method 2: AGENT_DEBUG
AGENT_DEBUG=1 9remote

# Logs will show:
# [debug] tunnelUrl set
# [debug] edge probe ...
# [debug] network change ...
```

Debug output includes:
- Tunnel URL changes
- Network change detection
- Edge probe results
- Reconnection attempts
- SSE event broadcasts
- Socket.IO connection lifecycle
