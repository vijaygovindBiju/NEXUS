# Nexus Net ⚡

> **Cross-Platform Gaming Network Optimizer & Traffic Isolation Engine**  
> *Lower ping. Eliminate jitter. Keep distractions out.*

---

## 📌 Overview

**Nexus Net** is a lightweight, cross-platform desktop application designed to optimize network paths and isolate gaming traffic for competitive and latency-sensitive games. Unlike cloud-routed commercial tunneling solutions that require recurring subscriptions, Nexus Net acts directly on the local operating system's network stack—measuring live link telemetry, dynamically prioritizing optimal physical network adapters, and enforcing strict application traffic isolation during gaming sessions.

---

## 🚀 Key Capabilities

* **Real-Time Telemetry & Probing:** Low-overhead asynchronous ICMP and UDP probes calculate rolling latency (RTT), jitter (standard deviation), and packet loss across all active physical interfaces.
* **Explainable Route Optimizer:** Multi-candidate scoring engine that dynamically ranks interfaces (e.g., Ethernet vs 5 GHz Wi-Fi vs 5G USB Tethering) based on customizable objectives: *Lowest Latency*, *Balanced Gaming*, *Maximum Throughput*, or *Maximum Stability*.
* **Gaming Lockdown (Traffic Isolation):** Restricts outbound network traffic strictly to verified game executables, launchers, and voice chat apps during play, preventing background downloads, Windows Update, and cloud backups from inducing bufferbloat.
* **Fail-Safe Watchdog:** Kernel and session-safe lease timer and emergency unlock guarantees that network access is never permanently blocked even if the game or application crashes.
* **Game-Aware Profiles & Detection:** Native process watcher that automatically detects running games (e.g., *VALORANT*, *Counter-Strike 2*, *Apex Legends*, *Fortnite*) and loads corresponding network profiles.
* **Adapter Center:** Hardware-level parameter inspect and tuning (power saving, transmit power, band preference) tailored to driver capabilities.
* **Diagnostics Lab:** 4-stage health check analyzing local link health, gateway jitter, intermediate ISP hop reachability (traceroute), and throughput.

---

## 💻 Cross-Platform Support Matrix

Nexus Net is engineered to run seamlessly across both **Windows** (10 / 11) and **Linux** (including SteamOS on Steam Deck).

| Feature | Windows (10/11) | Linux (Desktop & SteamOS) |
| :--- | :--- | :--- |
| **GUI Framework** | Tauri v2 (WebKit/Edge WebView2) | Tauri v2 (WebKitGTK) |
| **Telemetry & Probing** | Native Win32 Sockets & `IcmpSendEcho2` | Raw ICMP (`cap_net_raw`) & UDP echo |
| **Route Metric Switching** | `Set-NetIPInterface` / Win32 IP Helper API | `rtnetlink` / `ip route replace ... metric` |
| **Gaming Lockdown** | Windows Filtering Platform (WFP) dynamic session | `nftables` transient table (`table inet nexus_net`) |
| **Process Monitoring** | Win32 Toolhelp32 snapshot / ETW | `/proc` virtual filesystem & `cgroup v2` |
| **Adapter Tuning** | Registry Driver properties via PowerShell | `iw`, `ethtool`, `wpa_supplicant` |

---

## 🛡️ Anti-Cheat & Security Policy

Nexus Net is designed from the ground up to be **100% compliant with kernel-level anti-cheat systems** (*Riot Vanguard*, *Easy Anti-Cheat*, *BattlEye*, *Valve Anti-Cheat*):
* **No DLL Injection:** Nexus Net never injects code into or modifies the memory space of game processes.
* **OS-Level Network Routing:** All traffic isolation and metric adjustments are performed through sanctioned, native operating system firewall and routing APIs.
* **Local-First & Zero Telemetry:** Network measurements and game history never leave the user's machine.

---

## 📂 Project Structure

```
nexius/
├── README.md                          # Project overview, capabilities, and quickstart
├── ARCHITECTURE.md                    # System architecture, HAL traits, and components
├── PROTOCOL.md                        # IPC schemas, events, probe specs, and algorithms
├── TODO.md                            # Comprehensive development roadmap & milestones
├── SECURITY.md                        # Anti-cheat compliance, watchdog safety, and privileges
├── NexusNet_RouteOptimizer.html       # Reference UI prototype (HTML5/CSS/JS SPA)
├── src-ui/                            # Application frontend assets
│   ├── index.html
│   ├── css/
│   └── js/
└── src-tauri/                         # Rust backend & OS abstraction layers
    ├── Cargo.toml
    └── src/
        ├── main.rs                    # Tauri entrypoint & IPC router
        ├── engine/                    # Probing, route scoring, watchdog
        └── platform/                  # OS-specific drivers (Windows & Linux)
```

---

## 🛠️ Tech Stack & Requirements

* **Frontend:** HTML5, Modern CSS (Design Tokens, SVG Canvas), Vanilla ES6+ JavaScript.
* **Core Application Backend:** [Rust](https://www.rust-lang.org/) (1.80+), [Tauri v2](https://v2.tauri.app/).
* **Async Runtime:** `tokio` with asynchronous networking (`pnet`, `socket2`).
* **Packaging:** 
  * Linux: `.deb`, `.rpm`, `.AppImage`.
  * Windows: `.msi`, NSIS standalone executable.

---

## 📖 Documentation Index

For in-depth technical specifications, please consult the dedicated documentation:
1. [ARCHITECTURE.md](ARCHITECTURE.md) — System design, HAL abstractions, and runtime flow.
2. [PROTOCOL.md](PROTOCOL.md) — Tauri IPC commands, event payloads, and scoring algorithms.
3. [TODO.md](TODO.md) — Phased task backlog and implementation checklist.
4. [SECURITY.md](SECURITY.md) — Anti-cheat compliance, privilege model, and watchdog fail-safes.
