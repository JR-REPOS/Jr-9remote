# How to Install, Launch, Preview, and Build 9Remote

This guide provides step-by-step instructions for installing, launching, previewing, and building the 9Remote application from source or pre-compiled packages.

---

## 📋 Prerequisites

Before starting, make sure your system meets the following requirements:

- **Node.js**: `v18.0.0` or higher (Node.js 20+ recommended)
- **Package Manager**: `npm` (v9+) or `yarn` / `pnpm`
- **Operating System**: macOS, Linux, or Windows (Windows 10/11 or Windows Server)
- **Optional Dependencies**:
  - `cloudflared` (automatically downloaded if missing when remote tunnel is enabled)
  - `build-essential` / `gcc` / `g++` / `make` (if building native modules like `node-pty` or `robotjs`)

---

## 📥 1. Installation

You can install and set up 9Remote either via `npm` or from this repository.

### Option A: Install via NPM (Global CLI)
```bash
# Install globally from NPM
npm install -g 9remote

# Verify installation
9remote help
```

### Option B: Local Repository / Standalone Setup
If you cloned this repository or extracted the release package:

```bash
# Navigate to the package directory
cd package

# Install dependencies
npm install
```

> **Note**: During `npm install`, the postinstall script (`dist/install.cjs`) runs automatically to configure platform-specific helpers.

---

## 🚀 2. Launching 9Remote

9Remote can be launched in multiple ways depending on your use case:

### Interactive TUI Menu
To start the interactive command-line interface:
```bash
# Global installation
9remote

# Local package directory
node dist/cli.cjs
```

### Foreground Headless Server
To run the server and remote tunnel directly in the foreground:
```bash
# Global installation
9remote start

# Local package directory
node dist/cli.cjs start
```

### Direct Server Launch
To start the core HTTP/WebSocket server directly:
```bash
# Local package directory
node dist/server.cjs
```

---

## 👁️ 3. Previewing the App

Once launched, 9Remote provides both a local web interface and a secure remote access URL.

### Local Web Dashboard
Open your browser and navigate to:
```text
http://localhost:2208
```
*(Default port is `2208`)*

### Remote Access (Cloudflare Tunnel)
When `9remote start` or the background server is running:
1. The terminal displays a QR code and a remote access URL (e.g. `https://9remote.cc/login?k=...` or a `*.trycloudflare.com` tunnel URL).
2. Scan the QR code with your mobile device or open the link on any browser to pair and access your terminal and remote desktop.

### Connection Keys & Device Approval
- **View Pairing Key & URL**:
  ```bash
  9remote key
  ```
- **Create One-Time Access Key**:
  ```bash
  9remote otk
  ```
- **Manage Approved Devices**:
  ```bash
  9remote devices
  9remote auto-approve <on|off>
  ```

---

## 🛠️ 4. Building the App & UI

If you want to customize or rebuild the Web UI assets:

### Build UI Assets
The web interface source is built using Vite into `dist/ui`:
```bash
cd package
npm run build:ui
```

### Development Mode with Live Reload
To run the UI in development mode with Vite hot-reloading:
```bash
cd package
npm run dev:ui
```

---

## ⚙️ 5. Key Environment Variables (Recommended)

9Remote supports several environment variables to customize server port, debugging, and worker URLs:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `PORT` | `2208` | Port for the HTTP and WebSocket server. Example: `PORT=3308 9remote start` |
| `NODE_ENV` | *(unset)* | Set to `development` to enable debug logging and verbose output. |
| `AGENT_DEBUG` | *(unset)* | Set to `1` to enable debug-level logging. |
| `NREMOTE_WORKER_URL` | `https://9remote.cc` | Worker base URL for session pairing API (useful for self-hosted workers). |

### Example usage:
```bash
# Run server on a custom port with debug logging enabled
PORT=3000 NODE_ENV=development 9remote start
```

---

## ❓ Troubleshooting & Useful Links

- **Port Conflict**: If port 2208 is occupied, pass `PORT=xxxx` or change the port settings in `~/.9remote/state/settings.json`.
- **Permissions**: On Linux/macOS, ensure your user has appropriate terminal permissions for PTY allocation.
- **Documentation**: For full documentation, visit [docs.9remote.cc](https://docs.9remote.cc).
