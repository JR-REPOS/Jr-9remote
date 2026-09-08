# Constants and Defaults

> **Purpose:** Document all hardcoded constants, timeouts, and default values used by 9Remote.
> **Scope:** Key thresholds, timeouts, retry policies, and default configuration values.
> **Source:** `package/dist/cli.cjs`, `package/dist/server.cjs`, `package/dist/ptyDaemon.cjs`

## 1. Overview

This document lists all hardcoded constants, default values, timeout durations, retry policies, and threshold values used throughout the 9Remote codebase. These values are critical for understanding system behavior, troubleshooting performance issues, and tuning for specific environments.

## 2. Network & Port Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `DEFAULT_PORT` | `2208` | Default HTTP server port |
| `VITE_DEV_PORT` | `5173` | Vite development server port |
| `PROXY_PORTS` | `[2208, 5173]` | Ports considered "local" for proxy URLs |
| `LOCALHOST_ADDRESSES` | `["localhost", "127.0.0.1"]` | Addresses treated as local access |

## 3. Tunnel Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `TUNNEL_SPAWN_TIMEOUT` | `90000` (90s) | Max time to wait for cloudflared to produce a tunnel URL |
| `HEALTH_CHECK_TIMEOUT` | `5000` (5s) | HTTP health check timeout for tunnel URL |
| `SERVER_BOOT_DELAY` | `2000` (2s) | Delay after starting server before tunnel spawn |
| `POST_READY_HOLD_MS` | `2000` (2s) | Hold time after tunnel is ready |
| `BG_SERVER_READY_TIMEOUT` | `15000` (15s) | Max wait for server in background mode |
| `BG_SERVER_READY_INTERVAL` | `300` (ms) | Poll interval for background server health |
| `BG_SPAWN_FLUSH_MS` | `400` (ms) | Delay after bg spawn before exit |

## 4. Retry Policies (Exponential Backoff)

| Policy | Strategy | Base | Max | Use Case |
|--------|----------|------|-----|----------|
| `server` | `exp` | 1000ms | 60000ms (1min) | General server operations |
| `tunnelRestart` | `exp` | 2000ms | 300000ms (5min) | Tunnel restart after URL rotation |
| `tunnelSpawn` | `linear` | 5000ms | 60000ms (1min) | Initial tunnel spawn |
| `tunnelRateLimit` | `exp` | 60000ms | 900000ms (15min) | Rate limiting after repeated failures |
| `internet` | `linear` | 3000ms | 60000ms (1min) | Internet connectivity checks |
| `sse` | `linear` | 2000ms | 2000ms | SSE reconnection |
| `urlSync` | `linear` | 5000ms | 60000ms (1min) | Tunnel URL synchronization |
| `updateRetry` | `exp` | 1000ms | 60000ms (1min) | Update version check |
| `checkInterval` | `exp` | 1000ms | 60000ms (1min) | Internet connectivity retry |

**Exponential formula:** `base * 2^(attempt-1)`, capped at `max`
**Linear formula:** `base * attempt`, capped at `max`

## 5. Check Intervals

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `INTERNET_CHECK_INTERVAL` | `15000` (15s) | Internet connectivity check interval |
| `INTERNET_CHECK_TIMEOUT` | `3000` (3s) | Internet check timeout |
| `NETWORK_POLL_INTERVAL` | `2000` (2s) | Network change detection interval |
| `CMD_POLL_INTERVAL` | `1000` (1s) | Command file (cmd.json) poll interval |
| `SERVER_READY_INTERVAL` | `200` (ms) | Server ready check interval (TUI mode) |
| `BG_SERVER_READY_INTERVAL` | `300` (ms) | Server ready interval (bg mode) |
| `UPDATE_CHECK_INTERVAL` | `3600000` (1hr) | Update check interval (hourly) |
| `SLEEP_INHIBIT_CHECK` | `30000` (30s) | Sleep inhibit verification interval |

## 6. Session & PTY Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `PTY_PROTOCOL_VERSION` | `"46"` | PtyDaemon protocol version string |
| `PTY_DEFAULT_COLS` | `80` | Default terminal columns |
| `PTY_DEFAULT_ROWS` | `24` | Default terminal rows |
| `PTY_FLUSH_DELAY` | `setImmediate` | Output flush scheduling (deferred) |
| `PTY_BUFFER_THRESHOLD` | `4096` | Buffer size before truncation (server-side) |
| `PTY_OUTPUT_BUFFER_LIMIT` | `4096` | Old output buffer retained (daemon-side) |

## 7. Key Management Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `ONE_TIME_KEY_TTL` | `1800000` (30min) | One-time key expiry |
| `PERMANENT_KEY_LENGTH` | `16` | Machine-derived key length |
| `ONE_TIME_KEY_RANDOM_LEN` | `4` | Random characters in one-time key |
| `ONE_TIME_KEY_HMAC_LEN` | `6` | HMAC characters in key |
| `KEY_FORMAT` | `sk-{8}-{4}-{6}` | Key format pattern |

