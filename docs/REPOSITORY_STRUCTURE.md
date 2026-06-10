# Beszel Repository Structure

## Complete Project Layout

```
beszel/
├── Root Configuration & Build Files
│   ├── beszel.go                 # Core application constants and version info
│   ├── go.mod                    # Go module definition and dependencies
│   ├── Makefile                  # Build automation and task runner
│   ├── i18n.yml                  # Internationalization configuration
│   ├── readme.md                 # Project overview
│   ├── SECURITY.md               # Security policy and vulnerability disclosure
│   ├── LICENSE                   # MIT License
│   │
│   └── Dockerfiles (in project root)
│       ├── dockerfile_agent              # Agent image
│       ├── dockerfile_agent_alpine       # Agent Alpine Linux variant
│       ├── dockerfile_agent_intel        # Agent with Intel GPU support
│       └── dockerfile_agent_nvidia       # Agent with Nvidia GPU support
│
├── agent/                        # AGENT MONITORING COMPONENT
│   │
│   ├── Core Agent System
│   │   ├── agent.go              # Main Agent struct and lifecycle
│   │   ├── agent_test.go         # Agent unit tests
│   │   ├── agent_cache.go        # Metrics caching system
│   │   ├── agent_cache_test.go   # Cache tests
│   │   ├── agent_test_helpers.go # Test utilities
│   │   │
│   │   ├── server.go             # SSH server implementation
│   │   ├── handlers.go           # SSH session handlers
│   │   ├── handlers_test.go      # Handler tests
│   │   ├── response.go           # Response formatting (CBOR/JSON)
│   │   │
│   │   ├── client.go             # SSH client helpers
│   │   ├── client_test.go        # Client tests
│   │   ├── connection_manager.go # Connection lifecycle management
│   │   ├── connection_manager_test.go
│   │   │
│   │   └── update.go             # Agent update checking
│   │
│   ├── Metrics Collection (System)
│   │   ├── system.go             # System initialization and details
│   │   ├── system_test.go        # System collection tests
│   │   │
│   │   ├── cpu.go                # CPU metrics collection
│   │   ├── memory.go             # Memory metrics (RAM, swap, ZFS ARC)
│   │   │
│   │   ├── disk.go               # Disk usage and I/O metrics
│   │   ├── disk_test.go          # Disk metrics tests
│   │   │
│   │   ├── network.go            # Network interface and bandwidth metrics
│   │   ├── network_test.go       # Network metrics tests
│   │   │
│   │   ├── sensors.go            # Temperature sensor collection
│   │   ├── sensors_default.go    # Cross-platform sensor defaults
│   │   ├── sensors_windows.go    # Windows-specific sensors
│   │   ├── sensors_test.go       # Sensor tests
│   │   │
│   │   └── load.go               # Load average (included in system.go)
│   │
│   ├── GPU Monitoring
│   │   ├── gpu.go                # GPU manager and collection dispatcher
│   │   ├── gpu_test.go           # GPU tests
│   │   │
│   │   ├── gpu.go                # Generic GPU interface
│   │   ├── gpu_nvml.go           # Nvidia NVML API (Linux, Windows)
│   │   ├── gpu_nvml_linux.go     # Linux-specific NVML implementation
│   │   ├── gpu_nvml_unsupported.go # Stub for unsupported platforms
│   │   ├── gpu_nvml_windows.go   # Windows NVML implementation
│   │   ├── gpu_nvtop.go          # nvtop command parser
│   │   │
│   │   ├── gpu_amd_linux.go      # AMD GPU on Linux (rocm-smi)
│   │   ├── gpu_amd_linux_test.go # AMD GPU tests
│   │   ├── gpu_amd_unsupported.go# Stub for non-Linux AMD
│   │   │
│   │   ├── gpu_darwin.go         # macOS GPU support (PowerMetrics)
│   │   ├── gpu_darwin_test.go    # macOS GPU tests
│   │   ├── gpu_darwin_unsupported.go
│   │   │
│   │   └── gpu_intel.go          # Intel GPU support (via NVML/oneAPI)
│   │
│   ├── Storage Health Monitoring
│   │   ├── smart.go              # S.M.A.R.T. data collection dispatcher
│   │   ├── smart_test.go         # S.M.A.R.T. tests
│   │   ├── smart_windows.go      # Windows smartctl.exe implementation
│   │   ├── smart_nonwindows.go   # Unix smartctl implementation
│   │   │
│   │   ├── emmc_common.go        # eMMC wear/EOL tracking (cross-platform)
│   │   ├── emmc_common_test.go   # eMMC tests
│   │   ├── emmc_linux.go         # Linux eMMC implementation
│   │   ├── emmc_linux_test.go    # Linux eMMC tests
│   │   ├── emmc_stub.go          # Non-Linux eMMC stub
│   │   │
│   │   ├── mdraid_linux.go       # Linux mdraid array health
│   │   ├── mdraid_linux_test.go  # mdraid tests
│   │   └── mdraid_stub.go        # Non-Linux mdraid stub
│   │
│   ├── Container Support
│   │   ├── docker.go             # Docker/Podman API manager
│   │   ├── docker_test.go        # Docker tests
│   │   │
│   │   └── [Container data models in internal/entities/container/]
│   │
│   ├── Other System Features
│   │   ├── fingerprint.go        # Agent fingerprint (identity verification)
│   │   ├── fingerprint_test.go   # Fingerprint tests
│   │   │
│   │   ├── battery/              # Battery monitoring
│   │   │   ├── battery.go        # Battery interface
│   │   │   ├── battery_linux.go  # Linux implementation
│   │   │   ├── battery_linux_test.go
│   │   │   ├── battery_darwin.go # macOS implementation
│   │   │   ├── battery_windows.go# Windows implementation
│   │   │   └── battery_stub.go   # Unsupported platforms
│   │   │
│   │   ├── systemd_linux.go      # Systemd service monitoring
│   │   ├── systemd_test.go       # Systemd tests
│   │   ├── systemd_nonlinux.go   # Non-Linux systemd stub
│   │   ├── systemd_nonlinux_test.go
│   │   │
│   │   └── zfs/                  # ZFS ARC memory tracking
│   │       ├── zfs_linux.go      # Linux ZFS ARC
│   │       ├── zfs_freebsd.go    # FreeBSD ZFS
│   │       └── zfs_unsupported.go# Unsupported platforms
│   │
│   ├── Data Management
│   │   ├── data_dir.go           # Agent data directory management
│   │   ├── data_dir_test.go      # Data directory tests
│   │   │
│   │   └── deltatracker/         # Delta tracking for rate calculations
│   │       ├── deltatracker.go   # Generic delta tracker
│   │       └── deltatracker_test.go
│   │
│   ├── Health Monitoring
│   │   └── health/               # System health status
│   │       ├── health.go         # Health check implementation
│   │       └── health_test.go
│   │
│   ├── Utilities
│   │   ├── utils.go              # Common utilities
│   │   ├── utils_test.go         # Utility tests
│   │   │
│   │   └── tools/                # Utility tools
│   │       └── fetchsmartctl/    # Smart tool downloader
│   │
│   ├── Data Models
│   │   └── [Shared models in internal/entities/]
│   │
│   ├── LibraryHardwareMonitor (.NET/C#)
│   │   └── lhm/                  # .NET library for hardware monitoring
│   │       ├── beszel_lhm.cs    # Main library file
│   │       └── beszel_lhm.csproj # .NET project file
│   │
│   └── Test Data & Fixtures
│       └── test-data/
│           ├── system_info.json   # Mock system information
│           ├── container.json     # Mock container data
│           ├── container2.json    # Additional container test data
│           ├── nvtop.json        # Mock GPU (nvtop) output
│           ├── amdgpu.ids        # AMD GPU device information
│           └── smart/            # S.M.A.R.T. test fixtures
│
├── internal/                     # INTERNAL APPLICATION CODE
│   │
│   ├── Hub Command Entry Point
│   │   └── cmd/hub/
│   │       └── main.go           # Hub server entry point
│   │
│   ├── Agent Command Entry Point
│   │   └── cmd/agent/
│   │       └── main.go           # Agent entry point
│   │
│   ├── Core Hub Application
│   │   ├── hub/
│   │   │   ├── hub.go            # Hub initialization and lifecycle
│   │   │   ├── hub_test.go       # Hub tests
│   │   │   ├── hub_test_helpers.go
│   │   │   │
│   │   │   ├── server.go         # HTTP server setup
│   │   │   ├── server_production.go  # Production server config
│   │   │   ├── server_development.go # Development server config
│   │   │   │
│   │   │   ├── api.go            # API route registration
│   │   │   ├── api_test.go       # API tests
│   │   │   │
│   │   │   ├── collections.go    # PocketBase collections setup
│   │   │   ├── collections_test.go
│   │   │   │
│   │   │   ├── agent_connect.go  # Agent connection logic
│   │   │   ├── agent_connect_test.go
│   │   │   │
│   │   │   ├── update.go         # Hub update checking
│   │   │   │
│   │   │   ├── config/           # Configuration management
│   │   │   │   ├── config.go     # Hub configuration
│   │   │   │   └── config_test.go
│   │   │   │
│   │   │   ├── systems/          # System management
│   │   │   │   ├── system_manager.go        # System lifecycle management
│   │   │   │   ├── system.go                # Single system connection/data
│   │   │   │   ├── system_test.go
│   │   │   │   ├── system_realtime.go       # Real-time data streaming
│   │   │   │   ├── systems_production.go    # Production-specific logic
│   │   │   │   ├── system_smart.go         # S.M.A.R.T. data handling
│   │   │   │   ├── system_smart_test.go
│   │   │   │   ├── system_systemd_test.go  # Systemd integration
│   │   │   │   ├── systems_test.go
│   │   │   │   └── systems_test_helpers.go
│   │   │   │
│   │   │   ├── ws/              # WebSocket support
│   │   │   │   ├── [WebSocket handlers and utilities]
│   │   │   │
│   │   │   ├── transport/       # Data transport & communication
│   │   │   │   ├── [Protocol handlers and utilities]
│   │   │   │
│   │   │   ├── expirymap/       # Expiring cache map utility
│   │   │   │   ├── [Expiry map implementation]
│   │   │   │
│   │   │   ├── heartbeat/       # System heartbeat monitoring
│   │   │   │   ├── [Heartbeat implementation]
│   │   │   │
│   │   │   └── utils/           # Hub-specific utilities
│   │   │       ├── [Utility functions]
│   │   │
│   │   ├── alerts/              # Alert System
│   │   │   ├── alerts.go        # Main alert manager
│   │   │   ├── alerts_test.go
│   │   │   ├── alerts_api.go    # Alert API endpoints
│   │   │   ├── alerts_api_test.go
│   │   │   │
│   │   │   ├── alerts_system.go # System-level alerts
│   │   │   ├── alerts_system_test.go
│   │   │   ├── alerts_status.go # Status change alerts
│   │   │   ├── alerts_status_test.go
│   │   │   │
│   │   │   ├── alerts_cache.go  # Alert deduplication cache
│   │   │   ├── alerts_cache_test.go
│   │   │   │
│   │   │   ├── alerts_disk_test.go  # Disk-specific alerts
│   │   │   ├── alerts_battery_test.go# Battery alerts
│   │   │   ├── alerts_quiet_hours_test.go
│   │   │   ├── alerts_smart.go  # S.M.A.R.T. alerts
│   │   │   ├── alerts_smart_test.go
│   │   │   │
│   │   │   ├── alerts_history.go# Alert history tracking
│   │   │   ├── alerts_test_helpers.go
│   │   │
│   │   ├── records/             # Historical Data Storage
│   │   │   ├── records.go       # Record storage and retrieval
│   │   │   ├── records_test.go
│   │   │   ├── records_deletion.go   # Record cleanup
│   │   │   ├── records_deletion_test.go
│   │   │   ├── records_averaging_test.go
│   │   │   └── records_test_helpers.go
│   │   │
│   │   └── users/              # User Management
│   │       ├── [User management implementation]
│   │
│   ├── Data Models & Entities
│   │   ├── entities/
│   │   │   ├── container/       # Container data models
│   │   │   │   ├── container.go # Container struct definitions
│   │   │   │
│   │   │   ├── smart/           # S.M.A.R.T. data models
│   │   │   │   ├── smart.go     # S.M.A.R.T. definitions
│   │   │   │
│   │   │   ├── system/          # System metrics data models
│   │   │   │   ├── system.go    # System and combined data structs
│   │   │   │
│   │   │   └── systemd/         # Systemd data models
│   │   │       ├── systemd.go   # Service data definitions
│   │   │
│   │   ├── common/              # Shared protocol definitions
│   │   │   ├── common-ssh.go    # SSH security constants
│   │   │   └── common-ws.go     # WebSocket/Hub-Agent protocol
│   │   │
│   │   ├── records/             # [See records above]
│   │   │
│   │   └── users/               # [See users above]
│   │
│   ├── Database
│   │   ├── migrations/          # Database schema migrations
│   │   │   ├── initial-settings.go    # Initial database setup
│   │   │   └── 0_collections_snapshot_0_19_0_dev_1.go
│   │   │
│   │   └── ghupdate/            # GitHub update checking
│   │       ├── [Update check implementation]
│   │
│   ├── Web UI
│   │   ├── site/                # Frontend application (Vue.js)
│   │   │   ├── package.json     # JavaScript dependencies
│   │   │   ├── package-lock.json
│   │   │   ├── src/
│   │   │   │   ├── App.vue      # Main application component
│   │   │   │   ├── main.ts      # Entry point
│   │   │   │   ├── components/  # Vue components
│   │   │   │   ├── views/       # Page components
│   │   │   │   ├── services/    # API services
│   │   │   │   ├── stores/      # State management (Pinia)
│   │   │   │   ├── styles/      # CSS/SCSS
│   │   │   │   ├── utils/       # Utility functions
│   │   │   │   └── assets/      # Static assets
│   │   │   ├── public/          # Public static files
│   │   │   ├── dist/            # Built frontend (generated)
│   │   │   ├── build/           # Build configuration
│   │   │   ├── tailwind.config.js # Tailwind CSS config
│   │   │   ├── vite.config.ts   # Vite build config
│   │   │   └── tsconfig.json    # TypeScript config
│   │   │
│   │   └── [Site compiled into binary at build time]
│   │
│   └── Testing Utilities
│       └── tests/
│           ├── [Common test utilities and fixtures]
│
├── supplemental/                # SUPPLEMENTAL & DEPLOYMENT FILES
│   ├── CHANGELOG.md             # Release notes and version history
│   │
│   ├── debian/                  # Debian/Ubuntu packages
│   │   ├── package.json         # DEB package metadata
│   │   ├── scripts/
│   │   │   ├── postinst         # Post-installation script
│   │   │   ├── preinst          # Pre-installation script
│   │   │   └── postrm           # Post-removal script
│   │   └── [Debian package files]
│   │
│   ├── docker/                  # Docker Compose & Deployment
│   │   ├── docker-compose.yml   # Example Docker Compose
│   │   ├── docker-compose.prod.yml
│   │   ├── .env.example         # Environment variables template
│   │   └── [Docker-related configuration]
│   │
│   ├── guides/                  # Documentation & Guides
│   │   ├── [Getting started, setup, usage guides]
│   │
│   ├── kubernetes/              # Kubernetes Deployment
│   │   ├── deployment.yaml      # K8s deployment manifest
│   │   ├── service.yaml         # K8s service definition
│   │   ├── configmap.yaml       # K8s configuration
│   │   └── [Other K8s resources]
│   │
│   ├── licenses/                # Third-party licenses
│   │   ├── [License files for dependencies]
│   │
│   └── scripts/                 # Utility scripts
│       ├── [Build, deployment, utility scripts]
│
└── docs/                        # DOCUMENTATION (This directory)
    ├── ARCHITECTURE.md          # High-level architecture & design
    ├── SYSTEM_DESIGN.md         # Detailed system design & components
    └── REPOSITORY_STRUCTURE.md  # This file - directory structure
```

