# Nexus Net ⚡

> **Cross-Platform Gaming Network Optimizer & Traffic Isolation Engine**  
> *Lower ping. Eliminate jitter. Keep distractions out.*

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Rust](https://img.shields.io/badge/Rust-1.80%2B-orange.svg)](https://www.rust-lang.org)
[![Tauri](https://img.shields.io/badge/GUI-Tauri%20v2-24C8D8.svg)](https://v2.tauri.app)
[![Platform: Windows & Linux](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20(SteamOS)-informational.svg)]()
[![Anti-Cheat: 100% Compliant](https://img.shields.io/badge/Anti--Cheat-Vanguard%20%7C%20EAC%20Safe-brightgreen.svg)]()

---

## 📌 Overview

**Nexus Net** is a high-performance, cross-platform desktop application designed to optimize network routing and isolate gaming traffic for competitive and latency-sensitive games (such as *VALORANT*, *Counter-Strike 2*, *Apex Legends*, and *Fortnite*).

Unlike cloud-routed commercial tunneling VPNs that introduce latency overhead and require monthly subscriptions, Nexus Net acts directly on the local operating system's network stack:
1. **Real-time multi-interface probing:** Continuously evaluates telemetry across all active physical network adapters (Ethernet, Wi-Fi 5/6, and 5G USB tethering).
2. **Dynamic Route Scoring:** Automatically assigns optimal metric priorities to preferred network routes without dropping active connections.
3. **Gaming Lockdown:** Enforces non-intrusive traffic isolation via native OS firewall filtering (Windows Filtering Platform / Linux `nftables`), preventing background cloud backups, Windows updates, and browser downloads from inducing bufferbloat.
4. **Kernel Watchdog:** Guaranteed failsafe timer releases all rules if the application or game terminates unexpectedly.

---

## 🏗️ Architecture & Engine Overview

```text
┌────────────────────────────────────────────────────────────────────────┐
│                   Nexus Net UI (Tauri v2 / WebKit)                    │
│    Dashboard  ·  Networks  ·  Route Optimizer  ·  Lockdown  ·  Lab     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Tauri IPC (JSON Commands & Events)
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   Nexus Net Rust Core Engine                           │
│                                                                        │
│   ┌────────────────────┐   ┌────────────────────┐   ┌──────────────┐   │
│   │ Telemetry Engine   │   │ Multi-Route Scorer │   │ Watchdog     │   │
│   │ Async ICMP & UDP   │   │ Latency / Jitter   │   │ Failsafe     │   │
│   └─────────┬──────────┘   └─────────┬──────────┘   └──────┬───────┘   │
└─────────────┼────────────────────────┼─────────────────────┼───────────┘
              │                        │                     │
              ▼                        ▼                     ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   Hardware & OS Abstraction Layer (HAL)                │
├──────────────────────────────────────┬─────────────────────────────────┤
│        Windows 10 / 11 HAL           │    Linux & SteamOS (Steam Deck) │
│  • Win32 IP Helper API (Metrics)     │  • Netlink / rtnetlink (Routes) │
│  • Win32 IcmpSendEcho2 Probes        │  • Raw ICMP (cap_net_raw) / UDP │
│  • Windows Filtering Platform (WFP)  │  • nftables Transient Table     │
│  • Toolhelp32 Process Watcher        │  • /proc & cgroup v2 Watcher    │
└──────────────────────────────────────┴─────────────────────────────────┘
```

---

## 🚀 Key Capabilities

- **⚡ Real-Time Telemetry & Probing:** Low-overhead asynchronous ICMP and UDP socket probes calculate rolling Round-Trip Time (RTT), standard-deviation jitter, and packet loss percentages across all active adapters.
- **🧠 Multi-Candidate Route Optimizer:** Evaluates candidate adapters against mathematical scoring profiles:
  $$\text{Score} = w_l \cdot \text{Latency} + w_j \cdot \text{Jitter} + w_p \cdot \text{Loss} + w_t \cdot \text{Throughput}$$
  Supports user-selectable optimization profiles:
  - `Lowest Latency` — Heavily penalizes high RTT (ideal for twitch FPS titles).
  - `Balanced Gaming` — Equal weighting across ping stability and jitter.
  - `Maximum Stability` — Strict zero-packet-loss preference (prioritizes wired Ethernet).
- **🔒 Gaming Lockdown (Traffic Isolation):** Restricts non-game outbound bandwidth during active matches. Automatically whitelists verified voice chat services (Discord, TeamSpeak) and game launchers (Steam, Epic Games, Riot Client) while blocking telemetry and background updates.
- **⏱️ Failsafe Watchdog Lease:** Every active firewall lockdown operates on a rolling kernel heartbeat lease. If the client crashes, power is lost, or the user quits, the lease expires and all network traffic is restored immediately.
- **🎮 Game-Aware Detection:** Process watcher identifies active game executables and automatically switches profiles upon launch and game exit.

---

## 💻 Cross-Platform Support Matrix

| Subsystem | Windows (10 / 11) | Linux & SteamOS (Steam Deck) |
| :--- | :--- | :--- |
| **Presentation Tier** | Tauri v2 (Edge WebView2) | Tauri v2 (WebKitGTK) |
| **Telemetry & Probing** | Native Win32 Sockets & `IcmpSendEcho2` | Raw ICMP (`cap_net_raw`) & UDP echo |
| **Route Metric Switching** | `Set-NetIPInterface` / Win32 IP Helper API | `rtnetlink` / `ip route replace ... metric` |
| **Gaming Lockdown** | Windows Filtering Platform (WFP) dynamic session | `nftables` transient table (`table inet nexus_net`) |
| **Process Monitoring** | Win32 Toolhelp32 snapshot / ETW | `/proc` virtual filesystem & `cgroup v2` |
| **Adapter Tuning** | Registry Driver properties via PowerShell | `iw`, `ethtool`, `wpa_supplicant` |

---

## 🛡️ Anti-Cheat & Security Invariants

Nexus Net is engineered to be **100% compliant with kernel-level anti-cheat platforms** (*Riot Vanguard*, *Easy Anti-Cheat*, *BattlEye*, *Valve Anti-Cheat*):
- **Zero DLL Injection:** Never hooks game processes, modifies memory, or tampers with binary code.
- **Standard OS Networking:** Route metrics and firewall isolation use official, documented operating system APIs.
- **Local-First & Private:** Probing metrics and application lists remain strictly on your local machine; zero external telemetry is collected or transmitted.

---

## 🛠️ Quick Start

### 1. Interactive UI Prototype

Test the live Route Optimizer interface in any modern browser:

```bash
# Open reference single-page application prototype
xdg-open NexusNet_RouteOptimizer.html   # On Linux
start NexusNet_RouteOptimizer.html      # On Windows
```

### 2. Building from Source

#### Prerequisites
- **Rust 1.80+** (`rustup default stable`)
- **Node.js 18+** & `npm` / `pnpm`
- Platform Build Essentials:
  - **Linux:** `sudo apt install libwebkit2gtk-4.1-dev build-essential curl wget libssl-dev libgtk-3-dev libayatana-appindicator3-dev librsvg2-dev`
  - **Windows:** Microsoft C++ Build Tools & Windows SDK

#### Compiling Tauri Desktop Application
```bash
# Clone the repository
git clone https://github.com/vijaygovindBiju/NEXUS.git
cd NEXUS

# Install frontend dependencies
npm install

# Run application in development mode
cargo tauri dev

# Build production executable / installer
cargo tauri build
```

---

## 📂 Project Structure

```text
nexius/
├── ARCHITECTURE.md                    # Core HAL traits, data flows, and subsystem specifications
├── PROTOCOL.md                        # Tauri IPC schemas, events, probe specs, and algorithms
├── TODO.md                            # Development backlog and delivery milestones
├── SECURITY.md                        # Anti-cheat compliance, watchdog safety, and privileges
├── NexusNet_RouteOptimizer.html       # Standalone interactive UI prototype
├── src-ui/                            # Frontend assets and views
└── src-tauri/                         # Rust backend
    ├── Cargo.toml
    └── src/
        ├── main.rs                    # Entrypoint & IPC handlers
        ├── engine/                    # Telemetry probes, route scoring, watchdog
        └── platform/                  # OS-specific drivers (Windows WFP & Linux nftables)
```

---

## 📚 Technical Specifications

- 🏛️ **[ARCHITECTURE.md](ARCHITECTURE.md)** — Hardware abstraction traits, data pipelines, and design invariants.
- 🔌 **[PROTOCOL.md](PROTOCOL.md)** — IPC command schemas, event payloads, and scoring algorithms.
- 🛡️ **[SECURITY.md](SECURITY.md)** — Anti-cheat compliance, permission requirements, and watchdog safety.
- 📋 **[TODO.md](TODO.md)** — Phased development tasks and delivery roadmap.

---

## 📄 License

Nexus Net is open-source software licensed under the **[MIT License](LICENSE)**.
