# Nexus Net — Development Roadmap & Actionable TODOs

---

## 🎯 Progress Overview

- [x] UI Prototype Review & Architectural Feasibility Analysis
- [x] System Architecture & Cross-Platform HAL Specifications
- [x] IPC & Telemetry Protocol Definitions
- [ ] Phase 1: Desktop Shell & Frontend Integration
- [ ] Phase 2: Live Cross-Platform Probing Engine
- [ ] Phase 3: Route Optimizer & Metric Controller
- [ ] Phase 4: Gaming Lockdown & Watchdog Firewall
- [ ] Phase 5: Game Process Watcher & Profiles
- [ ] Phase 6: Adapter Center & Diagnostics Lab
- [ ] Phase 7: Cross-Platform Packaging & Release

---

## Phase 1: Desktop Shell & Frontend Integration
*Goal: Package the existing polished HTML/CSS UI inside a lightweight, cross-platform Tauri v2 application.*

- [ ] **1.1. Tauri v2 Scaffolding:** Initialize Tauri v2 workspace with Rust 2021 edition. `[HIGH]`
- [ ] **1.2. Frontend Asset Migration:** Move `NexusNet_RouteOptimizer.html` into `src-ui/index.html` and extract inline styles/scripts into modular files. `[HIGH]`
- [ ] **1.3. IPC Bridge Setup:** Replace in-memory mock variables in JS with calls to `window.__TAURI__.core.invoke`. `[HIGH]`
- [ ] **1.4. Frameless Window & System Tray:** Configure a modern frameless dark-mode window with titlebar drag support and minimize-to-tray capability. `[MED]`

---

## Phase 2: Live Cross-Platform Probing Engine
*Goal: Replace simulated mock numbers with real network interface telemetry.*

- [ ] **2.1. Adapter Discovery:**
  - [ ] Linux: Enumerate network interfaces using `rtnetlink` / `/sys/class/net/`. `[HIGH]`
  - [ ] Windows: Enumerate interfaces via Win32 `GetAdaptersAddresses`. `[HIGH]`
- [ ] **2.2. Asynchronous Probe Engine:**
  - [ ] Implement non-privileged UDP echo/DNS probes to `1.1.1.1` and `8.8.8.8` using `tokio::net::UdpSocket`. `[HIGH]`
  - [ ] Optional: Raw ICMP socket fallback when elevated permissions are available. `[LOW]`
- [ ] **2.3. Telemetry Mathematics:** Compute rolling sample buffer (size 30) for RTT, standard deviation jitter, and packet loss percentage. `[HIGH]`
- [ ] **2.4. Telemetry Stream Event:** Wire live telemetry updates to Tauri event `telemetry://tick` to render the SVG live performance graph. `[HIGH]`

---

## Phase 3: Route Optimizer & Metric Controller
*Goal: Dynamically route gaming traffic through the best physical interface.*

- [ ] **3.1. Scoring Engine Implementation:** Implement normalization algorithms for *Lowest Latency*, *Balanced Gaming*, *Maximum Throughput*, and *Most Stable*. `[HIGH]`
- [ ] **3.2. Explainable Recommendation Generator:** Port natural-language rationale generator from prototype to Rust backend. `[MED]`
- [ ] **3.3. Dynamic Metric Adjustment:**
  - [ ] Linux: Manipulate default route metrics using Netlink (`RTM_NEWROUTE` / `RTM_DELROUTE`). `[HIGH]`
  - [ ] Windows: Adjust interface metrics using Win32 `SetIpForwardEntry2` or `Set-NetIPInterface`. `[HIGH]`
- [ ] **3.4. Auto-Routing Daemon:** Optional background task that automatically switches default route when a candidate's score exceeds the active interface by a threshold ($>15\%$) for over 5 seconds. `[MED]`

---

## Phase 4: Gaming Lockdown & Watchdog Firewall
*Goal: Provide strict traffic isolation while gaming to eliminate background network congestion.*

- [ ] **4.1. Fail-Safe Watchdog Timer:**
  - [ ] Implement lease-based watchdog thread that automatically flushes firewall rules if heartbeat expires. `[CRITICAL]`
  - [ ] Implement Emergency Unlock IPC command. `[CRITICAL]`
- [ ] **4.2. Linux Firewall (`nftables`):**
  - [ ] Create transient `nexus_lockdown` table. `[HIGH]`
  - [ ] Apply egress filter allowing loopback, DNS, and whitelisted socket UIDs/cgroups. `[HIGH]`
- [ ] **4.3. Windows Firewall (WFP):**
  - [ ] Open dynamic WFP session (`FWPM_SESSION_FLAG_DYNAMIC`). `[HIGH]`
  - [ ] Add outbound ALE connection filter matching allowlisted executables. `[HIGH]`
- [ ] **4.4. UI Allowlist Synchronization:** Wire frontend allowlist checkboxes to backend firewall configuration. `[HIGH]`

---

## Phase 5: Game Process Watcher & Profiles
*Goal: Automatically detect running games and apply configured profiles.*

- [ ] **5.1. Process Scanning Engine:**
  - [ ] Linux: Parse `/proc/[pid]/comm` and `/proc/[pid]/cmdline`. `[MED]`
  - [ ] Windows: Poll `CreateToolhelp32Snapshot` / ETW process start notifications. `[MED]`
- [ ] **5.2. Known Games Database:** Maintain JSON database of popular game executables (*VALORANT*, *Counter-Strike 2*, *Apex Legends*, *Overwatch 2*, *Fortnite*). `[MED]`
- [ ] **5.3. Profile Import & Export:** Implement JSON schema validation for saving and loading community profiles. `[LOW]`

---

## Phase 6: Adapter Center & Diagnostics Lab
*Goal: Provide hardware parameter tuning and a 4-stage network diagnostic pipeline.*

- [ ] **6.1. Adapter Parameter Querying:**
  - [ ] Windows: Read registry driver properties via WMI / PowerShell. `[MED]`
  - [ ] Linux: Query power-saving and wireless modes using `iw` and `ethtool`. `[MED]`
- [ ] **6.2. Safe Defaults & Rollback:** Prevent applying unsupported driver parameters. `[HIGH]`
- [ ] **6.3. Diagnostics Pipeline:**
  - [ ] Stage 1: Local gateway ping (link-layer check). `[MED]`
  - [ ] Stage 2: Public DNS ping (ISP stability check). `[MED]`
  - [ ] Stage 3: Hop-by-hop traceroute (MTR bottleneck locator). `[MED]`
  - [ ] Stage 4: Throughput sample test chunk. `[MED]`

---

## Phase 7: Cross-Platform Packaging & Release
*Goal: Build clean, digitally verifiable distributables.*

- [ ] **7.1. Linux Builds:** CI/CD for `.deb`, `.rpm`, and `.AppImage` with setcap permissions for non-root probing. `[MED]`
- [ ] **7.2. Windows Builds:** MSI installer and lightweight NSIS setup with UAC prompt for helper service. `[MED]`
- [ ] **7.3. Code Signing & Release Verification:** Ensure binaries are signed to avoid Windows SmartScreen and anti-cheat false positives. `[HIGH]`