---

## Directory Structure Summary

### By Component

#### **Agent** (`/agent/`)
- **Purpose**: Runs on monitored systems to collect metrics
- **Key Files**: `agent.go`, `server.go`, `handlers.go`
- **Metrics**: CPU, memory, disk, network, GPU, temperature, containers, SMART
- **Platform Support**: Linux, macOS, Windows, FreeBSD
- **Communication**: SSH server on port 45876

#### **Hub** (`/internal/hub/`)
- **Purpose**: Central monitoring server and web interface
- **Key Files**: `hub.go`, `server.go`, `api.go`
- **Components**: System manager, alert manager, record manager, user manager
- **Database**: PocketBase (SQLite/PostgreSQL/MySQL)
- **Frontend**: Vue.js-based web UI

#### **Shared Code** (`/internal/common/`, `/internal/entities/`)
- **Purpose**: Shared data structures and protocols
- **Protocol**: CBOR serialization, Hub-Agent communication
- **Data Models**: System metrics, container data, SMART data, systemd info
- **Security**: SSH algorithm constraints, encryption parameters

#### **Deployment** (`/supplemental/`)
- **Purpose**: Deployment and distribution files
- **Formats**: Docker, Debian packages, Kubernetes, Guides
- **Configuration**: Examples, templates, documentation

