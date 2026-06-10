# Beszel System Design

## Table of Contents

1. [Core Systems](#core-systems)
2. [Metrics Collection System](#metrics-collection-system)
3. [Alert System](#alert-system)
4. [Data Persistence](#data-persistence)
5. [WebSocket & Real-time Updates](#websocket--real-time-updates)
6. [Configuration System](#configuration-system)
7. [Error Handling & Recovery](#error-handling--recovery)
8. [Testing Architecture](#testing-architecture)

---

## Core Systems

### 1. System Manager (`internal/hub/systems/`)

**Responsibility**: Manages lifecycle of monitored systems and their connections.

#### Key Components:

**SystemManager**:
- Maintains store of active `System` objects
- Manages SSH connections to agents
- Handles system add/remove/pause operations
- Coordinates data collection updates
- Implements exponential backoff for reconnections

**System**:
- Represents a single monitored system
- Stores connection details (host, port)
- Maintains current status (up/down/paused/pending)
- Tracks system metadata and fingerprint
- Manages WebSocket connections for real-time updates

**Connection Lifecycle**:
```
Create System
    ↓
Initialize SSH Connection
    ↓
Verify Fingerprint
    ↓
Request System Data
    ↓
Store Results
    ↓
Broadcast via WebSocket
    ↓
Schedule Next Update
    ↓
Repeat
```

#### Event Hooks:

The system manager binds to PocketBase events:
- `OnRecordBeforeCreate`: Validate system config
- `OnRecordBeforeDelete`: Cleanup connections
- `OnRecordAfterUpdate`: Apply config changes
- `OnModelAfterCreate`: Initialize monitoring
- Cron jobs for periodic health checks

#### Fingerprint Verification:

```
First Connection:
1. Hub requests fingerprint from agent
2. Agent signs request with SSH key
3. Hub verifies signature matches public key
4. Hub stores fingerprint hash
5. Future connections verify against stored hash
   (Detects man-in-the-middle attacks)
```

### 2. Alert Manager (`internal/alerts/`)

**Responsibility**: Evaluate metrics against thresholds and deliver notifications.

#### Alert Types:

**System Alerts**:
- CPU usage exceeds threshold
- Memory usage exceeds threshold
- Disk usage exceeds threshold
- Bandwidth exceeds threshold
- Temperature exceeds threshold
- Load average exceeds threshold
- Battery charge too low
- Disk health degraded

**Status Alerts**:
- System comes online (after being down)
- System goes offline
- System monitoring paused/resumed

#### Alert Processing Pipeline:

```
New System Data Received
    ↓
Load Alert Configuration for System
    ↓
Create Alert Evaluation Context
    ├─ For each alert type:
    │  ├─ Compare value against threshold
    │  ├─ Check trigger count (prevents flapping)
    │  └─ Track consecutive triggers
    ├─ Check quiet hours (skip if in window)
    ├─ Check alert cache (skip if recent)
    └─ Generate alert if triggered
    ↓
Serialize Alert Data
    ↓
Multi-Channel Delivery
    └─ Cache alert to prevent spam
```

#### Notification Channels:

1. **Email**
   - SMTP configuration required
   - Template-based messages
   - Recipient list from user preferences

2. **Webhooks**
   - HTTP POST to configured URL
   - JSON payload with alert details
   - Supports basic authentication

3. **Notification Services (Shoutrrr)**
   - Discord, Slack, Telegram
   - PushBullet, IFTTT
   - 20+ other services supported
   - Service-specific formatting

#### Alert Cache:

```go
// Prevents alert spam for same condition
type CachedAlertData struct {
    LastAlert     time.Time
    Count         uint8
    CacheDuration time.Duration
}

// If cache not expired and condition persists
// → Skip sending notification
// → Update cache timestamp
```

#### Quiet Hours:

```
Configuration: "22:00-06:00"
    ↓
Check current time against window
    ↓
If within quiet hours and not critical
    ├─ Suppress notification
    └─ Still log to alert history
```

#### Alert History:

- Stores all alert events to database
- Tracks triggered/resolved alerts
- Provides audit trail
- Enables historical trend analysis

### 3. Record Manager (`internal/records/`)

**Responsibility**: Store and manage historical metrics data.

#### Data Models:

**System Stats** (time-series data):
```
Record:
  - SystemID: Reference to monitored system
  - Timestamp: When metrics were collected
  - Stats: JSON blob containing:
    - CPU usage %
    - Memory used/total
    - Disk usage by partition
    - Network bytes in/out
    - GPU utilization
    - Temperature readings
    - Load average
    - Battery charge
```

**S.M.A.R.T. Records**:
```
Record:
  - SystemID: Reference to system
  - DeviceID: Device identifier
  - Data: Raw S.M.A.R.T. attributes
  - Status: OK/Warning/Critical
  - Timestamp: Last update
```

**Container Stats**:
```
Record:
  - SystemID: Reference to system
  - ContainerID: Container identifier
  - Stats: Container-specific metrics
  - Timestamp: Collection time
```

#### Data Aggregation:

Records support averaging and aggregation:
- Calculate moving averages (1h, 6h, 24h)
- Peak usage detection
- Trend analysis
- Data cleanup/retention policies

#### Performance Considerations:

- Indexed by SystemID and Timestamp for fast queries
- Configurable retention (e.g., keep 1 year)
- Batch inserts for multiple metrics
- Archive old data to reduce storage

### 4. User Manager (`internal/users/`)

**Responsibility**: Multi-user support and permission management.

#### User Roles:

**Admin**:
- Create/delete users
- View all systems
- Modify system settings
- Configure hub settings
- Manage alerts globally

**Regular User**:
- View owned systems
- View shared systems
- Configure personal alerts
- Modify personal settings

#### System Visibility:

```
System Types:
├─ Public: Visible to all users
├─ Private: Visible only to owner
└─ Shared: Visible to specific users

User Access:
├─ View: Can see system dashboard
├─ Edit: Can modify system settings
└─ Delete: Can remove system
```

#### Authentication:

1. **Local Passwords**:
   - Bcrypt hashing
   - Configurable minimum requirements
   - Password reset via email

2. **OAuth2/OIDC**:
   - Google, GitHub, Microsoft, etc.
   - External identity provider integration
   - JIT (Just-In-Time) user provisioning

3. **API Tokens**:
   - Personal access tokens
   - Programmatic access
   - Token expiration support

---

## Metrics Collection System

### Agent-Side Collection

#### Collection Flow:

```
Agent Startup
    ↓
Initialize Collectors
├─ CPU Collector
├─ Memory Collector
├─ Disk Collector
├─ Network Collector
├─ GPU Collector
├─ Temperature Collector
├─ Docker/Podman Collector
└─ Other Platform-Specific Collectors
    ↓
Hub Connects via SSH
    ↓
Hub Requests Data
    ↓
Collection Pipeline Runs
    ├─ Each collector gathers data
    ├─ Apply delta tracking for rates
    └─ Aggregate into CombinedData
    ↓
Serialize to CBOR
    ↓
Send to Hub
```

#### Per-Collector Details:

**CPU Collector**:
- Global CPU usage percentage
- Per-core usage percentages
- Load average (1, 5, 15 minutes)
- Thread count

**Memory Collector**:
- Physical RAM (used/total)
- Swap (used/total)
- ZFS ARC cache (if present)
- Configurable calculation methods

**Disk Collector**:
- Per-partition usage (used/total)
- Per-partition I/O rates
- Caching to avoid waking sleeping disks
- S.M.A.R.T. health status

**Network Collector**:
- Per-interface bandwidth
- Delta tracking for rate calculation
- Handles both real and virtual interfaces
- Unix socket and TCP socket support

**GPU Collector**:
- Nvidia: nvidia-smi parsing
- AMD: rocm-smi parsing
- Intel: GPU monitoring API
- Tegrastats for Nvidia Jetson
- Power draw reporting

**Temperature Collector**:
- Platform-specific implementations:
  - Linux: hwmon via sysfs
  - macOS: IOKit APIs
  - Windows: WMI queries
- Per-sensor readings
- Alert thresholds

**Container Collector**:
- Docker API polling
- Per-container metrics
- Container lifecycle tracking
- Log streaming capability

#### Caching Strategy:

```go
type SystemDataCache struct {
    // Cache keyed by interval duration
    data map[uint16]CachedData
    
    // Example intervals: 10s, 30s, 60s, 5min
    // Each interval cached independently
}

// Usage:
cache.Get(60000 /* ms */) // Returns 60s cached data
cache.Invalidate()        // Clears all caches on demand
```

**Benefits**:
- Prevents repeated expensive system calls
- Supports multiple update rates
- Allows clients to request different intervals
- Respects disk sleep settings

#### Delta Tracking:

```
Track Previous Values
    ↓
Compare Current vs Previous
    ↓
Calculate Rate
    = (Current - Previous) / Time Delta
    ↓
Update Previous Values
    ↓
Include Rate in Response
```

**Tracked Metrics**:
- Network bytes sent/received → bandwidth
- Disk read/write bytes → disk I/O rate
- Disk operations → IOPS

---

## Alert System

### Design Details

#### Alert Evaluation:

```go
type SystemAlertData struct {
    Name        string  // Alert name (CPU, Memory, etc.)
    Value       float64 // Current value
    Threshold   float64 // Alert threshold
    Triggered   bool    // Whether alert is triggered
    Count       uint8   // Consecutive triggers
    Min         uint8   // Minimum triggers before alert
}
```

**Trigger Logic**:
```
if value > threshold {
    count++
    if count >= min {
        triggered = true
    }
}
else {
    count = 0
    triggered = false
}
```

This prevents flapping from transient spikes.

#### Alert Caching:

```
Alert triggers at 10:00:00
    ↓
Notification sent
    ↓
Cache entry: (SystemID, AlertType) → 10:00:00
    ↓
Alert condition persists at 10:01:00
    ↓
Check cache: Within cache duration?
    ├─ Yes: Skip notification
    └─ No: Send notification, update cache
```

#### Multi-Channel Example:

```
Alert: CPU > 90% on server-01
    ↓
User has alerts enabled for:
    ├─ Email: send@example.com
    ├─ Slack: #monitoring
    └─ Webhook: https://custom.api/alerts
    ↓
Concurrent delivery:
    ├─ Email service handles send
    ├─ Slack API call posted
    └─ Webhook HTTP POST executed
    ↓
All succeed → Log success
Any fail → Retry with backoff
```

#### Status Alerts:

```
System State Change:
├─ Down → Up
├─ Up → Down
├─ Paused → Running
└─ Running → Paused
    ↓
Trigger status alert
    ↓
Notification sent
    ↓
Alert history updated
```

---

## Data Persistence

### Database Schema

#### Primary Collections:

**systems**:
```
- id: unique identifier
- name: display name
- host: hostname/IP
- port: SSH port
- status: up/down/paused/pending
- public: boolean (visibility)
- updated: last update timestamp
- agent_version: version of agent
```

**system_stats**:
```
- id: unique identifier
- system_id: reference to system
- stats: JSON object containing metrics
- created: timestamp
```

**users**:
```
- id: unique identifier
- username: unique username
- email: user email
- password_hash: bcrypt hash
- created: account creation time
- updated: last modification time
```

**alerts**:
```
- id: unique identifier
- system_id: reference to system
- user_id: alert owner
- type: cpu/memory/disk/etc
- threshold: trigger value
- enabled: active/inactive
- settings: JSON with options
```

**system_shares**:
```
- id: unique identifier
- system_id: reference to system
- user_id: recipient user
- permission: view/edit/delete
```

#### Indexing Strategy:

- Primary key: id (unique)
- Foreign keys: system_id, user_id
- Compound indexes:
  - (system_id, created) for time-series queries
  - (user_id, system_id) for user visibility
  - (status, updated) for status queries

### Backup System

**Automatic Backups**:
- Scheduled backups (hourly/daily configurable)
- Local filesystem storage
- S3-compatible storage (Minio, AWS S3, DigitalOcean Spaces)

**Backup Contents**:
- SQLite database (or export from PostgreSQL/MySQL)
- Configuration files
- SSH key data
- System fingerprints

**Restore Procedures**:
- Point-in-time restore capability
- Verification of backup integrity
- Transaction consistency

---

## WebSocket & Real-time Updates

### WebSocket Architecture

#### Connection Lifecycle:

```
Client connects via WebSocket
    ↓
Subscribe to System
    ├─ Send subscription message
    └─ Register in websocket manager
    ↓
Receive real-time updates
    ├─ On each data collection
    └─ Hub sends CombinedData via WebSocket
    ↓
Client disconnects
    ├─ Unregister subscription
    └─ Cleanup resources
```

#### Update Flow:

```
Hub receives agent response
    ↓
Parse CombinedData
    ↓
Store to database
    ↓
Check subscriptions
    ├─ Who is viewing this system?
    └─ For each subscriber:
       ├─ Format update message
       ├─ Serialize to CBOR/JSON
       └─ Send via WebSocket
       
Client receives update
    ↓
Update local state
    ↓
Re-render UI
```

#### Message Types:

```
Subscribe: { action: "subscribe", system_id: "abc" }
Unsubscribe: { action: "unsubscribe", system_id: "abc" }
Update: { action: "update", system_id: "abc", data: {...} }
Status: { action: "status", system_id: "abc", status: "up" }
```

---

## Configuration System

### Environment Variables

**Agent Configuration**:
```
LISTEN_ADDR=:45876          # SSH listen address
LISTEN_NETWORK=tcp          # tcp or unix
DATA_DIR=/var/lib/beszel    # Data directory
MEM_CALC=system             # Memory calculation method
LOG_LEVEL=info              # debug/info/warn/error
DISABLE_SSH=false           # Disable SSH server
DISK_USAGE_CACHE=15m        # Cache disk usage to avoid waking disks
```

**Hub Configuration**:
```
BESZEL_DATA_DIR=/data/beszel
BESZEL_ENCRYPTION_KEY=...
BESZEL_ADMIN_EMAIL=admin@example.com
BESZEL_ADMIN_PASSWORD=...
BESZEL_PUBLIC_URL=https://beszel.example.com

# Email configuration
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USERNAME=...
SMTP_PASSWORD=...
SMTP_FROM=notifications@example.com

# OAuth configuration
OAUTH_PROVIDERS_JSON=...
```

### Configuration Files

**config.yaml** (System setup):
```yaml
systems:
  - name: "web-server-01"
    host: "192.168.1.100"
    port: 45876
    public: true
    
  - name: "database-01"
    host: "db.internal"
    port: 45876
    public: false
```

---

## Error Handling & Recovery

### Connection Failures

#### Retry Strategy:

```
Connection attempt 1: Immediate
    ↓ Fails
    ↓
Wait 2 seconds
    ↓
Connection attempt 2
    ↓ Fails
    ↓
Wait 4 seconds (exponential backoff)
    ↓
Connection attempt 3
    ↓ Fails
    ↓
Wait 8 seconds
    ↓
...
    ↓
Max backoff: 2 minutes
    ↓
If reconnected: Reset backoff to start
```

**Status Transitions**:
```
Connected → Disconnected (Immediate)
    ↓
Pending (trying to reconnect)
    ↓
Down (if no success after grace period)
    ↓
Status alert triggered
```

### Data Validation

**Agent Response Validation**:
```
Receive CBOR data
    ↓
Deserialize
    ↓
Validate structure
├─ Required fields present
├─ Values in valid ranges
└─ Timestamps reasonable
    ↓
Valid: Process and store
Invalid: Log error, skip
```

### Graceful Degradation

**Partial Data**:
- If some collectors fail, others still run
- Missing data points handled gracefully
- Historical data used for missing values

**Service Degradation**:
- WebSocket connection lost: Client retries
- Database unavailable: Queue updates in memory
- Alert delivery failed: Retry with backoff

---

## Testing Architecture

### Test Organization

**Unit Tests** (`*_test.go` files):
```
agent/
├─ system_test.go        # CPU, memory collection
├─ network_test.go       # Network interface tracking
├─ docker_test.go        # Container metrics
├─ gpu_test.go           # GPU data collection
└─ ...

internal/hub/
├─ systems/system_test.go
├─ alerts/alerts_test.go
└─ ...
```

### Test Data

**Fixtures** (`test-data/`):
```
test-data/
├─ system_info.json      # Mock system info
├─ container.json        # Mock container data
├─ nvtop.json           # Mock GPU output
├─ amdgpu.ids           # AMD GPU device info
└─ smart/               # S.M.A.R.T. test data
```

### Testing Strategies

**Mocking**:
- Mock Docker API responses
- Mock GPU command outputs
- Mock filesystem operations
- Mock SSH connections

**Fixtures**:
- Real system output examples
- Edge cases (errors, timeouts)
- Large data sets for performance testing

**Integration Tests**:
- Hub-to-Agent communication
- Full metrics collection cycle
- Alert triggering
- Database operations

---

## Performance Optimization

### Caching Layers

1. **Agent-side**: Per-interval caching (60s default)
2. **Connection reuse**: Persistent SSH connections possible
3. **Delta tracking**: Avoid recalculating bandwidth/IOPS
4. **WebSocket batching**: Bundle updates

### Query Optimization

- Index system_id for fast lookups
- Index timestamps for time-series queries
- Batch database inserts
- Connection pooling for database

### Resource Limits

- SSH idle timeout: 70 seconds
- Max concurrent connections: Configurable
- Memory caching: LRU eviction
- Log file rotation: Prevent disk fill

---

## Summary

Beszel's system design prioritizes:

| Aspect | Implementation |
|--------|-----------------|
| **Reliability** | Exponential backoff, graceful degradation |
| **Performance** | Multi-layer caching, delta tracking, batch operations |
| **Scalability** | Distributed metrics collection, efficient serialization |
| **Maintainability** | Modular components, clear separation of concerns |
| **Testability** | Comprehensive test data, mocking capabilities |

This design enables Beszel to efficiently collect, store, and deliver metrics at scale while remaining lightweight and reliable.
