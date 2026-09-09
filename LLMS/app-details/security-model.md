# Security Model

> **Purpose:** Document the security architecture, authentication, and threat model of 9Remote.
> **Scope:** Authentication, encryption, access control, data handling, and threat mitigation.
> **Source:** `README.md`, `package/dist/cli.cjs`, `package/dist/server.cjs`

## 1. Overview

9Remote's security model is built around three core principles:

1. **Zero Trust** — Every device must be explicitly approved before accessing the host
2. **No Open Ports** — Uses outbound-only Cloudflare Tunnel, no port forwarding required
3. **Privacy First** — No terminal data, screen data, or files are collected or stored on servers

## 2. Authentication

### 2.1 Permanent Key

Each host generates a **permanent key** based on the machine ID:

```
Process:
1. machine-id = machineIdSync()  (platform-specific: ioreg/reg/machine-id)
2. salt = MACHINE_ID_SALT env or "9remote-salt"
3. key = SHA256(machineId + salt).substring(0, 16)
4. keyId = key.substring(0, 8)
5. random = 4 chars from [a-z0-9]
6. hmac = HMAC-SHA256(apiKeySecret, keyId + random).slice(0, 6)
7. permanentKey = "sk-" + keyId + "-" + random + "-" + hmac
```

**Storage:** `~/.9remote/keys.json` — contains `machineId`, `key` (permanent key), `name`, `createdAt`.

**Note:** The `apiKeySecret` defaults to `"9remote-api-key-secret"` but can be overridden via `API_KEY_SECRET` env var.

### 2.2 One-Time Key (QR Login)

For each session, a **one-time key** is generated:

```
Format: sk-{first8}-{4random}-{6hmac}
Expiry: 30 minutes (timestamp: 1800000ms)
Usage: Single use — once consumed by a device, it's cleared
```

The QR code encodes the URL `https://9remote.cc/login?k={tempKey}`.

### 2.3 Session Creation

When the CLI starts (TUI or UI mode):

1. Generates permanent key from machine ID
2. Creates one-time key with 30-minute expiry
3. Starts Cloudflare tunnel
4. POSTs to `https://9remote.cc/api/session/create` with the permanent API key
5. Displays QR code with one-time key URL

When a phone connects:

1. Phone visits the tunnel URL or scans QR
2. Phone enters/app-uses the one-time key
3. Connection is established via Socket.IO over the tunnel
4. Server receives connection with device metadata

## 3. Authorization (Pair Device)

### 3.1 Pair Device Approval

New devices are **not** automatically connected. The system implements a **Pair Device** approval flow:

1. When a new device connects, the CLI/server displays a prompt:
   ```
   🔔 New Device Connection
   Device: ABCD1234...
   IP: 192.168.1.100
   Allow this device? (y/N)
   ```

2. The user must explicitly type `y` to approve.

3. Approved devices are tracked and can connect without re-approval.

### 3.2 Auto-Approval

Auto-approval can be enabled via:
- TUI menu: `Devices → Auto-approve new devices`
- API: `POST /api/device/auto-approve { "enabled": true }`

When enabled, all new connections are automatically approved without prompting.

### 3.3 Device Management

Devices can be managed from the TUI:
- View connected devices
- Revoke device access
- View pending/approved/rejected devices

## 4. Network Security

### 4.1 Cloudflare Tunnel

- Uses `cloudflared tunnel --url http://localhost:2208`
- **Outbound-only** — no inbound ports opened on the host
- Traffic flows: Phone → Cloudflare Edge → Tunnel → Local Server
- Works behind NAT, corporate firewalls, mobile hotspots, VPNs

### 4.2 Access Control (Server-side)

The server router enforces access control on all `/api/*` routes:

```
Rules:
1. Local requests (127.0.0.1, ::1, ::ffff:127.0.0.1) → ALLOW
2. Cloudflare requests (CF-Connecting-IP header present) → ALLOW
3. All other requests → 403 Forbidden
```

### 4.3 Origin Validation

For sensitive endpoints, the `origin` header is validated against an allowlist:
- `https://9remote.cc`
- `https://*.trycloudflare.com`
- `http://localhost:*`
- `http://127.0.0.1:*`