#### **Documentation** (`/docs/`)
- **Purpose**: Project documentation
- **Files**: Architecture, system design, repository structure

---

## Key Files by Function

### Agent Startup
1. `internal/cmd/agent/main.go` - Agent entry point
2. `agent/agent.go` - Agent initialization
3. `agent/server.go` - SSH server startup
4. `agent/handlers.go` - Connection handling

### Hub Startup
1. `internal/cmd/hub/main.go` - Hub entry point
2. `internal/hub/hub.go` - Hub initialization
3. `internal/hub/server.go` - HTTP server setup
4. `internal/hub/api.go` - Route registration

### Metrics Collection (Agent)
1. `agent/system.go` - System details
2. `agent/cpu.go` - CPU metrics
3. `agent/memory.go` - Memory metrics
4. `agent/disk.go` - Disk metrics
5. `agent/network.go` - Network metrics
6. `agent/docker.go` - Container metrics
7. `agent/gpu*.go` - GPU metrics

### Metrics Processing (Hub)
1. `internal/hub/systems/system_manager.go` - System lifecycle
2. `internal/hub/systems/system.go` - Connection & data handling
3. `internal/records/records.go` - Database storage
4. `internal/hub/alerts/alerts.go` - Alert evaluation
5. `internal/hub/users/user_manager.go` - User permissions

