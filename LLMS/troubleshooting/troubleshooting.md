# Troubleshooting Guide

> **Purpose:** Common issues, error messages, and their solutions.
> **Scope:** All 9Remote error scenarios and debugging steps.
> **Source:** `README.md`, `package/dist/cli.cjs`, `package/dist/server.cjs`

## 1. Overview

This guide covers the most common issues encountered when using 9Remote, organized by symptom. Use `NODE_ENV=development` or `AGENT_DEBUG=1` to enable verbose logging for deeper debugging.

## 2. Connection Issues

### 2.1 "Port 2208 already in use"

**Symptom:** Server fails to start, error message says port 2208 is in use.

**Cause:** Another 9Remote instance (or another service) is already bound to port 2208.

**Solution:**
```bash
# Kill the existing instance
pkill -f 9remote

# Or use a different port
PORT=3308 9remote
```

### 2.2 "Can't connect from phone"

**Symptom:** Phone shows "Connecting..." or "Server not found" but the host is running.

**Solutions (try in order):**

1. **Check internet on both devices:** Both host and phone must have internet access.
2. **Check tunnel status in TUI:** Ensure status shows `READY` (green). If `STOPPED`, restart the tunnel from the menu.
3. **Force tunnel mode:** If on the same WiFi, the LocalFirst adapter might have issues:
   - TUI: `Settings → Connection → Tunnel only`
   - Or: `Settings → Connection → LAN preferred` → toggle off
4. **Check tunnel URL:** Verify the tunnel URL is displayed and accessible.
5. **Reconnect:** Close and reopen the 9Remote mobile app, then scan QR again.

### 2.3 "QR code expired"

**Symptom:** Phone shows "Invalid or expired key" when scanning the QR code.

**Cause:** One-time keys expire after 30 minutes.

**Solution:**
```
TUI menu → Keys → Regenerate
```

Or restart 9Remote — a new one-time key is generated on each startup.

## 3. Cloudflare Tunnel Issues

### 3.1 "Cloudflare tunnel failed to start"

**Symptom:** TUI shows error about tunnel, or status stays at `TUNNELING` / `STOPPED`.

**Solutions:**

1. **Check internet connection:** Tunnel requires outbound HTTPS (port 443).
2. **Manual cloudflared install:** If auto-download fails:
   ```bash
   # Install cloudflared manually
   # macOS: brew install cloudflare/cloudflare/cloudflared
   # Linux: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/install/
   ```
3. **Check for corporate firewall:** Some corporate firewalls block cloudflared. Try:
   - VPN off (if VPN blocks outbound connections)
   - Different network (mobile hotspot)
4. **Debug logs:** Run with debug enabled:
   ```bash
   NODE_ENV=development 9remote
   ```

### 3.2 "Tunnel unreachable" / "Tunnel health check timed out"

**Symptom:** Tunnel URL is detected but health check fails.

**Cause:** Cloudflare edge has no active connections, or the tunnel process died.

**Solution:** The server will automatically retry. If persistent:
```bash
# Check if cloudflared is running
ps aux | grep cloudflared
cat ~/.9remote/pids/cloudflared.pid

# Kill and restart
kill -9 $(cat ~/.9remote/pids/cloudflared.pid)
9remote              # or restart the TUI
```

## 4. macOS Permission Issues

### 4.1 "Screen Recording / Accessibility permission denied"

**Symptom:** Remote Desktop toggle shows error, or desktop streams are black.

**Cause:** macOS requires explicit permission for screen recording and accessibility.

**Solution:**
1. Grant permissions:
   - `System Settings → Privacy & Security → Screen Recording` → Enable Terminal (or app)
   - `System Settings → Privacy & Security → Accessibility` → Enable Terminal (or app)
2. Restart 9Remote after granting.
3. If already granted, try removing and re-adding Terminal in the permissions list.

**From TUI:**
```
Menu → Desktop → Toggle ON
```
This will prompt for permissions if not granted.

**From API:**
```bash
# Trigger permission dialog
curl -X POST http://localhost:2208/api/permissions \
  -H "Content-Type: application/json" \
  -d '{"type":"screenRecording"}'
```

## 5. Update Issues

### 5.1 Update fails / "Verify failed"

**Symptom:** Update process fails, possibly with a version mismatch error.

**Cause:** The new version's SHA256 didn't match, or the installed binary is broken.

**Solution:**
1. **Check logs:**
   ```bash
   cat ~/.9remote/logs/agent.log
   ```
2. **Manual update:**
   ```bash
   npm install -g 9remote@latest
   ```
3. **Rollback:** If update broke the installation:
   ```bash
   npm install -g 9remote@<previous-version>
   ```

### 5.2 "Background server failed to start"

**Symptom:** After update, the background process can't start.

**Solution:**
1. Check if the old process is still running and kill it:
   ```bash
   pkill -f 9remote
   pkill -f cloudflared
   ```
