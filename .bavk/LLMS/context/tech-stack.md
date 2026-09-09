# Technology Stack

> **Purpose:** Detailed breakdown of all technologies used in the 9Remote project.
> **Scope:** Dependencies, build tools, and runtime technologies.
> **Source:** `package/package.json`, `README.md`

## 1. Overview

9Remote is a Node.js-based application that uses a polyglot technology stack spanning desktop, web, and cloud infrastructure. It combines native modules for system access, WebAssembly/WebRTC for real-time communication, and a modern web framework for the UI.

## 2. Runtime

| Technology | Version | Usage |
|---|---|---|
| **Node.js** | 20+ | Primary runtime for CLI, server, and daemon |
| **npm** | 10+ | Package management and CLI distribution |

## 3. Core Libraries

### Server & CLI

| Library | Version | Purpose |
|---|---|---|
| **socket.io** | ^4.8.3 | Real-time WebSocket communication for terminal I/O |
| **chalk** | ^5.4.1 | Terminal text styling for TUI output |
| **ora** | ^8.1.1 | Terminal spinner animations |
| **inquirer** | ^9.3.8 | Interactive TUI prompts and menus |
| **qrcode** | ^1.5.4 | QR code generation for device pairing |
| **qrcode-terminal** | ^0.12.0 | ASCII QR codes for terminal display |
| **preact** | ^10.26.2 | Lightweight React-compatible UI framework (embedded dashboard) |
| **chokidar** | ^3.6.0 | File system watching (config changes, update checks) |
| **sharp** | 0.33.5 | Image processing for screenshots/desktop capture |
| **http-proxy** | ^1.18.1 | Proxying local dev sites through the tunnel |
| **multicast-dns** | ^7.2.5 | Local device discovery on LAN |
| **node-screenshots** | ^0.2.8 | Screen capture for remote desktop |
| **node-machine-id** | ^1.1.12 | Unique machine identifier for permanent keys |
| **archiver** | ^5.3.2 | Archive creation (update packages) |
| **web-push** | ^3.6.7 | Push notifications for build/deploy alerts |
| **edge-tts-node** | ^1.5.7 | Text-to-speech for notifications |
| **koffi** | ^2.16.3 | FFI for calling native C libraries (macOS display services) |
| **https-proxy-agent** | ^7.0.0 | HTTP(S) proxy support |

### Optional / Native Dependencies

| Library | Version | Platform | Purpose |
|---|---|---|---|
| **node-pty** | 1.2.0-beta.12 | All | PTY session management for terminal emulators |
| **node-datachannel** | 0.32.3 | All | WebRTC implementation for desktop streaming |
| **@hurdlegroup/robotjs** | ^0.12.1 | All | Desktop input control (mouse/keyboard) and screen capture |
| **@julusian/jpeg-turbo** | ^3.0.1 | All | Fast JPEG encoding for screen frames |

### Build Tools & Development

| Tool | Version | Purpose |
|---|---|---|
| **vite** | ^6.2.2 | Build system for the Preact web UI |
| **tailwindcss** | ^3.4.17 | CSS utility framework |
| **postcss** | ^8.5.3 | CSS processing |
| **autoprefixer** | ^10.4.21 | CSS vendor prefixing |
| **@preact/preset-vite** | ^2.9.5 | Vite plugin for Preact |
| **vite-plugin-javascript-obfuscator** | ^3.1.0 | JS obfuscation for production builds |
| **nodemon** | ^3.1.11 | Development auto-restart |
| **npm-run-all** | ^4.1.5 | Parallel/concurrent script runner |
| **wait-on** | ^9.0.4 | Wait for server to be ready in dev mode |

## 4. Cloud Infrastructure

| Service | Provider | Usage |
|---|---|---|
| **Cloudflare Tunnel** | Cloudflare | Zero-config secure tunnel (cloudflared) |
| **Cloudflare Workers** | Cloudflare | Edge API for session management + TURN credentials |
| **Cloudflare R2** | Cloudflare | Object storage (update packages, assets) |

## 5. Third-Party Integrations

| Service | Purpose |
|---|---|
| **npm Registry** | Package distribution and updates |
| **Claude Code** | AI coding agent integration |
| **OpenAI Codex** | AI coding agent integration |
| **Cursor** | AI editor integration |
| **OpenClaw** | AI coding agent integration |

## 6. Build & Distribution

- **Package format:** CommonJS (`.cjs` compiled bundles)
- **Package type:** `module` in package.json but uses `.cjs` extensions
- **NPM package:** `9remote` published to npm registry
- **Post-install:** `install.cjs` auto-installs system tray dependencies
- **UI build:** Vite bundles Preact app → `ui/assets/index-*.{js,css}`
- **UI framework:** Preact 10.x with JSX, compiled by Vite

## 7. System Dependencies

### External binaries (auto-managed)
- **cloudflared** — Downloaded and cached on first run for tunneling
- **systray2** — Installed in `~/.9remote/runtime/` for system tray

### OS-level permissions (macOS)
- **Screen Recording** — Required for remote desktop capture
- **Accessibility** — Required for input control (mouse/keyboard injection)

## 8. UI Technology Details

### Embedded Dashboard (Preact)
- **Framework:** Preact 10.x (React-compatible, ~3KB)
- **Build:** Vite 6.x with JavaScript obfuscation in production
- **Styling:** Tailwind CSS 3.x
- **Icons:** Material Symbols (Google Fonts)
- **Fonts:** Sora (headings) + JetBrains Mono (code)
- **Theme:** Dark/light mode, persisted in `localStorage` as `9remote-theme`

### Web Client (Phone/Browser)
- **Framework:** Next.js 16 + React 19
- **Styling:** Tailwind CSS 4
- **Deployment:** 9remote.cc (Cloudflare)
