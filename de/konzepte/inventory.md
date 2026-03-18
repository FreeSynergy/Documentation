# Inventory — Die lokale Wahrheit

[← Zurück zum Index](../INDEX.md) | [Store](../programme/store/README.md) | [Bridges](bridges.md)

---

## Store vs. Inventory

| | Store | Inventory |
|---|---|---|
| Frage | "Was gibt es?" | "Was habe ich?" |
| Datenquelle | Git-Repo, andere Stores, URLs | Nur lokale SQLite |
| Netzwerk | Ja (Git, HTTP) | Nein (nur lokal) |
| Inhalt | Katalog aller verfügbaren Ressourcen | Nur installierte Ressourcen + Status |
| Verantwortung | Validierung, Signatur-Check, Download | Lokaler Zustand, APIs, Bridges |

Der Store redet mit der Außenwelt. Das Inventory redet nur mit dem lokalen System.

---

## Inventory-Schema

```rust
pub struct Inventory {
    pub db: SqlitePool,  // fsn-inventory.db (EIGENE DB)
}
```

### Installierte Ressource (JEDER Typ)

```rust
pub struct InstalledResource {
    pub id: String,                   // "kanidm"
    pub resource_type: ResourceType,  // ContainerApp, Widget, Theme, ...
    pub version: String,              // "1.5.0"
    pub channel: ReleaseChannel,      // Stable, Testing, Nightly
    pub installed_at: DateTime,
    pub status: ResourceStatus,       // Active, Stopped, Error, Updating
    pub config_path: PathBuf,
    pub data_path: PathBuf,
    pub validation: ValidationStatus, // ✅ ⚠️ ❌
}

pub enum ResourceStatus {
    Active,
    Stopped,
    Error(String),
    Updating,
    Installing,
}
```

### Service-Instanz (für Container-Apps)

```rust
pub struct ServiceInstance {
    pub resource_id: String,          // → InstalledResource
    pub instance_name: String,        // Vom Benutzer vergeben
    pub roles_provided: Vec<Role>,
    pub roles_required: Vec<Role>,
    pub bridges: Vec<BridgeRef>,      // Welche Bridges aktiv
    pub variables: Vec<ConfiguredVar>,
    pub network: String,
    pub status: ServiceStatus,        // Running, Stopped, Error, Starting
    pub port: Option<u16>,
    pub s3_paths: Vec<String>,
}
```

### Bridge-Instanz

```rust
pub struct BridgeInstance {
    pub bridge_id: String,            // "kanidm-iam-bridge"
    pub role: Role,                   // "iam"
    pub service_instance: String,     // "main-iam"
    pub api_base_url: String,         // "http://kanidm:8443"
    pub status: BridgeStatus,
}
```

---

## Wer fragt das Inventory?

| Wer | Fragt was |
|---|---|
| **Bus** | "Wer hat Rolle X?" → Route Events zur richtigen Bridge |
| **Lenses** | "Welche Services bieten Daten zum Thema Y?" |
| **Search** | "Welche Services unterstützen search-Recht?" |
| **Container App Manager** | "Welche Services laufen, welche sind gestoppt?" |
| **Widgets** | "Sind meine required_roles erfüllt?" |
| **Store** | "Was ist installiert? Was braucht ein Update?" |
| **Resource Builder** | "Welche Rollen sind lokal verfügbar für Tests?" |

---

## Das Inventory ist die EINZIGE Wahrheit

Wenn das Inventory sagt "Kanidm läuft auf Port 8443 mit Rolle iam" — dann ist das so. Kein anderes System darf eine eigene Liste führen. Der Bus fragt immer das Inventory, nie direkt die Services.

---

Weiter: [Store](../programme/store/README.md) | [Bridges](bridges.md) | [Bus](bus.md)