### Web Interface
1. `internal/site/src/App.vue` - Main app component
2. `internal/site/src/views/*.vue` - Page components
3. `internal/site/src/components/*.vue` - UI components
4. `internal/site/src/services/*.ts` - API communication

---

## Data Flow Paths

### Real-time Metric Update Flow

```
User → Web UI
  ↓
Subscribe to System via WebSocket
  ↓
internal/site/services/api.ts (Frontend)
  ↓
internal/hub/ws/ (WebSocket handler)
  ↓
internal/hub/systems/system_manager.go (Trigger update)
  ↓
agent/server.go (SSH connection)
  ↓
agent/* collectors (Gather metrics)
  ↓
agent/response.go (Serialize CBOR)
  ↓
internal/hub/systems/system.go (Receive)
  ↓
internal/records/records.go (Store)
  ↓
internal/alerts/alerts.go (Evaluate)
  ↓
internal/hub/ws/ (Broadcast)
  ↓
Web UI Update Display
```

### Alert Trigger Flow

```
internal/records/records.go (New metric)
  ↓
internal/hub/alerts/alerts.go (Evaluate)
  ↓
Check threshold
  ↓
Check quiet hours
  ↓
Check cache (prevent spam)
  ↓
Generate AlertData
  ↓
Multi-channel dispatch
├─ Email → SMTP server
├─ Webhook → HTTP POST
└─ Service → Shoutrrr integration
  ↓
internal/alerts/alerts_history.go (Log)
```

