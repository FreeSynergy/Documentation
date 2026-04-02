# fs-info

**System-Information für FreeSynergy**

Erkennt und meldet CPU-Auslastung, Speicher, Disk-Nutzung, Netzwerk, Thermal-Sensoren und
System-Uptime. Wird von Desktop-Widgets, Store, Managers und Alerting verwendet.

**Repository:** `FreeSynergy/fs-info` — Library-Crate (kein eigener Prozess)

---

## Architektur

```
SystemInfo (Trait)       MetricsCollector (Trait)
     │                          │
     └──────── FsInfo ──────────┘
               (Facade)

FsInfo delegates to:
  CpuInfo      — CPU-Auslastung, Load Average, Kern-Anzahl
  MemInfo      — RAM + Swap-Nutzung
  DiskInfo     — Partitionsliste mit genutztem/freiem Speicher
  NetworkInfo  — Netzwerk-Interface-Statistiken
  ThermalInfo  — CPU-Temperatursensoren
  Uptime       — System-Uptime
  OsInfo       — Betriebssystem-Informationen

Statische Daten (24h Cache):
  SysInfoCache     — ~/.config/fsn/sysinfo.toml
  DetectedFeatures — systemd, Podman, Docker, Git, SSH, smartctl,
                     WaylandDisplay, X11Display, Terminal
```

### Traits

#### `SystemInfo`

Interface für alle System-Info-Abfragen. Consumer-Code sollte nur gegen diesen Trait programmieren.

| Methode | Rückgabe | Beschreibung |
|---|---|---|
| `cpu()` | `CpuInfo` | CPU-Auslastung + Load Average |
| `memory()` | `MemInfo` | RAM + Swap |
| `disk()` | `DiskInfo` | Partition-Nutzung |
| `network()` | `NetworkInfo` | Interface-Statistiken |
| `uptime()` | `Uptime` | System-Uptime |
| `os()` | `OsInfo` | OS-Typ, Version, Hostname |
| `thermal()` | `ThermalInfo` | CPU-Temperatur |

#### `MetricsCollector`

Liefert eine flache Liste aller Metriken als `Vec<Metric>`.

```rust
pub struct Metric {
    pub name:  String,   // z. B. "memory.used_percent"
    pub value: f64,
    pub unit:  String,   // z. B. "%", "bytes", "°C", "s"
}
```

---

## API-Verwendung

```rust
use fs_info::{FsInfo, SystemInfo, MetricsCollector};

let info = FsInfo::new();

// Einzelne Subsysteme
let mem = info.memory();
println!("RAM: {:.1}% used", mem.used_percent());

let disk = info.disk();
for part in &disk.partitions {
    println!("{}: {:.1}%", part.mount_point, part.used_percent());
}

let up = info.uptime();
println!("Uptime: {}", up.display()); // z. B. "3d 4h 12m"

// Alle Metriken auf einmal
let metrics = info.collect();
for m in &metrics {
    println!("{} = {} {}", m.name, m.value, m.unit);
}
```

---

## Alerting

```rust
use fs_info::{AlertChecker, AlertThresholds};

let checker = AlertChecker::new(AlertThresholds {
    disk_full_percent:   90.0,
    cpu_hot_celsius:     85.0,
    memory_full_percent: 90.0,
});

let alerts = checker.check_once();
for alert in &alerts {
    println!("[{}] {}", alert.bus_topic(), alert.description());
}
```

Bus-Topics: `sysinfo.alert.disk`, `sysinfo.alert.memory`, `sysinfo.alert.cpu`

---

## Statische Daten + Cache

```rust
use fs_info::SysInfoCache;

let cache = SysInfoCache::default_path(); // ~/.config/fsn/sysinfo.toml
let (os_info, features) = cache.get_or_detect(); // aus Cache oder frisch ermitteln

println!("OS: {} {}", os_info.os_type.label(), os_info.version);
println!("systemd:  {}", features.has(fs_info::Feature::Systemd));
println!("wayland:  {}", features.has(fs_info::Feature::WaylandDisplay));
println!("x11:      {}", features.has(fs_info::Feature::X11Display));
println!("terminal: {}", features.has(fs_info::Feature::Terminal));

// Convenience-Methoden
println!("display:  {}", features.has_display_server());
println!("tty:      {}", features.has_terminal());
```

---

## Optionales Feature: SMART

```toml
fs-info = { path = "../fs-info", features = ["smart"] }
```

Liest SMART-Disk-Gesundheit via `smartctl`. Ergibt `SmartInfo` + Alert `sysinfo.alert.smart`.

---

## Abgrenzung

| | fs-info | fs-session | fs-inventory |
|---|---|---|---|
| Frage | Wie ist der Hardware-Zustand? | Wer ist eingeloggt? | Was ist installiert? |
| Datenbasis | OS-APIs (sysinfo) | SQLite | SQLite |
| Lebensdauer | On-demand | Login–Logout | Persistent |

---

[← Index](../INDEX.md) | [Konzept: SysInfo](../konzepte/sysinfo.md)
