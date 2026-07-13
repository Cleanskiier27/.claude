# NetworkBuster GLC — CLI Reference

> Generated from [DOCS-m2m PR #1](https://github.com/Cleanskiier27/DOCS-m2m/pull/1):  
> *Add GLC cmake mixed-mode build, profile recon pipeline stage, and web root certificate docs*

---

## Table of Contents

1. [profile_recon.py](#profile_reconpy)
   - [CLI Mode](#cli-mode)
   - [Server Mode](#server-mode)
   - [Environment Variables](#environment-variables)
   - [API Endpoints](#api-endpoints)
2. [glc-cli](#glc-cli)
   - [Build](#build)
   - [Install](#install)
3. [build-all.ps1](#build-allps1)
4. [Quick Reference](#quick-reference)

---

## profile_recon.py

**File:** `profile_recon.py`  
**Runtime:** Python 3.x + Flask  
**Purpose:** Reconnaissance and profiling engine for the NetworkBuster pipeline. Probes pipeline services via TCP, collects host/OS metadata, and profiles the local git repository.

### CLI Mode

Run a one-shot recon report printed to stdout:

```bash
python profile_recon.py
```

**Output sections:**

| Section | Description |
|---------|-------------|
| Host | Hostname, platform, user, Python/Node/CMake/Git versions |
| Repo | Remote URL, branch, last commit SHA, clean/dirty status |
| Pipeline Builds | Numbered build list parsed from `build-pipeline.js` |
| Service Probes | TCP open/closed + latency for each registered pipeline port |

**Sample output:**

```
════════════════════════════════════════════════════════════
  NetworkBuster — Profile Recon Report
════════════════════════════════════════════════════════════

  Host:       my-host (Windows-11-10.0.26100)
  User:       developer
  Python:     3.12.0
  Node:       v24.12.0
  CMake:      cmake version 3.28.1
  Git:        git version 2.43.0

  Repo:       https://github.com/Cleanskiier27/DOCS-m2m.git
  Branch:     main
  Last Commit:8743f7d40a36
  Clean:      ✅

  Pipeline Builds (5):
    [1] Web Server
    [2] API Server
    [3] Audio Server
    [4] Auth Server
    [5] Profile Recon

  Service Probes:
    Web Server           http://localhost:3000       ✅ UP (2.5ms)
    API Server           http://localhost:3001       ✅ UP (1.8ms)
    Audio Server         http://localhost:3002       ❌ DOWN
    Auth Server          http://localhost:3003       ✅ UP (3.1ms)
    Mission Control      http://localhost:5000       ❌ DOWN
    Network Map          http://localhost:6000       ❌ DOWN
    YouTube Trends       http://localhost:8001       ❌ DOWN
    Profile Recon        http://localhost:9100       ✅ UP (0.9ms)

════════════════════════════════════════════════════════════
```

### Server Mode

Start the Flask HTTP server (registered as **Build 5** in the pipeline):

```bash
python profile_recon.py --serve
```

The server binds to `0.0.0.0` on port **9100** (overridable via `RECON_PORT`).

```bash
# Custom port
RECON_PORT=9200 python profile_recon.py --serve

# Windows PowerShell
$env:RECON_PORT = "9200"; python profile_recon.py --serve
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `RECON_PORT` | `9100` | TCP port for the HTTP server |

### API Endpoints

All endpoints are `GET` and return JSON.

| Endpoint | Description |
|----------|-------------|
| `GET /api/health` | Health check — `{"status":"ok","service":"profile-recon","port":9100}` |
| `GET /api/recon/services` | TCP probe all pipeline services, returns port open/closed + latency |
| `GET /api/recon/host` | Local host profile (OS, versions, user, cwd) |
| `GET /api/recon/repo` | Git repository profile (branch, remote, last commit, clean/dirty) |
| `GET /api/recon/builds` | Pipeline build entries parsed from `build-pipeline.js` |
| `GET /api/recon/report` | Full combined report (host + repo + builds + services) |

**Example:**

```bash
# Health check
curl http://localhost:9100/api/health

# Full recon report (JSON)
curl http://localhost:9100/api/recon/report

# Services only
curl http://localhost:9100/api/recon/services
```

---

## glc-cli

**File:** `tools/glc_cli.c` or `tools/glc_cli.cpp`  
**Runtime:** Native binary (C11 / C++17, built via CMake)  
**Purpose:** Optional native CLI executable for the GLC (Git Link Controller) library.

### Build

```bash
# Configure (Release build)
cmake -B build -DCMAKE_BUILD_TYPE=Release

# Build everything (library + CLI if tools/glc_cli.c[pp] exists)
cmake --build build

# Binary location
./build/bin/glc-cli
```

The CLI target is **optional** — it is only compiled when `tools/glc_cli.c` or `tools/glc_cli.cpp` is present. To verify the target was built:

```bash
cmake --build build --target glc-cli
```

**Compiler requirements:**

| Component | Standard |
|-----------|----------|
| C sources (`lib/*.c`) | C11 |
| C++ sources (`lib/*.cpp`, `tools/*.cpp`) | C++17 |

**CMake options:**

| Option | Default | Description |
|--------|---------|-------------|
| `CMAKE_BUILD_TYPE` | `Release` | Build configuration (`Release`, `Debug`, `RelWithDebInfo`) |
| `CMAKE_INSTALL_PREFIX` | `/usr/local` | Installation prefix |

### Install

```bash
# Install binary to <prefix>/bin and docs to <prefix>/share/networkbuster/docs
cmake --install build --prefix /usr/local
```

After install, `glc-cli` is available system-wide:

```bash
glc-cli --help
```

---

## build-all.ps1

**File:** `build-all.ps1`  
**Runtime:** PowerShell 5.1+ / PowerShell 7+  
**Purpose:** Orchestrates the full build sequence: CMake (GLC native), then Node.js/React builds.

```powershell
# Run the full build pipeline
.\build-all.ps1
```

**Build order:**

1. **CMake configure** — `cmake -B build -DCMAKE_BUILD_TYPE=Release`
2. **CMake build** — `cmake --build build`
3. **Node.js / React builds** — subsequent frontend build steps

---

## Quick Reference

```bash
# One-shot recon report (CLI)
python profile_recon.py

# Start recon server
python profile_recon.py --serve

# Query full report from running server
curl http://localhost:9100/api/recon/report

# Build GLC native components
cmake -B build -DCMAKE_BUILD_TYPE=Release && cmake --build build

# Full pipeline build (Windows)
.\build-all.ps1
```

---

*Source: [Cleanskiier27/DOCS-m2m PR #1](https://github.com/Cleanskiier27/DOCS-m2m/pull/1)*