---

## Build Artifacts

### Build Targets

```
make build               # Build agent and hub (default)
make build-agent        # Build agent binary
make build-hub          # Build hub binary (includes web UI)
make build-hub-dev      # Build hub for development

Output: build/
├── beszel-agent_linux_amd64         # Agent binary
├── beszel-agent_darwin_arm64        # Agent macOS
├── beszel-agent_windows_amd64.exe   # Agent Windows
├── beszel_linux_amd64               # Hub binary
├── beszel_darwin_arm64              # Hub macOS
└── beszel_windows_amd64.exe         # Hub Windows
```

### Docker Builds

```
Dockerfiles:
├── dockerfile_agent         # Base agent image
├── dockerfile_agent_alpine  # Minimal agent (Alpine Linux)
├── dockerfile_agent_intel   # Agent with Intel GPU support
├── dockerfile_agent_nvidia  # Agent with Nvidia GPU support
└── dockerfile_hub           # Hub image

Published Images:
├── henrygd/beszel:latest               # Hub
├── henrygd/beszel-agent:latest         # Agent
├── henrygd/beszel-agent:intel          # Agent + Intel GPU
└── henrygd/beszel-agent:nvidia         # Agent + Nvidia GPU
```

---

## Testing Structure

### Test Files Organization

```
Agent Tests:
├── agent/*_test.go              # Unit tests
├── agent/test-data/             # Test fixtures
└── internal/tests/              # Integration tests

Hub Tests:
├── internal/hub/**/*_test.go    # Hub component tests
├── internal/alerts/**/*_test.go # Alert tests
├── internal/records/*_test.go   # Storage tests
└── internal/users/*_test.go     # User tests

Running Tests:
make test                        # Run all tests
go test ./...                    # Run all tests (verbose)
go test -tags=testing ./...      # Run with testing tag
```

