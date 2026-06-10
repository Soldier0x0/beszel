# Beszel Architecture

## Overview

Beszel is a lightweight, distributed server monitoring platform designed with a **Hub-and-Agent** architecture. The system enables centralized monitoring of multiple systems with a web-based dashboard, historical data tracking, and sophisticated alert management.

**Current Version**: 0.18.7

### Key Design Philosophy

- **Lightweight**: Minimal resource consumption on monitored systems
- **Distributed**: Decentralized agent-based monitoring model
- **Secure**: SSH-based communication with public key authentication
- **Scalable**: Support for monitoring many systems from a single hub
- **Feature-rich**: Comprehensive metrics including Docker/Podman containers, GPU monitoring, SMART data, and more

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Hub (Web Application)                 │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  PocketBase Core                                         ││
│  │  - Database (SQLite/PostgreSQL/MySQL)                   ││
│  │  - REST API                                              ││
│  │  - User Authentication (Password, OAuth, OIDC)          ││
│  │  - Real-time Sync with WebSockets                        ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Beszel Hub Services                                     ││
│  │  - System Manager: Manages agent connections & updates  ││
│  │  - Alert Manager: Triggers and delivers alerts           ││
│  │  - Record Manager: Stores historical metrics             ││
│  │  - User Manager: Multi-user management & permissions     ││
│  │  - Heartbeat Monitor: Detects system offline/online      ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Web UI & APIs                                           ││
│  │  - Dashboard (Vue.js-based frontend)                    ││
│  │  - System pages with real-time updates                   ││
│  │  - Alert configuration & management                      ││
│  │  - User & system administration                          ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │ SSH Tunneling
                            │ (Encrypted Data Transfer)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Agent (Multiple Instances)                 │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  SSH Server (Port 45876 by default)                      ││
│  │  - Public key authentication                             ││
│  │  - Secure channel for communication                       ││
│  │  - Idle timeout: 70 seconds                              ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Metrics Collection Engines                              ││
│  │  - CPU Monitor          - GPU Manager                    ││
│  │  - Memory Monitor       - SMART Manager                  ││
│  │  - Disk I/O Monitor     - Systemd Monitor               ││
│  │  - Network Monitor      - Battery Manager                ││
│  │  - Temperature Sensors  - ZFS Manager                    ││
│  │  - Container Manager    - Docker/Podman API              ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │  Data Cache & Delta Tracking                             ││
│  │  - Per-cache-time statistics aggregation                 ││
│  │  - Delta calculation for bandwidth/disk I/O              ││
│  │  - In-memory caching with configurable TTL               ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. Hub Architecture

The **Hub** is the central monitoring server built on PocketBase, providing:

#### A. PocketBase Foundation
- **Database**: Flexible backend supporting SQLite, PostgreSQL, MySQL
- **Authentication**: Built-in user management with OAuth2/OIDC support
- **REST API**: Auto-generated APIs for all data models
- **Realtime Sync**: WebSocket-based live data updates

#### B. System Manager
Manages all connected monitoring agents:
- Maintains SSH client configuration
- Establishes connections to agents
- Handles system lifecycle (add, remove, pause, resume)
- Tracks system status (up, down, paused, pending)
- Manages system reconnection with exponential backoff
- Stores last agent fingerprint for security verification

#### C. Alert Manager
Sophisticated alerting system with:
- Multi-channel delivery (Email, Webhooks, custom notification services via Shoutrrr)
- Alert caching to prevent notification spam
- Configurable thresholds for CPU, memory, disk, bandwidth, temperature, load average, battery, and disk health
- Quiet hours support to prevent alerts during configured time windows
- Alert history tracking
- Status change notifications (system online/offline)

#### D. Record Manager
Historical data storage:
- Stores metrics from agent responses
- Tracks CPU, memory, disk, network, GPU, temperature, load average, battery
- Supports data averaging and aggregation
- Maintains S.M.A.R.T. health data
- Stores Docker/Podman container metrics

#### E. User Manager
Multi-user support:
- User authentication (local password, OAuth, OIDC)
- Permission management
- System sharing between users
- Admin capabilities for system management

#### F. Heartbeat Monitor
Connection health monitoring:
- Periodic health checks to agents
- Tracks offline/online transitions
- Triggers status-change alerts
- Prevents false positives with grace periods

### 2. Agent Architecture

The **Agent** runs on each monitored system with minimal dependencies:

#### A. SSH Server
- Listens on configurable port (default: 45876) or Unix socket
- Uses ED25519 public key authentication
- Implements secure SSH with restricted algorithms:
  - **Key Exchange**: curve25519-sha256
  - **MAC**: hmac-sha2-256-etm@openssh.com
  - **Cipher**: chacha20-poly1305@openssh.com
- Disables PTY (no interactive shell)
- Auto-closes idle connections after 70 seconds

