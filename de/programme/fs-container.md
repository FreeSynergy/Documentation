# fs-container

**Container-Lifecycle-Management für FreeSynergy**

Verwaltet Podman-Container via **Quadlet** (systemd Unit Files) — ohne Daemon-Socket, ohne
Bollard. Alle Operationen gehen über Datei-Generierung und `systemctl --user`.

**Repository:** `FreeSynergy/fs-container` — Library-Crate (kein eigener Prozess)

---

## Architektur

```
ContainerEngine (Trait)
    │
    └── QuadletManager  — Quadlet-Dateien + systemctl --user

ServiceConfig  — deklarative Container-Beschreibung (Builder Pattern)
HealthCheck    — konfigurierbarer Gesundheitscheck
Volume         — Host ↔ Container Volume-Mount
PortBinding    — Host ↔ Container Port-Mapping
SystemctlManager — async-Wrapper für systemctl
```

### `ContainerEngine` Trait

| Methode | Beschreibung |
|---|---|
| `deploy(config)` | Quadlet schreiben + daemon-reload + start |
| `start(name)` | Service starten |
| `stop(name)` | Service stoppen |
| `restart(name)` | Service neustarten |
| `remove(name)` | Service stoppen + Quadlet-Datei löschen |
| `status(name)` | Laufzeitstatus abfragen |
| `logs(name, lines)` | Letzte N Log-Zeilen via journalctl |
| `list()` | Alle verwalteten Services + Status |

---

## Warum Quadlet statt Bollard?

| | Quadlet (diese Impl.) | Bollard / Socket |
|---|---|---|
| Daemon-Socket | Nicht benötigt | Pflicht |
| Rootless | Ja (systemctl --user) | Ja (Podman) |
| Rollback | Backup-Datei + restart | Manuell |
| Deklarativ | Ja (.container Unit-Datei) | Nein |
| Systemd-Integration | Voll (Restart, Journal) | Keine |

---

## API-Verwendung

```rust
use fs_container::{ContainerEngine, QuadletManager, ServiceConfig, Volume, HealthCheck, PortBinding};

let mgr = QuadletManager::user_default();

// Service definieren
let svc = ServiceConfig::new("kanidm", "ghcr.io/kanidm/kanidm:1.5.0")
    .with_description("Identity management")
    .with_volume(Volume::bind("/data/kanidm", "/data"))
    .with_healthcheck(HealthCheck::new(["curl", "-fs", "https://localhost/status"]))
    .with_env("LOG_LEVEL", "info");

// Deployen: Quadlet-Datei schreiben + daemon-reload + start
mgr.deploy(&svc).await?;

// Status abfragen
let status = mgr.status("kanidm").await?;
println!("{}: {}", status.name, status.active_state);

// Logs abrufen
let logs = mgr.logs("kanidm", 50).await?;

// Alle Services auflisten
let all = mgr.list().await?;

// Entfernen
mgr.remove("kanidm").await?;
```

---

## Rollback

Bei einem fehlerhaften Deploy kann die vorherige Konfiguration wiederhergestellt werden:

```rust
// Vor dem Deploy wird automatisch ein Backup angelegt.
// Bei Problemen:
mgr.rollback("kanidm").await?;
```

---

## Quadlet-Datei (Beispiel)

```ini
[Unit]
Description=Identity management
After=network-online.target
Wants=network-online.target

[Container]
Image=ghcr.io/kanidm/kanidm:1.5.0
ContainerName=fs-kanidm
Network=fsn
Label=managed-by=freeSynergy
Environment=LOG_LEVEL=info
Volume=/data/kanidm:/data
HealthCmd=curl -fs https://localhost/status
HealthInterval=30s
HealthTimeout=10s
HealthRetries=3
HealthStartPeriod=5s

[Service]
Restart=always
TimeoutStartSec=300

[Install]
WantedBy=default.target
```

---

[← Index](../INDEX.md)
