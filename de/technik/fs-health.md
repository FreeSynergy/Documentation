# fs-health

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-health` ist das generische Health-Check-Framework für FreeSynergy.
Es stellt Traits und Typen bereit, um jeden Service oder jede Ressource
selbst-beschreibend hinsichtlich ihres Gesundheitszustands zu machen.

`fs-health` ist ein reines Library-Crate ohne Container oder Daemon.
Alle Services implementieren den `HealthCheck`-Trait — Code programmiert
immer gegen das Interface, nie gegen konkrete Typen.

---

## Architektur

```
HealthCheck           ← Trait: health() → HealthStatus

HealthStatus          ← Aggregiertes Ergebnis (overall + issues)
HealthLevel           ← Ok | Warning | Error  (Ord, vergleichbar)
HealthIssue           ← Einzelner Fund (level + i18n-Key)

HealthRules           ← Fluent Builder: require() / warn() → HealthStatus
HealthMonitor         ← Observer: beobachtet N HealthCheck-Implementors
LevelMeta             ← Trait: Präsentations-Metadaten (indicator, i18n_key)
```

---

## `HealthCheck`-Trait

```rust
pub trait HealthCheck {
    fn health(&self) -> HealthStatus;
}
```

Jede Ressource (Service, Konfiguration, Host, …) implementiert diesen Trait.
Object-safe: `Box<dyn HealthCheck>` funktioniert.

```rust
use fs_health::{HealthCheck, HealthRules, HealthStatus};

struct MyService { host: Option<String> }

impl HealthCheck for MyService {
    fn health(&self) -> HealthStatus {
        HealthRules::new()
            .require(self.host.is_some(), "health-service-no-host")
            .build()
    }
}
```

---

## `HealthStatus`

Aggregiertes Ergebnis eines Health-Checks.

| Feld | Typ | Beschreibung |
|---|---|---|
| `overall` | `HealthLevel` | Schlechtester Level aller Issues |
| `issues` | `Vec<HealthIssue>` | Alle gefundenen Issues |

```rust
let mut status = HealthStatus::ok();
status.error("health-service-down");
status.warning("health-no-monitoring");
assert_eq!(status.overall, HealthLevel::Error);
```

Zwei Status lassen sich zusammenführen:
```rust
let mut a = HealthStatus::ok();
let b = HealthStatus::ok();
a.merge(b);
```

Filtern nach Level:
```rust
for issue in status.issues_at_level(HealthLevel::Error) {
    println!("{}", issue.msg_key);
}
```

---

## `HealthLevel`

```
Ok      → "✓ (ok)"      — alles in Ordnung
Warning → "⚠ (warning)" — eingeschränkt funktionsfähig
Error   → "✗ (error)"   — Deployment/Betrieb nicht möglich
```

`HealthLevel` implementiert `Ord` — Error > Warning > Ok.

```rust
assert!(HealthLevel::Error > HealthLevel::Warning);
assert!(HealthLevel::Ok.is_ok());
assert_eq!(HealthLevel::Warning.i18n_key(), "health-warning");
```

---

## `HealthRules` (Fluent Builder)

Builder-Pattern für deklarative Health-Checks:

```rust
let status = HealthRules::new()
    .require(host.is_some(), "health-missing-host")        // Error wenn false
    .warn(monitoring.is_some(), "health-no-monitoring")    // Warning wenn false
    .build();
```

---

## `HealthMonitor` (Observer Pattern)

Beobachtet eine Menge von `HealthCheck`-Implementors und aggregiert
ihre Ergebnisse.

```rust
let mut monitor = HealthMonitor::new();
monitor.register("database", db_service);
monitor.register("auth",     auth_service);

// Gesamtzustand (schlimmster Level aller Subjects)
let overall: HealthStatus = monitor.overall();

// Einzelergebnisse
let results: HashMap<&str, HealthStatus> = monitor.run_all();
```

---

## FTL-Keys

Datei: `fs-i18n/locales/{lang}/health.ftl`

| Key | Deutsch | Englisch |
|---|---|---|
| `health-ok` | Gesund | Healthy |
| `health-warning` | Eingeschränkt | Degraded |
| `health-error` | Nicht verfügbar | Unhealthy |
| `health-status-label` | Gesundheitsstatus | Health status |
| `health-overall-label` | Gesamtzustand | Overall health |
| `health-missing-host` | Kein Host konfiguriert. | No host configured. |
| `health-service-down` | Der Dienst läuft nicht. | Service is not running. |

---

## Design Patterns

| Pattern | Wo |
|---|---|
| **Strategy** | `HealthCheck`-Trait — jede Ressource hat eigene Prüflogik |
| **Observer** | `HealthMonitor` beobachtet N `HealthCheck`-Subjects |
| **Builder** | `HealthRules` — fluent API für deklarative Regeln |
| **OOP** | `HealthLevel` trägt eigene Präsentations-Metadaten via `LevelMeta` |

---

## Repo

- Lokal: `/home/kal/Server/fs-libs/fs-health/`
- GitHub: `git@github.com:FreeSynergy/fs-libs.git` (Workspace-Member)
