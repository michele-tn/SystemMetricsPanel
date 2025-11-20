
SystemMetricsPanel
======================

An overview of a lightweight Bash-based system monitor
----------------------------------------------------------------

### Abstract

`SystemMetricsPanel` is a self-contained Bash module for low-overhead observation of host-level performance indicators over Bash shell.  
Originally implemented as a single-line “monitoring bar,” the system has evolved into a **multi-line framed dashboard**, providing a clearer and more expressive representation of system state while maintaining minimal resource consumption.

It exposes two complementary execution modes:

1. **Realtime mode**, which refreshes an in-place textual panel at a fixed cadence, and  
2. **Snapshot mode**, which emits a single multi-line report suitable for logging or embedding into other tools.

The script relies exclusively on standard GNU/Linux interfaces such as `/proc`, `/sys`, and `df`, ensuring maximum portability and zero external dependencies.

---

## 1. Design goals

The script is designed around the following objectives:

- **Portability** – Runs on any common Linux distribution using only POSIX tools.  
- **Low overhead** – Sampling incurs negligible system load.  
- **SSH compatibility** – Designed for terminal-based remote sessions.  
- **Self-configuration** – Automatically detects a suitable network interface.  
- **Clear separation of logic** – Data gathering and presentation layers remain distinct.

---

## 2. Functional overview

Each refresh cycle performs the following steps:

1. **Determine the network interface**
2. **Sample system metrics**  
   - CPU load  
   - Memory usage  
   - Network throughput  
   - Disk utilization  
   - Host metadata  
3. **Normalize and compute meaningful statistics**
4. **Render the panel**
   - Multi-line Unicode frame  
   - Colors via ANSI sequences  
   - CPU mini-bar (text-based)  
5. **Mode behavior**
   - Realtime: refresh in-place  
   - Snapshot: print once and exit  

---

## 3. Metric computation details

### 3.1 CPU utilization (delta method)

Two `/proc/stat` samples are taken one second apart:

- Compute total and idle times  
- `CPU% = (1 - Δidle / Δtotal) * 100`  
- Rendered as:  
  ```
  CPU  : 23% [####------]
  ```

### 3.2 Memory usage

Derived from `/proc/meminfo`:

- `MemTotal - MemAvailable`  
- Converted to GiB  
- Example:  
  ```
  MEM  : 3.12 GB / 7.80 GB
  ```

### 3.3 Network throughput

Reads `/sys/class/net/<iface>/statistics/*_bytes`:

- Computes deltas over the interval  
- Converts bytes → Mbps  
- Example:  
  ```
  NET  : ↑2.44 Mb/s | ↓0.97 Mb/s
  ```

### 3.4 Disk utilization

Uses `df -hP`, extracts `%` for `/` and `/boot/efi`:

```
DISK : /: 42%, /boot/efi: 12%
```

---

## 4. Execution modes

### 4.1 Snapshot mode

Single printed panel → exit.

### 4.2 Realtime mode

Updates continuously:

- Recomputes metrics  
- Moves cursor up  
- Redraws entire panel  
- Quit with `q`  

---

## 5. Usage

```bash
chmod +x SystemMetricsPanel.sh

# Default (snapshot)
./SystemMetricsPanel.sh

# Realtime (if script prompts for mode)
./SystemMetricsPanel.sh

# Specify interface
./SystemMetricsPanel.sh eth0
```

---

## 6. Representative output

### 6.1 Realtime example

```text
┌────────────────────────────────────────────────────────────────────────────┐
│                          SYSTEM METRICS PANEL (LIVE)                       │
├────────────────────────────────────────────────────────────────────────────┤
│HOST : srv-node01                                                           │
│USER : admin                                                                │
│UP   : 152 h                                                                │
│CPU  : 17% [###-------]                                                     │
│MEM  : 2.91 GB / 7.80 GB                                                    │
│NET  : ↑3.12 Mb/s | ↓1.02 Mb/s                                              │
│DISK : /: 42%, /boot/efi: 12%                                               │
│Press q to quit                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Snapshot example

```text
┌────────────────────────────────────────────────────────────────────────────┐
│                           SYSTEM METRICS SNAPSHOT                          │
├────────────────────────────────────────────────────────────────────────────┤
│HOST       : srv-node01                                                     │
│USER       : admin                                                          │
│UPTIME     : 152 h                                                          │
├────────────────────────────────────────────────────────────────────────────┤
│CPU LOAD   : 23% [####------]                                               │
│MEMORY     : 3.12 GB / 7.80 GB                                              │
│NETWORK    : ↑2.44 Mb/s | ↓0.97 Mb/s                                        │
│DISK       : /: 42%, /boot/efi: 12%                                         │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Extensibility

Possible future enhancements:

- Per-core CPU view  
- Historical plot storage  
- Temperature sensors  
- JSON/CSV export  

---

## 8. Licensing and disclaimer

Provided “as-is,” without warranty.  
You may freely adapt or extend the script.