### 4.4 LocalFirst Adapter

The LocalFirstAdapter races LAN vs tunnel connections:
- If phone and host are on the same WiFi, traffic stays local
- No data leaves the local network for same-WiFi connections
- Cloudflare tunnel acts as fallback when remote

## 5. Data Handling

### 5.1 What Is NOT Collected

- Terminal output is never logged or transmitted to servers
- File contents are never stored on servers
- Screen captures are never sent to servers
- Keystrokes are never intercepted or logged
- One-time keys are **never** stored on servers after the session ends

### 5.2 What Is Collected (Minimal)

- **Connection metadata:** Socket ID, IP address, connection timestamp
- **Device metadata:** Device ID for pairing
- **Error logs:** Crash reports with stack traces (local file only, not transmitted)
- **Update checks:** Version number, platform, architecture

All collected data is used solely for the operation of the service and is not used for analytics or tracking.

## 6. Encryption

| Channel | Protocol | Encryption |
|---------|----------|------------|
| Cloudflare Tunnel | HTTPS/WSS | TLS 1.2+ (Cloudflare-managed) |
| WebRTC Desktop | DTLS-SRTP | End-to-end (WebRTC standard) |
| Socket.IO (local) | WebSocket | None (LAN only, optional) |
| Terminal I/O | WebSocket over tunnel | TLS via Cloudflare |
| cloudflared CLI | HTTPS | TLS 1.2+ |

**Note:** Terminal data is not separately encrypted — it relies on TLS via the Cloudflare tunnel. For same-LAN connections, terminal data is unencrypted WebSocket (local network assumed trusted).

## 7. Key Management

### 7.1 Key Storage

| Key Type | Location | Format | Persistence |
|----------|----------|--------|-------------|
| Permanent Key | `~/.9remote/keys.json` | `sk-{8}-{4}-{6}` | Persistent (survives restarts) |
| One-Time Key | In-memory + state.json | `sk-{8}-{4}-{6}` | Ephemeral (30-min expiry) |
| API Key Secret | `API_KEY_SECRET` env var | String | Configurable at runtime |
| Machine ID Salt | `MACHINE_ID_SALT` env var | String | Configurable at runtime |

### 7.2 Key Rotation

- **One-time keys:** Auto-expire after 30 minutes; can be regenerated from TUI menu: `Keys → Regenerate`
- **Permanent keys:** Tied to machine ID; changing the machine ID or salt requires re-pairing all devices
- **API key secret:** Changing `API_KEY_SECRET` invalidates all existing one-time keys

## 8. Threat Model

### 8.1 Mitigated Threats

| Threat | Mitigation |
|--------|-----------|
| Port scanning | No open ports — Cloudflare tunnel is outbound-only |
| Man-in-the-middle | TLS termination at Cloudflare edge |
| Unauthorized device access | Pair Device approval system |
| Key interception | One-time keys expire in 30 minutes |
| Data exfiltration | No data collected/stored on servers |
| Session hijacking | Socket.IO authentication via permanent key |
| Server compromise | Minimal server-side state; keys are client-side |

### 8.2 Residual Risks

| Risk | Mitigation |
|------|-----------|
| Cloudflare compromise | Data still flows through TLS; local network bypass available |
| Same-LAN attacker | Terminal traffic is unencrypted WebSocket on LAN |
| Physical access to host | Standard OS security (screen lock, etc.) |
| Malicious AI agent | Sandboxed shell sessions; no elevated privileges by default |

## 9. Security Checklist

- [x] No open ports on host machine
- [x] All remote traffic encrypted via TLS
- [x] Zero data collection (terminal, files, screen)
- [x] Pair Device approval for each new device
- [x] One-time keys expire in 30 minutes
- [x] Permanent keys derived from machine ID (non-portable)
- [x] Access control: localhost or CF-Connecting-IP only
- [x] Origin validation on sensitive endpoints
- [x] Log rotation (2KB max, 7-day cleanup)
- [x] PID files use restricted permissions (mode 384 = 0600)
- [x] Unix socket permissions: mode 384 (0600)
- [ ] Terminal traffic encrypted on LAN (relies on network security)
- [ ] Full end-to-end encryption (relies on Cloudflare TLS)
