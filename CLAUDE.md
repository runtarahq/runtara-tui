# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Runtara TUI is a terminal-based monitoring interface for the Runtara durable execution platform. It connects to `runtara-environment` via QUIC and displays real-time information about instances, images, metrics, and health status.

## Build Commands

```bash
# Build
cargo build

# Run (requires runtara-environment running on port 8002)
cargo run

# Run with custom server
cargo run -- --server 192.168.1.100:8002

# Run with tenant filter (required for metrics)
cargo run -- --tenant my-tenant-id

# Run with custom refresh interval (seconds)
cargo run -- --refresh 10
```

## Architecture

```
src/
├── main.rs   # Entry point, terminal setup, event loop, keyboard handling
├── app.rs    # Application state (App struct), SDK interactions, data fetching
└── ui.rs     # Ratatui rendering (tabs, tables, modals, popups)
```

### Key Components

- **App** (`src/app.rs`): Central state container holding connection config, fetched data (instances, images, metrics, health), view mode, and selection state. Creates `ManagementSdk` on each refresh to fetch data.

- **ViewMode** enum: Controls navigation between `List` (main tabs), `InstanceDetail`, `CheckpointsList`, and `CheckpointDetail` views.

- **Tab** enum: Four main tabs - Instances, Images, Metrics, Health.

### Data Flow

1. `App::refresh()` creates a new `ManagementSdk`, connects via QUIC, fetches all data
2. Auto-refresh triggers based on `refresh_interval` (default 5s)
3. UI renders current `App` state via `ui::draw()`

## Environment Variables

- `RUNTARA_ENV_ADDR` - Server address (default: `127.0.0.1:8002`)
- `RUNTARA_SKIP_CERT_VERIFICATION` - Skip TLS verification (default: `true`)

## Keyboard Shortcuts

| Key | List View | Detail Views |
|-----|-----------|--------------|
| `q`/`Esc` | Quit | Back |
| `Tab` | Next tab | - |
| `1-4` | Jump to tab | - |
| `j`/`k` | Navigate list | Scroll |
| `Enter` | Open detail (Instances) | View checkpoint |
| `f` | Cycle status filter | - |
| `g` | Toggle metrics granularity | - |
| `r` | Refresh | - |
| `c` | - | View checkpoints (Instance detail) |

## Dependencies

- `runtara-management-sdk` - QUIC client for runtara-environment
- `ratatui` + `crossterm` - Terminal UI
- `tokio` - Async runtime
- `clap` - CLI argument parsing
