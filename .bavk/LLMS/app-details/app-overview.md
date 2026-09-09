# Application Overview

> **Purpose:** High-level overview of what 9Remote does, its features, and how it compares to alternatives.
> **Scope:** Product-level understanding for LLMs.
> **Source:** `README.md`, `package/package.json`, `package/README.md`

## 1. What Is 9Remote?

**9Remote** (v2.5.8) is a single-tool solution that gives developers remote access to their Mac, Linux, or Windows machine from **any phone or browser, anywhere**. It combines:

- **Remote Terminal** — Full PTY shell accessible via browser
- **Remote Desktop** — Live screen streaming via WebRTC
- **File Explorer** — Browse, upload, and download files
- **Code Editor** — Built-in editor with syntax highlighting
- **Git Integration** — Run git commands with visual status
- **Local Sites Proxy** — Expose `localhost:3000` to your phone
- **Push Notifications** — Get notified when builds finish
- **AI Integration** — Works with Claude Code, Codex, Cursor, OpenClaw

### The Problem

Remote access today is painful:
- SSH requires firewall rules, port forwarding, SSH keys, IP whitelisting
- VPN is overkill just to check a terminal
- ngrok/tunnels expire and lose connections
- TeamViewer is slow, desktop-only, and paid for commercial use
- Chrome Remote Desktop has no terminal or file explorer
- Termius is SSH-only with no remote desktop or browser access

### The Solution

9Remote provides **one-command setup** with **zero configuration**:

```bash
npm install -g 9remote
9remote
```

It uses **Cloudflare Quick Tunnel** (outbound-only, no port forwarding) and **QR code pairing** for instant, secure access.

## 2. Key Features Matrix

| Feature | What It Does | Why It Matters |
|---------|-------------|-----------------|
| Remote Terminal | Full PTY shell via WebSocket | Code like you're sitting at your Mac |
| Remote Desktop | Live screen streaming via WebRTC | View & control your machine from phone |
| File Explorer | Browse, upload, download files | Manage files without SSH/SFTP |
| Code Editor | Built-in editor with syntax highlighting | Quick edits without opening IDE |
| Git Integration | Run git commands with visual status | Commit/push from your phone |
| Mobile Optimized | Touch-friendly UI, gesture controls | Full workspace on a 6" screen |
| QR Login | One-time 30-min key, scan to connect | Zero-friction mobile access |
| Auto Tunnel | Cloudflare tunnel, no port forwarding | Works behind any NAT/firewall |
| Persistent Sessions | PTY daemon survives restarts | Long-running commands stay alive |
| Multi-Device Sync | Same session across phone/tablet/laptop | Switch devices without losing context |
| Push Notifications | Build finished? Get notified | Never miss a critical event |
| AI Integration | Works with Claude Code, Codex, OpenClaw | Code with AI from anywhere |
| Local Sites Proxy | Expose `localhost:3000` to phone | Test dev servers on mobile instantly |
| Low Latency | <50ms typical, WebRTC for desktop | Feels like local |
| Pair Device | Approve each device before it connects | No unauthorized access, full control |
| No Account Required | Machine ID + QR key, zero signup | Privacy-first design |

## 3. Use Cases

### Case 1: "Code from bed"
It's 11 PM, you remember a bug but laptop is in another room.
1. Open 9remote app on phone
2. Scan QR (or use saved session)
3. Open terminal → fix bug → git push
4. Sleep well 😴

### Case 2: "Fix bugs at a cafe"
Production is down. You only have your phone and a bad café Wi-Fi.
1. Connect to your home/office Mac via 9remote
2. Tail logs in terminal
3. Edit config in built-in editor
4. Deploy → crisis averted

### Case 3: "Deploy while on vacation"
Client needs a hotfix. You're on the beach.
1. Phone → 9remote → your dev machine
2. git pull → build → deploy
3. Back to the beach in 5 minutes 🏖️

### Case 4: "On-call engineer"
PagerDuty alert at 3 AM. Don't want to power on laptop.
1. Push notification → tap → 9remote opens
2. Terminal + remote desktop ready
3. Diagnose + restart service from bed

## 4. CLI Commands

| Command | Description |
|---------|-------------|
| `9remote` | TUI mode — interactive menu with QR code |
| `9remote ui` | Web UI mode — opens browser dashboard at `localhost:2208` |

### Environment Variables

| Variable | Description | Default |
|---------|-------------|---------|
| `PORT` | Override server port | `2208` |
| `NREMOTE_REGISTRY` | NPM registry URL | `https://registry.npmjs.org/9remote/latest` |
| `NREMOTE_NPM_CLI` | Use custom npm CLI for updates | (unset) |
| `MACHINE_ID_SALT` | Salt for machine ID hashing | `9remote-salt` |
| `API_KEY_SECRET` | Secret for API key HMAC | `9remote-api-key-secret` |
| `NODE_ENV` | `development` enables debug logging | (unset) |
| `AGENT_DEBUG` | Set to `1` to enable debug logging | (unset) |

## 5. Comparison With Alternatives

| Feature | **9Remote** | Claude Remote | TeamViewer | Chrome Remote | Termius |
|---------|:-----------:|:-------------:|:----------:|:-------------:|:-------:|
| Zero Config | ✅ | ✅ | ✅ | ✅ | ❌ |
| Terminal Access | ✅ | ✅ | ❌ | ❌ | ✅ |
| Remote Desktop | ✅ | ❌ | ✅ | ✅ | ❌ |
| File Explorer | ✅ | ❌ | ✅ | ❌ | ✅ |
| Code Editor | ✅ | ❌ | ❌ | ❌ | ❌ |
| Git Integration | ✅ | ❌ | ❌ | ❌ | ❌ |
| Mobile Optimized | ✅ | ✅ | ❌ | ❌ | ✅ |
| Browser-Based | ✅ | ✅ | ❌ | ✅ | ❌ |
| QR Login | ✅ | ✅ | ❌ | ❌ | ❌ |
| Auto Tunnel | ✅ | ✅ | ✅ | ✅ | ❌ |
| Persistent Sessions | ✅ | ✅ | ❌ | ❌ | ✅ |
| Multi-Device Sync | ✅ | ✅ | ✅ | ❌ | ✅ |
| Push Notifications | ✅ | ✅ | ❌ | ❌ | ❌ |
| AI Integration | ✅ | ✅ | ❌ | ❌ | ❌ |
| No Port Forwarding | ✅ | ✅ | ✅ | ✅ | ❌ |
| No Account Required | ✅ | ❌ | ❌ | ❌ | ❌ |
| **TOTAL** | **16 / 16** | 11 / 16 | 7 / 16 | 5 / 16 | 7 / 16 |

## 6. Licensing

- **Published package:** MIT License
- **Source code:** Currently proprietary during development phase
- **Open-source milestone:** Will be fully open-sourced under MIT once enough GitHub stars are reached

## 7. Key Numbers

- **Default port:** 2208
- **Vite dev port:** 5173
- **Worker URL:** `https://9remote.cc`
- **Session key length:** 16 characters (first 8 of SHA256)
- **One-time key expiry:** 30 minutes
- **Tunnel timeout:** 90 seconds (health check)
- **Tunnel retry backoff:** exponential, base 2s, max 5 min
- **Log rotation:** 2KB max, 7-day cleanup
- **Update check retry:** exponential, base 1s, max 60s