#### B. Metrics Collection
Unified interface collecting from multiple sources:

**System Metrics**:
- CPU usage (overall and per-core)
- Memory usage (including swap and ZFS ARC)
- Load average (1/5/15 minute)

**Disk Metrics**:
- Disk usage per partition
- Disk I/O statistics (reads/writes, bytes transferred)
- S.M.A.R.T. health status (including eMMC wear and mdraid health)

**Network Metrics**:
- Per-interface bandwidth usage
- Bytes sent/received
- Support for both TCP and Unix sockets

**Hardware Metrics**:
- Temperature sensors (via platform-specific APIs)
- GPU utilization and power draw (Nvidia/AMD/Intel)
- Battery charge percentage and status
- ZFS ARC memory usage

**Container Metrics**:
- Docker and Podman container monitoring
- Per-container CPU, memory, and network usage
- Container status and lifecycle events
- Container log streaming

**System Services**:
- Systemd service status monitoring
- Service enable/disable state

#### C. Data Cache System
Sophisticated caching mechanism:
- **Cache Intervals**: Configurable time-based caching (e.g., 60 seconds)
- **Delta Tracking**: Per-cache-interval calculation of bandwidth and disk I/O
- **Disk Usage Caching**: Separate configurable cache to avoid waking sleeping disks
- **Per-Interface Tracking**: Individual delta trackers for each network interface
- **Snapshot Management**: Maintains previous cycle snapshots for accurate calculations

#### D. Platform Abstraction
Supports multiple operating systems:
- **Linux**: Full feature support including Docker, systemd, mdraid, ZFS
- **macOS**: GPU monitoring (PowerMetrics), battery status, system sensors
- **Windows**: GPU monitoring (Nvidia), S.M.A.R.T. via third-party tools
- **FreeBSD**: ZFS support, full monitoring capabilities

---

## Communication Protocol

### Request/Response Model

Communication uses **CBOR (Concise Binary Object Representation)** for efficient serialization:

#### 1. Hub Request Types

```go
type HubRequest[T any] struct {
    Action WebSocketAction  // Request type
    Data   T               // Request-specific data
    Id     *uint32         // Optional request ID
}

// Actions:
const (
    GetData         = iota // Request system data from agent
    CheckFingerprint       // Verify agent identity
    GetContainerLogs       // Fetch container logs
    GetContainerInfo       // Get container details
    GetSmartData          // Retrieve S.M.A.R.T. data
    GetSystemdInfo        // Get systemd service info
)
```

#### 2. Agent Response Structure

```go
type AgentResponse struct {
    Id          *uint32                    // Request ID (for correlation)
    SystemData  *system.CombinedData       // System metrics
    Fingerprint *FingerprintResponse       // Identity verification
    Error       string                     // Error message if failed
    String      *string                    // Text responses
    SmartData   map[string]smart.SmartData // S.M.A.R.T. data
    ServiceInfo systemd.ServiceDetails     // Systemd data
    Data        cbor.RawMessage            // Generic response payload
}
```

#### 3. System Data Structure

The `CombinedData` structure contains:
- **CPU Stats**: Usage percentage, per-core data
- **Memory Stats**: Used, total, swap, ZFS ARC
- **Disk Stats**: Capacity, usage per partition, I/O rates
- **Network Stats**: Bytes sent/received per interface
- **GPU Data**: Utilization, power draw, memory for each GPU
- **Temperature Data**: Per-sensor readings
- **Load Average**: 1, 5, 15 minute averages
- **Battery Info**: Charge percentage, status
- **Container Data**: Stats for all running containers
- **Disk Health**: S.M.A.R.T. status for each disk

#### 4. Hub-Agent Connection Flow

```
1. Hub connects to Agent via SSH
   ├─ Public key authentication
   └─ Establishes secure channel

2. Hub sends HubRequest (GetData action)
   ├─ Serializes request to CBOR
   └─ Sends via SSH channel

3. Agent receives request
   ├─ Collects system metrics
   ├─ Applies caching strategy
   └─ Serializes response to CBOR

4. Hub receives AgentResponse
   ├─ Deserializes CBOR response
   ├─ Updates database with metrics
   ├─ Evaluates alert conditions
   └─ Broadcasts updates via WebSocket

5. Hub closes SSH session
   └─ Connection reestablished on next update interval
```

---

## Data Flow Architecture

### Metrics Collection Pipeline

```
Agent Metrics Collection
    ↓
CBOR Serialization
    ↓
SSH Transmission
    ↓
Hub Reception
    ↓
Response Parsing
    ↓
Database Storage
    ↓
Alert Evaluation
    ↓
WebSocket Broadcast to Clients
    ↓
Web UI Display
```

### Alert Evaluation Pipeline