## 8. Display & TUI Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `HEADER_WIDTH` | `44` | TUI header box width |
| `MAX_LOG_LINES` | `200` | Max log entries in memory |
| `SPINNER_INTERVAL` | `80` (ms) | TUI spinner animation interval (non-Windows) |
| `SPINNER_INTERVAL_WIN` | `120` (ms) | TUI spinner interval (Windows) |
| `TUNNEL_STATUS_INDICATOR` | `["⣾","⣽","�","⢿","⡿","⣟","⣻","⣽"]` | Spinner frames |

## 9. Log Management Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `LOG_MAX_BYTES` | `2048` (2KB) | Max log file size before rotation |
| `LOG_CLEANUP_DAYS` | `7` | Log file cleanup age |
| `LOG_MAX_ENTRIES_SSE` | `80` | Max log entries sent via SSE |
| `LOG_RENAME_THRESHOLD` | `2KB` | When log is renamed to `.1` |

## 10. Permission Polling Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `PERMISSION_POLL_MS` | `10000` (10s) | macOS permission polling interval |
| `PERMISSION_VERIFY_TIMEOUT` | `15000` (15s) | Permission verification timeout |
| `MAX_PERMISSION_RETRIES` | `3` | Max permission check retries |

## 11. Update Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `UPDATE_MAX_RETRY` | `3` | Max update install retries |
| `UPDATE_RETRY_DELAY` | `3000` (3s) | Delay between update retries |
| `UPDATE_DOWNLOAD_TIMEOUT` | `120000` (2min) | Package download timeout |
| `UPDATE_INSTALL_TIMEOUT` | `120000` (2min) | npm install timeout |
| `UPDATE_VERIFY_TIMEOUT` | `5000` (5s) | Version verification timeout |

## 12. Cloudflare Tunnel Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `CLOUDFLARE_URL_PATTERN` | `/https:\/\/([a-z0-9-]+)\.trycloudflare\.com/gi` | Regex for tunnel URL extraction |
| `CLOUDFLARE_EDGE_PROBE_INTERVAL` | `30000` (30s) | Edge connection probe interval |
| `CLOUDFLARE_EDGE_PROBE_FAILURE_THRESHOLD` | `3` | Consecutive failures before restart |
| `CLOUDFLARE_SLEEP_WAKE_GRACE` | `120000` (2min) | Grace period after sleep/wake before probing |
| `CLOUDFLARE_URL_ROTATION_THRESHOLD` | `30000` (30s) | Min interval between URL rotation handling |

## 13. LocalFirst Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `LOCAL_FIRST_ACTIVE_TIMEOUT` | `400` (ms) | Active desktop frame interval (ms) |
| `LOCAL_FIRST_IDLE_TIMEOUT` | `60` (ms) | Idle desktop frame interval (ms) |
| `LOCAL_FIRST_MAX_LATENCY` | `50` (ms) | Target for local connection latency |

## 14. Desktop Capture Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `DESKTOP_CAPTURE_INTERVAL_ACTIVE` | `400` (ms) | Active screen capture interval |
| `DESKTOP_CAPTURE_INTERVAL_IDLE` | `60` (ms) | Idle screen capture interval |
| `DESKTOP_TILE_SIZE` | `64` | Tile width for diff-based capture |
| `DESKTOP_JPEG_QUALITY` | `80` | JPEG quality for screen frames |

## 15. API & Worker Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `WORKER_URL` | `https://9remote.cc` | Default Cloudflare Worker URL |
| `SESSION_CREATE_PATH` | `/api/session/create` | Session creation endpoint |
| `LOGIN_PATH` | `/login` | Device login path |
| `DNS_SERVERS` | `["1.1.1.1", "1.0.0.1", "8.8.8.8"]` | Custom DNS resolver servers |
| `FETCH_TIMEOUT` | `5000` (5s) | Fetch timeout for API calls |

## 16. File System Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `FILE_MODE_PRIVATE` | `384` (0o600) | Private file permission (owner read/write only) |
| `SOCKET_MODE` | `384` (0o600) | Unix socket permission |
| `DIR_MODE` | `493` (0o755) | Directory permission |
| `PID_FILE` | `{name}.pid` | PID file naming pattern |

## 17. Security Constants

| Constant | Default Value | Description |
|----------|--------------|-------------|
| `PERMANENT_KEY_SALT` | `9remote-salt` | Salt for machine ID hashing |
| `API_KEY_SECRET_DEFAULT` | `9remote-api-key-secret` | Default HMAC secret |
| `ORIGIN_ALLOWLIST` | (see server) | Allowed origins for API requests |
| `TOKEN_EXPIRY_WINDOW` | `1800000` (30min) | Same as one-time key TTL |

## 18. Color Palette (TUI)

| Color | ANSI Code | Usage |
|-------|-----------|-------|
| Orange | `\x1B[38;2;230;138;110m` | TUI borders, highlights |
| Orange Dim | `\x1B[38;2;200;120;95m` | Dimmed text |
| Green | `\x1B[32m` | Success, step complete |
| Red | `\x1B[31m` | Errors, failures |
| Yellow | `\x1B[33m` | Warnings |
| Cyan | `\x1B[36m` | Info, URLs |
| White | `\x1B[37m` | Default text |
| Dim | `\x1B[2m` | Secondary text |
| Bold | `\x1B[1m` | Headings |
| Reset | `\x1B[0m` | Reset styling |
