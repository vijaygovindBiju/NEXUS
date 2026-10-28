# IPC & Network Protocol Specification — Nexus Net

---

## 1. Tauri IPC Protocol (Frontend $\leftrightarrow$ Backend)

All communication between the user interface and the core engine occurs over Tauri v2's strongly typed IPC boundary via `window.__TAURI__.core.invoke` (request-response) and `window.__TAURI__.event.listen` (push streams).

---

### 1.1. Commands (Request / Response)

#### 1. `get_network_adapters`
Queries all physical and wireless network adapters connected to the system.
* **Request:** None
* **Response:**
  ```json
  [
    {
      "id": "eth0",
      "name": "Ethernet",
      "display_name": "Intel(R) Ethernet Controller I225-V",
      "ip_addresses": ["192.168.1.150"],
      "gateway": "192.168.1.1",
      "is_up": true,
      "is_wireless": false,
      "link_speed_mbps": 1000,
      "current_metric": 25
    },
    {
      "id": "wlan0",
      "name": "Wi-Fi",
      "display_name": "Intel(R) Wi-Fi 6E AX211 160MHz",
      "ip_addresses": ["192.168.1.151"],
      "gateway": "192.168.1.1",
      "is_up": true,
      "is_wireless": true,
      "link_speed_mbps": 512,
      "current_metric": 10
    }
  ]
  ```

#### 2. `get_live_metrics`
Fetches the latest computed telemetry reading for each candidate adapter.
* **Request:** None
* **Response:**
  ```json
  {
    "Wi-Fi": {
      "latency_ms": 9.2,
      "jitter_ms": 1.1,
      "packet_loss_pct": 0.0,
      "bytes_rx_sec": 67108864,
      "bytes_tx_sec": 19398656
    },
    "Ethernet": {
      "latency_ms": 21.0,
      "jitter_ms": 2.4,
      "packet_loss_pct": 0.0,
      "bytes_rx_sec": 131072000,
      "bytes_tx_sec": 131072000
    }
  }
  ```

#### 3. `set_gaming_lockdown`
Enables or disables strict application firewall isolation.
* **Request:**
  ```json
  {
    "enabled": true,
    "allowed_executables": [
      "valorant.exe",
      "RiotClientServices.exe",
      "vgc.exe",
      "Discord.exe"
    ],
    "timeout_minutes": 15
  }
  ```
* **Response:**
  ```json
  {
    "status": "active",
    "blocked_app_count_estimate": 12,
    "expires_at_unix": 1793235600
  }
  ```

#### 4. `evaluate_optimal_route`
Runs candidate route ranking against a specified objective function.
* **Request:**
  ```json
  {
    "goal": "latency" // "latency" | "balanced" | "throughput" | "stability"
  }
  ```
* **Response:**
  ```json
  {
    "recommended_adapter": "Wi-Fi",
    "explanation": "Wi-Fi has the strongest latency-first score: 9.2 ms RTT, 1.1 ms jitter, and 0.0% sample loss.",
    "rankings": [
      {
        "name": "Wi-Fi",
        "score": 96,
        "latency_score": 96,
        "stability_score": 98,
        "throughput_score": 51
      },
      {
        "name": "Ethernet",
        "score": 75,
        "latency_score": 75,
        "stability_score": 90,
        "throughput_score": 100
      }
    ]
  }
  ```

#### 5. `apply_preferred_route`
Assigns primary routing priority to a designated network adapter.
* **Request:**
  ```json
  {
    "adapter_name": "Wi-Fi",
    "manual_override": true
  }
  ```
* **Response:**
  ```json
  {
    "success": true,
    "active_preferred": "Wi-Fi",
    "metric_assigned": 5
  }
  ```

#### 6. `emergency_unlock`
Trips the safety release, immediately purging all firewall lockdown rules and restoring default interface metrics.
* **Request:** None
* **Response:**
  ```json
  {
    "cleared": true,
    "message": "All firewall restrictions removed; metrics normalized."
  }
  ```

---

### 1.2. Asynchronous Push Events (Backend $\to$ UI)

The frontend registers listeners for real-time reactive updates:

```javascript
import { listen } from '@tauri-apps/api/event';

// 1. Live stream of probe telemetry
await listen('telemetry://tick', (event) => {
  const { adapter, latency, jitter, loss } = event.payload;
  updateChart(adapter, latency);
});

// 2. Notification when an optimal route switch occurs
await listen('route://auto-switched', (event) => {
  const { new_adapter, old_adapter, reason } = event.payload;
  showToast(`Auto-switched to ${new_adapter}: ${reason}`);
});

// 3. Process detection notification
await listen('game://detected', (event) => {
  const { game_title, profile } = event.payload;
  promptStartSession(game_title, profile);
});
```

---

## 2. Telemetry Probe Protocol

Probing must be non-intrusive and low-overhead to avoid causing the very latency jitter the tool is designed to eliminate.

### Probe Specifications:
1. **Target Endpoints:**
   * Global Anycast DNS: `1.1.1.1` (Cloudflare), `8.8.8.8` (Google).
   * Game Region Endpoints: Valve CS2 datacenters, Riot AWS edge relays.
2. **Packet Structure:**
   * Unprivileged UDP echo / DNS query packet size: 48 bytes.
   * Transmit Interval: 2.0 seconds while idle; 1.0 second during active gaming session.
3. **Rolling Window:**
   * Buffer Size: 30 rolling samples per adapter.
   * Jitter calculation uses Standard Deviation ($\sigma$) rather than simple delta to discount single-sample anomalies.

---

## 3. Watchdog & Safety Lease Protocol

To prevent accidental denial of service (lockout) if the application crashes or encounters an unhandled exception:

```
┌─────────────────────────────────────────────────────────────────┐
│                      WATCHDOG LEASE STATE                       │
├─────────────────────────────────────────────────────────────────┤
│ Active: true                                                    │
│ Lease Duration: 15 Minutes (Configurable: 5m, 15m, 30m, 60m)    │
│ Heartbeat Interval: 5 Seconds                                   │
│ Missed Heartbeat Threshold: 3 consecutive failures              │
│ Action on Expiry / Failure: Automatic flush of all firewall     │
│                             rules and metric normalization.     │
└─────────────────────────────────────────────────────────────────┘
```

1. **Linux:** The `nftables` table contains a TTL and is linked to the process PID via a UNIX socket. If the daemon process closes, the OS kernel removes transient tables.
2. **Windows:** WFP filters are opened strictly using `FWPM_SESSION_FLAG_DYNAMIC`. The Windows BFE (Base Filtering Engine) cleans up all dynamic rules immediately upon process termination.

---

## 4. Configuration & Profile File Format (`.json`)

Community and custom game profiles adhere to the following schema:

```json
{
  "$schema": "https://nexusnet.dev/schemas/profile.v1.json",
  "name": "VALORANT Low-Latency",
  "version": "1.2.0",
  "game": "VALORANT",
  "target_executables": [
    "VALORANT-Win64-Shipping.exe",
    "RiotClientServices.exe",
    "vgc.exe"
  ],
  "recommended_objective": "latency",
  "suggested_allowlist": [
    "Selected game",
    "Game launcher",
    "Voice chat",
    "Anti-cheat / helper"
  ],
  "adapter_preferences": {
    "band": "5 GHz",
    "power_saving": "Maximum Performance"
  },
  "author": "Nexus Community Review",
  "verified": true
}
```
