# LLMS Documentation — 9Remote Project

This directory contains LLM-oriented documentation for the **9Remote** project (v2.5.8), a terminal-in-your-pocket application that provides remote terminal access, desktop streaming, file explorer, and code editing capabilities via a single CLI.

## Directory Structure

```
LLMS/
├── api/                    # API documentation
│   ├── api-endpoints.md    # HTTP API endpoint reference
│   ├── socket-events.md    # Socket.IO real-time events
│   └── sse-streams.md      # Server-Sent Events reference
├── context/                # High-level context and architecture
│   ├── architecture.md     # System architecture overview
│   ├── tech-stack.md       # Technology stack details
│   └── data-flows.md       # Data flow diagrams
├── app-details/            # Application details and design
│   ├── app-overview.md     # Application overview and features
│   ├── components.md       # Component breakdown
│   ├── platform-support.md # Platform/OS support matrix
│   └── security-model.md   # Security model
├── app-code-flow/          # Application code flow documentation
│   ├── startup-flow.md     # Application startup sequence
│   ├── tunnel-lifecycle.md # Cloudflare tunnel lifecycle
│   ├── session-lifecycle.md# PTY session lifecycle
│   ├── ui-lifecycle.md     # Web UI lifecycle and state management
│   ├── upgrade-flow.md     # Update/upgrade flow
│   └── first-run.md        # First run and onboarding
├── config/                 # Configuration reference
│   ├── config-files.md     # Configuration files and paths
│   ├── environment-vars.md # Environment variables
│   └── constants.md        # Key constants and defaults
├── deployment/             # Deployment documentation
│   ├── deployment.md       # Deployment architecture
│   ├── cloudflare-worker.md# Cloudflare Worker API
│   └── cloudflare-tunnel.md# Cloudflare Tunnel integration
├── cli/                    # CLI reference
│   ├── cli-commands.md     # CLI commands and flags
│   └── tui-menu.md         # TUI menu structure
├── troubleshooting/        # Troubleshooting guides
│   ├── troubleshooting.md  # Common issues and solutions
│   └── error-codes.md      # Error codes and messages
├── mode-assignments.md     # Mode-to-file assignment manifest
└── _template.md            # Documentation template
```

## Quick Links

- **[App Overview](app-details/app-overview.md)** — What 9Remote does and its key features
- **[Architecture](context/architecture.md)** — How the system is structured
- **[API Endpoints](api/api-endpoints.md)** — Complete HTTP API reference
- **[Startup Flow](app-code-flow/startup-flow.md)** — How the app boots
- [9Remote Official Docs](https://docs.9remote.cc) | [GitHub](https://github.com/decolua/9remote)
