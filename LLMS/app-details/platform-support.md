# Platform Support

> **Purpose:** Platform and OS support matrix for 9Remote.
> **Scope:** Host platforms, client platforms, and platform-specific features.
> **Source:** `README.md`, `package/dist/cli.cjs`, `package/dist/server.cjs`

## 1. Host Platform Support

The **host** is where the 9Remote agent (CLI + server + daemon) runs.

| Platform | Architecture | Status | Notes |
|----------|------------|--------|-------|
| **macOS** | Intel (x64) | ✅ Full | Screen Recording + Accessibility permissions |
| **macOS** | Apple Silicon (arm64) | ✅ Full | Native ARM support via Node.js |
| **Linux** | x64 | ✅ Full | Uses xclip for clipboard, systemd-inhibit for sleep |
| **Linux** | arm64 | ✅ Full | Tested on Raspberry Pi, ARM servers |
| **Windows** | x64 | ✅ Full | Uses PowerShell for clipboard, PowerCreateRequest for sleep |
| **Windows** | arm64 | ⚠️ Untested | May work but not officially tested |
| FreeBSD | — | ❌ Not supported | Machine ID detection throws "Unsupported platform" |

### macOS-Specific Features

1. **Screen Recording Permission** — Required for remote desktop capture
   - Path: `System Settings → Privacy & Security → Screen Recording`
   - Prompted via `cr()` function
   - Can be re-requested via `/api/permissions` with `type: "screenRecording"`

2. **Accessibility Permission** — Required for input control (mouse/keyboard)
   - Path: `System Settings → Privacy & Security → Accessibility`
   - Prompted via `cr()` function
   - Can be re-requested via `/api/permissions` with `type: "accessibility"`

3. **Autostart** — LaunchAgent plist
   - Path: `~/Library/LaunchAgents/cc.9remote.agent.plist`
   - Auto-starts on login

4. **Sleep Inhibition** — `caffeinate -imsd -w <pid>`
   - Prevents system sleep when remote sessions are active

5. **GPU Acceleration** — vImage framework for screen scaling
   - `/System/Library/Frameworks/Accelerate.framework/Accelerate`
   - Used for bilinear/batch image resizing on macOS

### Linux-Specific Features

1. **Clipboard** — Uses `xclip` for clipboard read/write
   - Read: `xclip -selection clipboard -o`
   - Write: `xclip -selection clipboard -t {image/png|text/uri-list}`

2. **Sleep Inhibition** — `systemd-inhibit`
   - Command: `systemd-inhibit --what=idle:sleep:handle-lid-switch --who=9remote --why=remote-active sleep infinity`

3. **Autostart** — systemd user service OR XDG autostart
   - Systemd: `~/.config/systemd/user/cc.9remote.agent.service`
   - XDG: `~/.config/autostart/cc.9remote.agent.desktop`

4. **Screen Capture** — Uses headless capture (x11/wayland)
   - `node-screenshots` handles platform abstraction

5. **Process Management** — Uses `kill`, `ps`, `lsof` for process discovery and management

### Windows-Specific Features

1. **Screen Capture** — Uses `node-screenshots` with Windows Graphics Capture API
   - Headless capture on Windows 10+

2. **GPU Acceleration** — OpenCL for screen scaling
   - Loads `C:/Windows/System32/opencl.dll`
   - Uses `clEnqueueNDRangeKernel` for parallel resize

3. **Clipboard** — Uses PowerShell / .NET `System.Windows.Forms`
   - Read: `powershell.exe -Command "Get-Clipboard -Raw"`
   - Write image: `Add-Type -AssemblyName System.Windows.Forms,System.Drawing`
   - Write file: `Set-Clipboard -Path <path>`

4. **Sleep Inhibition** — `PowerCreateRequest` / `PowerSetRequest` API (via `eQ()` function)
   - Custom C# code compiled at runtime via PowerShell
   - Prevents sleep with "9remote" power request

5. **Autostart** — VBS script in Startup folder
   - Path: `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\9Remote.vbs`

6. **System Tray** — Uses PowerShell tray bridge
   - `tray.ps1` runs as a PowerShell process
   - Tray icon: `trayIcon.ico`
   - Windows elevation: `desktop-elevate.cs` (C# compiled with `csc`/`dotnet`)

7. **Desktop Bridge** — For elevated desktop operations
   - `desktop-bridge.cs` compiled at runtime
   - Handles privilege escalation for desktop capture

## 2. Client Platform Support

The **client** is what connects to the host from a phone or browser.

| Client | Status | Notes |
|--------|--------|-------|
| **Modern Browser** | ✅ Full | Chrome, Safari, Firefox, Edge — full feature set |
| **iOS 14+** | ✅ Full | App Store app, or mobile browser |
| **Android 8+** | ✅ Full | Google Play app, or mobile browser |
| **Tauri Desktop** | ✅ Full | macOS, Windows, Linux native app |

## 3. Cross-Platform Abstraction Layers

The codebase uses several abstraction patterns to handle platform differences:

| Feature | macOS | Linux | Windows |
|---------|-------|-------|---------|
| Machine ID | `ioreg -rd1 -c IOPlatformExpertDevice` | `/var/lib/dbus/machine-id` or `/etc/machine-id` or `hostname` | `REG.exe QUERY HKLM\SOFTWARE\Microsoft\Cryptography /v MachineGuid` |
| Clipboard get | `pbpaste` | `xclip -selection clipboard -o` | `powershell.exe Get-Clipboard -Raw` |
| Clipboard set (text) | `pbcopy` | `xclip -selection clipboard` | `powershell.exe Set-Clipboard` |
| Clipboard set (image) | `osascript` | `xclip -t image/png` | PowerShell `.NET Bitmap` |
| Sleep inhibitor | `caffeinate` | `systemd-inhibit` | `eQ()`: `PowerCreateRequest` / `PowerSetRequest` |
| Autostart | LaunchAgent plist | systemd / XDG autostart | Startup VBS |
| System tray | `systray2` native binary | `systray2` native binary | PowerShell tray bridge |
| Process kill | `process.kill(SIGKILL)` / `lsof -ti:PORT` | `process.kill(SIGKILL)` / `lsof -ti:PORT` | `powershell.exe taskkill /F /T` / `netstat -aon` |
| Network interfaces | `os.networkInterfaces()` with utun/awdl filtering | Same | Same |
| PATH resolution | Standard | Standard | Standard + PATH env expansion |

## 4. Node.js Native Module Compatibility

| Module | Platform Support |
|--------|-----------------|
| `node-pty` | ✅ macOS, Linux, Windows (uses ConPTY on Windows) |
| `node-datachannel` | ✅ All platforms (WebRTC) |
| `@hurdlegroup/robotjs` | ✅ All platforms (screen capture, input control) |
| `@julusian/jpeg-turbo` | ✅ All platforms (JPEG encoding) |
| `koffi` | ✅ macOS, Linux (FFI — used for macOS Accelerate framework) |
| `systray2` | ✅ macOS, Linux (native binary); Windows via PowerShell |
| `node-screenshots` | ✅ All platforms |
| `sharp` | ✅ All platforms |

## 5. Development Platform

- **Build environment:** Node.js 20+
- **Development mode:** `NODE_ENV=development` enables debug logging
- **Hot reload:** `nodemon` watches `cli/index.js`
- **UI dev:** Vite dev server at `localhost:5173`, proxied to server at `localhost:2208`
- **UI build:** `vite build` → generates `ui/assets/index-*.js/css`
