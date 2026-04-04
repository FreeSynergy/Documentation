# Manager Service Controller

[← Zurück zu Konzepten](../INDEX.md)

---

## Zweck

Ein Manager ist nicht nur ein Setup-Wizard — er ist der **Service Controller** seiner Kategorie. Er steuert alle Dienste desselben Typs (aktive + installierbare), kennt den primären Dienst und kann zwischen Implementierungen wechseln.

---

## Design Pattern

| Pattern | Verwendung |
|---|---|
| **Command** | `ServiceCommand` (Start / Stop / Restart / Enable / Disable) |
| **Strategy** | `ServiceController` — systemd oder Container |
| **Composite** | `CategoryManager` — verwaltet alle Dienste einer Kategorie |

---

## Crate

**`fs-manager-core`** — in `fs-managers/core/`

Alle Manager-Crates hängen von `fs-manager-core` ab.

---

## ServiceController

```rust
trait ServiceController: Send + Sync {
    fn name(&self) -> &'static str;
    async fn start(&self)   -> Result<(), ManagerCoreError>;
    async fn stop(&self)    -> Result<(), ManagerCoreError>;
    async fn restart(&self) -> Result<(), ManagerCoreError>;
    async fn enable(&self)  -> Result<(), ManagerCoreError>;
    async fn disable(&self) -> Result<(), ManagerCoreError>;
    async fn status(&self)  -> Result<ServiceStatus, ManagerCoreError>;
    async fn execute(&self, cmd: ServiceCommand) -> Result<(), ManagerCoreError>;
}
```

### Implementierungen

| Typ | Beschreibung |
|---|---|
| `SystemdServiceController` | Wraps `systemctl` via `tokio::process::Command` |
| `ContainerServiceController` | Wraps `podman pod` direkt (ohne systemd); enable/disable delegiert an SystemdServiceController |

---

## CategoryManager

```rust
trait CategoryManager: Send + Sync {
    fn category(&self) -> ServiceCategory;
    async fn list_all(&self)             -> Result<Vec<ServiceInfo>, _>;
    async fn list_running(&self)         -> Result<Vec<ServiceInfo>, _>;
    async fn get_active(&self)           -> Result<Option<ServiceInfo>, _>;
    async fn set_active(&self, id: &str) -> Result<(), _>;
    // Default: Ok(None) — Manager mit Store-Anbindung überschreiben
    async fn update_available(&self, id: &str) -> Result<Option<String>, _>;
}
```

### ServiceInfo

```rust
struct ServiceInfo {
    id: String,
    display_name: String,
    installed: bool,
    is_primary: bool,
    status: ServiceStatus,
    version: Option<String>,
}
```

---

## ServiceStatus

| Variante | Bedeutung |
|---|---|
| `Running` | Läuft normal |
| `Stopped` | Gestoppt |
| `Failed` | Fehler, keine automatische Wiederherstellung |
| `Starting` | Startet |
| `Stopping` | Stoppt |
| `Unknown` | Status unbekannt (nicht installiert) |

---

## Role-Switching

Wenn zwei Dienste dieselbe `ServiceCategory` implementieren (z. B. Kanidm + Keycloak für IAM):

1. Admin wählt neuen Primary via Manager-UI
2. `CategoryManager::set_active(id)` aufrufen
3. `fs-registry` Capability-Eintrag auf neuen Dienst umschreiben
4. Alle abhängigen Dienste werden automatisch informiert (via fs-bus)

---

## Manager-UI Tab "Services"

Jeder Manager zeigt einen Tab **Dienste** mit:

- Liste aller installierten Dienste der Kategorie
- Status-Badge (Running / Stopped / Failed)
- Buttons: Start / Stop / Restart / Config (→ AppConfigurator) / Pod (→ PodConfigurator)
- "Als primär setzen"-Button (wenn mehrere installiert)
- Update-Hinweis wenn neue Version im Store verfügbar

---

## Implementierungen (Phase 5)

| Manager | Controller | Kategorie |
|---|---|---|
| `fs-manager-auth` | `KanidmIamController` | `ServiceCategory::Iam` |
| `fs-manager-mail` | `StalwartMailController` | `ServiceCategory::Mail` |
| `fs-manager-matrix` | `TuwunelMessengerController` | `ServiceCategory::Messenger` |
| `fs-manager-zentinel` | `ZentinelProxyController` | `ServiceCategory::Proxy` |

---

## i18n

FTL-Keys in `fs-i18n/locales/{lang}/manager-service.ftl`:

- `manager-service-status-*` (running, stopped, failed, …)
- `manager-service-cmd-*` (start, stop, restart, …)
- `manager-category-*` (iam, mail, proxy, …)
- `manager-service-tab-*`, `manager-service-set-primary`