2. Restart:
   ```bash
   9remote ui
   ```

## 6. PTY Session Issues

### 6.1 "Session not found"

**Symptom:** Connecting to a terminal session fails with "Session not found".

**Cause:** The session was created by a previous server instance and has been cleaned up, or the daemon restarted.

**Solution:** Create a new terminal session from the web UI.

### 6.2 "ptyDaemon not responding"

**Symptom:** Terminal doesn't display output, or sessions immediately close.

**Solution:**
1. Check if the daemon is running:
   ```bash
   cat ~/.9remote/pids/ptyDaemon.pid
   ps aux | grep ptyDaemon
   ```
2. Restart the daemon by restarting 9Remote:
   ```bash
   pkill -f 9remote
   pkill -f ptyDaemon
   9remote ui
   ```

### 6.3 Terminal shows garbled text

**Symptom:** Terminal output contains strange characters or escape sequences.

**Cause:** The terminal emulator settings don't match (TERM, COLORTERM).

**Solution:** The PTY daemon sets `TERM=xterm-256color` and `COLORTERM=truecolor` automatically. If issues persist:
1. Check shell config (`.zshrc`, `.bashrc`) for conflicting settings.
2. Restart the session.

## 7. Desktop Streaming Issues

### 7.1 Black screen / no video

**Symptom:** Desktop stream connects but shows a black screen.

**Causes and solutions:**

1. **Permissions (macOS):** Ensure Screen Recording and Accessibility are granted (see §4.1).
2. **robotjs not installed:** Check if `@hurdlegroup/robotjs` is available.
3. **GPU acceleration failure:** The system falls back to CPU capture if GPU init fails.
4. **Display sleep:** If the host display is asleep, no frames are captured. Move the mouse or disable display sleep.

### 7.2 "Remote desktop laggy"

**Symptom:** High latency or low frame rate in desktop streaming.

**Solutions:**
1. **Use LAN mode:** If on same WiFi, enable LocalFirst (should auto-detect).
2. **Reduce resolution:** Desktop capture quality can be adjusted in settings.
3. **Close other bandwidth-intensive apps.**
4. **Check CPU usage:** Desktop capture is CPU-intensive.

## 8. File Upload Issues

### 8.1 "File upload error"

**Symptom:** Uploading files via drag-and-drop fails.

**Solution:**
1. **Filename sanitization:** Special characters in filenames are replaced with `_`. Check the log for the sanitized name.
2. **File size:** Very large files may timeout. Try smaller files.
3. **Buffer path:** Files are written to `~/.9remote/buffers/` — ensure this directory is writable.

## 9. Log Analysis

### 9.1 Enabling Debug Mode

```bash
# Method 1: NODE_ENV
NODE_ENV=development 9remote ui

# Method 2: AGENT_DEBUG
AGENT_DEBUG=1 9remote

# Method 3: Command-line flag
9remote ui --debug
```

### 9.2 Key Log Messages

| Message | Meaning | Action |
|---------|---------|--------|
| `tunnel ready: https://...` | Tunnel URL detected | Wait for health check |
| `URL rotated: https://...` | Cloudflare changed the URL | None needed (auto-handled) |
| `edge probe failed (N/3)` | Tunnel may be losing connections | Wait for auto-restart |
| `Network change detected` | IP changed | Tunnel will restart |
| `Sleep/wake detected` | System slept/resumed | Tunnel will restart |
| `Session create failed` | Worker API error | Check internet, restart |
| `Integrity check failed` | Download corruption | Restart update |
| `Failed to start daemon` | PTY daemon crashed | Restart 9Remote |

### 9.3 Reading Logs

```bash
# View last 60 lines
cat ~/.9remote/logs/agent.log

# Real-time tail
tail -f ~/.9remote/logs/agent.log

# Filter by level
grep "\[error\]" ~/.9remote/logs/agent.log
grep "\[warn\]" ~/.9remote/logs/agent.log

# View via API
curl "http://localhost:2208/api/logs?lines=100"
```

## 10. Common Error Codes

| Code | Description |
|------|-------------|
| `403 Forbidden` | API request from non-local/non-CF source |
| `400 Bad Request` | Malformed JSON body |
| `401 Unauthorized` | Invalid API key (worker session creation) |
| `500 Internal Server Error` | Server-side exception |
| `SIGKILL` | Process killed by OS (OOM or signal) |

## 11. Getting Help

1. **Check logs:** `cat ~/.9remote/logs/agent.log`
2. **Enable debug:** `NODE_ENV=development 9remote`
3. **GitHub Issues:** [github.com/decolua/9remote/issues](https://github.com/decolua/9remote/issues)
4. **Community:** [facebook.com/groups/9teamvn](https://www.facebook.com/groups/9teamvn)
5. **Documentation:** [docs.9remote.cc](https://docs.9remote.cc)