```
New Metrics Received
    ↓
Retrieve Alert Rules
    ↓
Compare Against Thresholds
    ↓
Check Quiet Hours
    ↓
Check Alert Cache
    ├─ If cached (avoid spam) → Skip
    └─ If new → Continue
    ↓
Generate Alert
    ↓
Multi-Channel Delivery
├─ Email
├─ Webhooks
├─ Notification Services (Shoutrrr)
└─ Status change alerts
    ↓
Log to Alert History
```

---

## Security Architecture

### Authentication & Authorization

1. **Agent Authentication**
   - SSH public key authentication
   - Hub public key registered on agent
   - Fingerprint verification prevents man-in-the-middle attacks

2. **Hub Authentication**
   - OAuth2 / OIDC support
   - Local password authentication (optional)
   - JWT token-based API access
   - Per-user system visibility

3. **Encryption**
   - SSH channel encryption (ChaCha20-Poly1305)
   - TLS for web interface
   - Secure algorithm selection limiting to modern, audited ciphers

### Permission Model

- **Public Systems**: Shared across all users
- **Private Systems**: Owned by specific user
- **Admin Systems**: Only admins can modify settings
- **System-Level Permissions**: View, edit, delete capabilities per user

---

## Deployment Architectures

### Single-Hub Deployment

```
┌──────────────────┐
│   Beszel Hub     │
│  (Docker/Binary) │
└──────┬───────────┘
       │
       ├─────→ Agent on Server A
       ├─────→ Agent on Server B
       └─────→ Agent on Server C
```

### Multi-Hub Deployment (Future)

```
┌──────────────────┐    ┌──────────────────┐
│   Beszel Hub 1   │    │   Beszel Hub 2   │
│   (Primary)      │    │  (Backup/Federated)
└──────┬───────────┘    └──────┬───────────┘
       │                       │
       ├─────→ Agent on Server A
       ├─────→ Agent on Server B
       └─────→ Agent on Server C
```

### Docker-Based Deployment

```
┌─────────────────────────────────────────┐
│    Docker Network / Compose Stack       │
├─────────────────────────────────────────┤
│  ┌──────────────┐   ┌────────────────┐  │
│  │ Beszel Hub   │───│ Data Volume    │  │
│  │ (Container)  │   │ (SQLite/Backup)│  │
│  └──────────────┘   └────────────────┘  │
│  ┌──────────────┐   ┌────────────────┐  │
│  │ Beszel Agent │───│ Agent Config   │  │
│  │ (Container)  │   │ Volume         │  │
│  └──────────────┘   └────────────────┘  │
└─────────────────────────────────────────┘
```

---

## Performance Characteristics

### Resource Usage

**Hub**:
- Memory: ~80-150MB base, grows with metrics history
- CPU: Minimal when idle, spikes during metric processing
- Disk: ~1-5MB per 1000 metrics (depends on storage frequency)

**Agent**:
- Memory: ~15-30MB base, ~1-2MB per 100 containers monitored
- CPU: <1% idle, spikes during metrics collection (~2-5% peak)
- Network: ~5-50KB per connection (depends on system complexity)

### Update Intervals

- **Default**: 60 seconds (60,000ms) per system
- **Configurable**: Via agent cache duration settings
- **WebSocket Push**: Real-time for client subscriptions
- **Database**: Historical data stored based on retention policy

---

## Extension Points

### Custom Metrics Collection

Agents support pluggable metrics collection via:
- Environment-based sensor configuration
- Custom script execution hooks (future)
- Plugin architecture for platform-specific metrics

### Notification Integrations

Hub supports multiple notification channels:
- SMTP Email
- Webhooks (HTTP POST)
- Shoutrrr integrations:
  - Slack, Discord, Telegram
  - PushBullet, IFTTT
  - And 20+ other services

### API Extensions

PocketBase-based API can be extended with:
- Custom hooks and middleware
- New collection types
- External integrations

---

## Version Compatibility

The system maintains backward compatibility within major versions:

- **CBOR Protocol**: Minimum version 0.12.0
- **Agent Response Format**: Minimum version 0.13.0 for structured responses
- **Hub-Agent**: Negotiated via SSH client version string

Version mismatches trigger automatic fallback to legacy communication modes.

---

## Summary

Beszel's architecture balances:

| Aspect | Approach |
|--------|----------|
| **Reliability** | Multiple reconnection strategies, health monitoring |
| **Performance** | Caching, delta tracking, efficient CBOR encoding |
| **Security** | SSH public key auth, encrypted channels, secure algorithms |
| **Scalability** | Distributed agents, horizontal scaling of systems |
| **Usability** | Web UI, real-time WebSocket updates, multi-user support |
| **Maintainability** | Clean separation of concerns, modular components |

This design enables Beszel to be a lightweight yet capable monitoring solution suitable for small deployments to large distributed systems.
