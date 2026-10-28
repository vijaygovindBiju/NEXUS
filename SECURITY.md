# Security, Safety & Anti-Cheat Policy — Nexus Net

---

## 🛡️ 1. Anti-Cheat Compliance

Competitive multiplayer titles enforce strict anti-cheat protections (e.g., **Riot Vanguard**, **Easy Anti-Cheat (EAC)**, **BattlEye**, **Valve Anti-Cheat (VAC)**). These systems inspect process memory, driver hooks, and system calls. 

Nexus Net is engineered to ensure **zero anti-cheat interference**:

### Strict Non-Interference Guarantees:
1. **No Memory Injection or Hooking:** Nexus Net **never** opens a handle with `PROCESS_VM_WRITE` or `PROCESS_VM_READ` to game executables, and **never** injects DLLs (unlike older tools such as *ForceBindIP*).
2. **OS-Level Boundary Enforcement:** All traffic isolation and route preferences are executed through sanctioned operating system network APIs:
   * On Windows: Standard Windows Filtering Platform (WFP) and IP Helper (`IPHLPAPI`).
   * On Linux: Standard Linux Netlink and `nftables`.
3. **External Process Observation Only:** Game detection relies strictly on unprivileged process enumeration (`Toolhelp32` on Windows, `/proc` on Linux) which is completely benign and safe.

---

## 🔒 2. Principle of Least Privilege & Architecture Separation

To minimize the attack surface and protect user system integrity, Nexus Net separates responsibilities across two boundaries:

```
┌───────────────────────────────────────┐
│     Unprivileged Frontend & Core      │
│  • Runs as standard user              │
│  • Manages UI, charts, and telemetry  │
│  • Calculates route scores            │
└───────────────────┬───────────────────┘
                    │ Local Named Pipe / UNIX Socket (IPC)
┌───────────────────▼───────────────────┐
│       Privileged System Helper        │
│  • Runs with administrative / root    │
│  • Strictly scoped: only adjusts      │
│    firewall rules & route metrics     │
│  • Enforces fail-safe watchdog        │
└───────────────────────────────────────┘
```

* **No arbitrary shell execution:** The privileged helper only accepts structured, validated commands (e.g., `AddAllowedApp(path)`, `SetMetric(iface, metric)`). It never executes arbitrary shell commands.
* **Path Validation:** All executable paths in the allowlist are canonicalized and verified before being passed to firewall filters.

---

## 🦺 3. Fail-Safe Watchdog & Anti-Lockout Guarantees

Because **Gaming Lockdown** drops unapproved outbound network traffic, an unexpected application crash could leave a user without network connectivity. Nexus Net prevents this through multiple redundant safeguards:

1. **Ephemeral Kernel Sessions:**
   * **Windows:** WFP rules are created inside dynamic sessions (`FWPM_SESSION_FLAG_DYNAMIC`). If the Nexus Net process terminates (whether cleanly, via crash, or via Task Manager kill), the Windows OS kernel flushes all active filters automatically.
   * **Linux:** `nftables` rules are bound to an ephemeral table that is flushed on termination.
2. **Lease Watchdog Timer:**
   * A safety lease (default: 15 minutes) is established upon session start.
   * If the user fails to renew or if the main UI thread freezes, the watchdog thread trips the emergency release and restores full connectivity.
3. **Emergency Unlock Action:**
   * The UI provides a one-click **Emergency Unlock** button that unconditionally flushes all rules and normalizes interface metrics.

---

## 🕵️ 4. Privacy & Telemetry Standards

Nexus Net is **local-first**:
* **Zero Telemetry Collection:** Network readings, latency figures, IP addresses, and game history are kept in memory and local storage on your machine.
* **No Packet Payload Inspection:** Nexus Net does not decrypt or inspect packet payloads (DPI). It operates purely at the transport and interface layer (IP, port, and process ownership).
* **No Remote Account Required:** All optimization logic runs natively without requiring external cloud accounts or logins.
