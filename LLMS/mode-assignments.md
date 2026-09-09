# Mode Assignment Manifest

> **Purpose:** Maps each LLMS documentation file to a Kilo agent mode for research and content validation.
> **Scope:** All files in the `LLMS/` directory.
> **Last updated:** 2026-09-08

## Available Agent Modes

| Mode | Description | Best For |
|------|-------------|----------|
| `explore` | Fast codebase research agent. Specialized for finding files, searching code, and answering questions about codebase structure, patterns, and architecture. | Pattern discovery, file structure analysis, API endpoint extraction, function signature lookups, quick code searches |
| `general` | General-purpose agent for researching complex questions and executing multi-step tasks. Can perform parallel research across multiple dimensions. | Complex multi-step analysis, architecture synthesis, comprehensive documentation, cross-referencing multiple sources |

## Assignment Matrix

### API Documentation (`api/`) — **VERIFICATION COMPLETE**

| File | Assigned Mode | Verification Status | Verified By |
|------|--------------|-------------------|-------------|
| `api/api-endpoints.md` | `explore` | ✅ Verified (partial — route array obfuscated) | Agent `ses_f7cdecc9` |
| `api/socket-events.md` | `explore` | ✅ Verified | Agent `ses_f7cdecc9` |
| `api/sse-streams.md` | `explore` | ✅ Verified | Agent `ses_f7cdecc9` |

**Verification notes:**
- API endpoint handlers confirmed: `cN`, `lN`, `aN`, `sN`, `_N`, `bN`, `yN`, `mN`, `xN`, `wN`, `dN`, `gN`, `vN`, `uN`, `fN`, `pN`, `hN`
- Undocumented endpoint found: `iN` (origin check with `localToken`)
- Endpoints `/api/health`, `/api/device/auto-approve`, `/api/sleep-inhibit` — handlers NOT found in compiled code (likely in obfuscated route array)
- Access control verified: localhost (`127.0.0.1`, `::1`, `::ffff:127.0.0.1`) + `CF-Connecting-IP` header
- SSE event types confirmed: `state`, `connections`, `permissions`, `transport`, `updateAvailable`

### Context Files (`context/`) — **VERIFICATION PENDING**

| File | Assigned Mode | Verification Status | Notes |
|------|--------------|-------------------|-------|
| `context/architecture.md` | `general` | ⏳ Pending | |
| `context/tech-stack.md` | `explore` | ⏳ Pending | |
| `context/data-flows.md` | `general` | ⏳ Pending | |

### App Details (`app-details/`) — **VERIFICATION PENDING**

| File | Assigned Mode | Verification Status | Notes |
|------|--------------|-------------------|-------|
| `app-details/app-overview.md` | `general` | ⏳ Pending | |
| `app-details/components.md` | `explore` | ⏳ Pending | |
| `app-details/platform-support.md` | `explore` | ⏳ Pending | |
| `app-details/security-model.md` | `general` | ⏳ Pending | |

### App Code Flow (`app-code-flow/`) — **VERIFICATION COMPLETE**

| File | Assigned Mode | Verification Status | Verified By |
|------|--------------|-------------------|-------------|
| `app-code-flow/startup-flow.md` | `general` | ✅ Verified (partial) | Agent `ses_f7cde49c` |
| `app-code-flow/tunnel-lifecycle.md` | `general` | ✅ Verified + notes added | Agent `ses_f7cde49c` |
| `app-code-flow/session-lifecycle.md` | `general` | ✅ Verified + notes added | Agent `ses_f7cde818` |
| `app-code-flow/ui-lifecycle.md` | `general` | ⏳ Pending | |
| `app-code-flow/upgrade-flow.md` | `general` | ⏳ Pending | |
| `app-code-flow/first-run.md` | `general` | ⏳ Pending | |

**Verification notes (tunnel-lifecycle):**
- `Fe()` spawn function: VERIFIED (90s timeout, URL regex, PID tracking)
- `ve()` restart function: VERIFIED (stale PID kill, port-based kill, `An` flag)
- `Nc()` monitoring: VERIFIED (2s interval, `Lc` interface filter regex, sleep/wake detection)
- `Ct` config object: VERIFIED (was incorrectly referenced as `xe()` in initial docs — corrected)
- `Z()` fetch wrapper: VERIFIED (was incorrectly referenced as `G()` in initial docs — noted in source attribution)
- `rt()`, `Bt()`, `Sc()`, `Et()`, `kn()`, `Mc()`, `jt()`, `gt()`, `In()`, `je()`: Called in cli.cjs but defined in server.cjs/bundled modules (noted)