---

## Configuration Files

### Build Configuration

- **Makefile**: Build targets and configurations
- **go.mod / go.sum**: Go dependencies
- **i18n.yml**: Internationalization strings

### Frontend Configuration

- **internal/site/vite.config.ts**: Build tool configuration
- **internal/site/tailwind.config.js**: CSS framework
- **internal/site/tsconfig.json**: TypeScript configuration
- **internal/site/package.json**: JavaScript dependencies

### Deployment Configuration

- **supplemental/docker/docker-compose.yml**: Docker Compose setup
- **supplemental/kubernetes/**: Kubernetes manifests
- **supplemental/debian/**: Debian package metadata

---

## Third-Party Integrations

### Go Dependencies (Key)

| Package | Purpose |
|---------|---------|
| pocketbase/pocketbase | Database & REST API |
| gliderlabs/ssh | SSH server |
| gopsutil/v4 | System metrics |
| docker/client | Docker API |
| fxamacker/cbor/v2 | CBOR serialization |
| lxzan/gws | WebSocket |
| nicholas-fedor/shoutrrr | Notification delivery |
| blang/semver | Version parsing |

### Frontend Dependencies (Key)

| Package | Purpose |
|---------|---------|
| Vue 3 | UI framework |
| Vite | Build tool |
| TypeScript | Language |
| Pinia | State management |
| Tailwind CSS | Styling |
| Axios | HTTP client |

---

## Development Workflows

### Adding New Metrics

1. **Define Data Model** → `internal/entities/system/system.go`
2. **Implement Collection** → `agent/<metric>.go`
3. **Add Tests** → `agent/<metric>_test.go`
4. **Update Response** → `agent/response.go`
5. **Database Schema** → `internal/migrations/`
6. **Hub Processing** → `internal/hub/systems/system.go`
7. **Frontend Display** → `internal/site/src/components/`

### Adding New Alert Type

1. **Define Alert Logic** → `internal/alerts/alerts_<type>.go`
2. **Add Tests** → `internal/alerts/alerts_<type>_test.go`
3. **Register Handler** → `internal/alerts/alerts.go`
4. **Update API** → `internal/hub/alerts/alerts_api.go`
5. **Frontend UI** → `internal/site/src/views/Alerts.vue`

### Adding New Notification Channel

1. **Implement** → `internal/alerts/` (new method)
2. **Register** → `internal/alerts/alerts.go` in delivery
3. **Configure** → User settings in database
4. **Test** → `internal/alerts/alerts_test.go`

---

## Summary

Beszel's repository structure is organized as:

| Section | Purpose |
|---------|---------|
| **Root** | Build config, version info, main Dockerfiles |
| **agent/** | Metrics collection engine (distributed) |
| **internal/hub/** | Central server, web API, WebSocket |
| **internal/alerts/** | Alert evaluation and delivery |
| **internal/records/** | Historical data storage |
| **internal/entities/** | Shared data models |
| **internal/site/** | Vue.js web UI |
| **supplemental/** | Deployment files and documentation |
| **docs/** | Architecture and design documentation |

This modular organization enables clear separation of concerns and makes the codebase maintainable and scalable.