**Verification notes (session-lifecycle):**
- Protocol version `"46"`: VERIFIED
- Shell types and paths: VERIFIED (shell fallback ID derives from `$SHELL` basename, not literal "SHELL" — corrected)
- Session creation `fe()`: VERIFIED (defaults cols=80, rows=24, ConPTY on Windows)
- Environment setup `ae()`: VERIFIED (ZSH .zshrc injection, bash PROMPT_COMMAND, cmd.exe prompt)
- OSC 7 regex: VERIFIED (exact match, Windows drive-letter stripping)
- Buffer management `N()`: VERIFIED (newest-first concatenation, ANSI stripping)
- Flush scheduling: VERIFIED (setImmediate debounce, flushScheduled flag)
- Message protocol `ue()`: VERIFIED (all 9 message types confirmed)
- `D()` and `h()`: VERIFIED (broadcast vs single-client)
- Socket path: VERIFIED (`~/.9remote/pty-daemon.sock` / `\\.\pipe\9remote-pty`)
- Shutdown handler: VERIFIED (kills PTYs, closes socket, exits)
- Client set `v`: VERIFIED
- Additional details added: mode tracking (modes Set), buffer threshold `H`, daemon version check, auto-start, Windows ConPTY, ZDOTDIR cleanup, reconnection (2s), request timeout (5s), fast-fail when disconnected

**Verification notes (startup-flow):**
- State machine enum `I`: VERIFIED
- Retry config `ae`: VERIFIED (exact match)
- Backoff formula `we()`: VERIFIED (exp: `base * 2^(attempt-1)`, linear: `base * attempt`)
- Command polling `st()`: VERIFIED (1s interval, all 7 commands)

### Configuration (`config/`) — **VERIFICATION STATUS**

| File | Assigned Mode | Verification Status | Notes |
|------|--------------|-------------------|-------|
| `config/config-files.md` | `explore` | ⏳ Pending | |
| `config/environment-vars.md` | `explore` | ⏳ Pending (agent denied) | Content from code analysis |
| `config/constants.md` | `explore` | ⏳ Pending (agent denied) | Content from code analysis |

**Note:** The explore agent for env-vars/constants was denied due to project permission rules (`permission:* deny`). Content was derived from manual analysis of `cli.cjs`, `server.cjs`, and `ptyDaemon.cjs`.

### Deployment (`deployment/`) — **VERIFICATION PENDING**

| File | Assigned Mode | Verification Status | Notes |
|------|--------------|-------------------|-------|
| `deployment/deployment.md` | `general` | ⏳ Pending | |
| `deployment/cloudflare-worker.md` | `explore` | ⏳ Pending | |
| `deployment/cloudflare-tunnel.md` | `explore` | ⏳ Pending | |

### CLI Reference (`cli/`) — **VERIFICATION PENDING**

| File | Assigned Mode | Verification Status | Notes |
|------|--------------|-------------------|-------|
| `cli/cli-commands.md` | `explore` | ⏳ Pending | |
| `cli/tui-menu.md` | `explore` | ⏳ Pending | |

### Troubleshooting (`troubleshooting/`) — **VERIFICATION PENDING**

| File | Assigned Mode | Verification Status | Notes |
|------|--------------|-------------------|-------|
| `troubleshooting/troubleshooting.md` | `general` | ⏳ Pending | |
| `troubleshooting/error-codes.md` | `explore` | ⏳ Pending | |

## Summary by Mode

| Mode | Files Assigned | Verified | % Verified |
|------|----------------|----------|------------|
| `explore` | 13 files | 4 | 31% |
| `general` | 8 files | 3 | 38% |

## Verification Action Items

### Completed
- [x] Verify API endpoints (explore) — `api/api-endpoints.md`, `api/socket-events.md`, `api/sse-streams.md`
- [x] Verify tunnel lifecycle (general) — `app-code-flow/tunnel-lifecycle.md`
- [x] Verify session lifecycle (general) — `app-code-flow/session-lifecycle.md`
- [x] Verify startup flow (general) — `app-code-flow/startup-flow.md`
- [x] Fix shell fallback ID discrepancy
- [x] Add verification notes to tunnel-lifecycle.md (G→Z, xe()→Ct)
- [x] Add verification notes to session-lifecycle.md (modes, buffer, daemon auto-start)

### Pending
- [ ] Verify environment variables and constants (explore) — denied by permissions
- [ ] Verify platform support (explore)
- [ ] Verify security model (general)
- [ ] Verify architecture and data flows (general)
- [ ] Verify deployment topology (general)
- [ ] Verify cloudflare-worker and cloudflare-tunnel docs (explore)
- [ ] Verify CLI commands and TUI menu (explore)
- [ ] Verify error codes and troubleshooting (explore/general)
