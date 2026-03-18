# Ressourcen-System

[← Zurück zum Index](../INDEX.md) | [Store](../programme/store/README.md) | [Pakete](pakete.md)

---

## Grundregel: Alles ist eine Ressource

Alles was der Store anbietet ist eine **Ressource**: Apps, Container-Apps, Widgets, Icons, Schriftarten, Farbschemata — alles. Ein Bundle ist eine Ressource die auf andere Ressourcen verweist.

---

## Gemeinsame Pflichtfelder (JEDE Ressource)

```rust
pub struct ResourceMeta {
    pub id: String,                  // "kanidm", "midnight-blue-colors"
    pub name: String,                // "Kanidm"
    pub description: String,         // Kurzbeschreibung
    pub version: Version,            // SemVer
    pub author: String,
    pub license: String,
    pub icon: PathBuf,               // SVG, PFLICHT
    pub tags: Vec<String>,
    pub resource_type: ResourceType,
    pub dependencies: Vec<Dependency>,
    pub signature: Signature,
    pub status: ValidationStatus,    // ✅ OK / ⚠️ Unvollständig / ❌ Kaputt
}
```

---

## Ressource-Typen

```rust
pub enum ResourceType {
    // Programme
    App,              // Node, Desktop, Container App Manager
    ContainerApp,     // Kanidm, Forgejo, Outline (bringt YAML mit)
    
    // Zusammenstellungen
    Bundle,           // Sammlung anderer Ressourcen
    
    // Funktions-Ressourcen
    Widget,
    Bot,
    Bridge,
    Task,
    Language,
    
    // Theme-Ressourcen (einzeln ladbar)
    ColorScheme,      // Farbpalette
    Style,            // Spacing, Radius, Shadows (standardisiert!)
    FontSet,          // Schriftarten-Dateien
    CursorSet,        // Mauszeiger-Dateien
    IconSet,          // Icon-Sammlung (SVGs)
    ButtonStyle,      // Button-Aussehen
    WindowChrome,     // Titelleisten-Stil
    AnimationSet,     // Transitions, Keyframes
}
```

---

## App

```rust
pub struct AppResource {
    pub meta: ResourceMeta,
    pub platforms: Vec<Platform>,         // linux-x86_64, macos-aarch64, ...
    pub binary_name: String,              // "fsn-node"
    pub locales: Vec<String>,             // ["en", "de"] — eigene Sprachdateien
    pub config_schema: ConfigSchema,
    pub cli_commands: Vec<CliCommand>,
    pub api_endpoints: Vec<ApiEndpoint>,
    pub roles_provided: Vec<Role>,
    pub roles_required: Vec<RoleDep>,
}
```

Jede App hat ihre eigenen Sprachdateien, ihr Icon, ihre Dependencies.

---

## Container-App

```rust
pub struct ContainerAppResource {
    pub meta: ResourceMeta,
    pub compose_yaml: String,             // Originale YAML (downloadbar)
    pub services: Vec<ContainerService>,  // Haupt + Subservices
    pub roles_provided: Vec<Role>,
    pub roles_required: Vec<RoleDep>,
    pub apis: Vec<RoleApiRef>,            // Welche Rollen-APIs via Bridge
    pub variables: Vec<ContainerVariable>,
    pub networks: Vec<NetworkDef>,
    pub volumes: Vec<VolumeDef>,
}

pub struct ContainerService {
    pub name: String,
    pub image: String,
    pub is_main: bool,
    pub internal: bool,
    pub port: Option<u16>,
    pub healthcheck: Option<HealthCheck>,
    pub version_tag: String,
}

pub struct ContainerVariable {
    pub name: String,                     // "WEBUI_SECRET_KEY"
    pub var_type: VarType,                // Secret, Url, Hostname, Port, ...
    pub role: Option<Role>,               // z.B. iam.oidc-discovery
    pub required: bool,
    pub default: Option<String>,
    pub auto_from: Option<AutoSource>,    // Von anderem Service befüllen
    pub description: String,
    pub confidence: f32,                  // Bei Auto-Erkennung
}

pub enum AutoSource {
    InternalService(String),              // Von Subservice: "http://ollama:11434"
    RoleVariable(Role, String),           // Vom Inventory: Service mit Rolle X
}
```

Container-Apps haben **keine eigenen Übersetzungen** — das machen die Leute selbst.

Die YAML wird mitgeliefert und ist separat downloadbar. Der Container App Manager macht daraus Quadlet-Dateien.

---

## Widget

```rust
pub struct WidgetResource {
    pub meta: ResourceMeta,
    pub required_roles: Vec<RoleRequirement>,
    pub data_sources: Vec<DataSource>,
    pub min_size: (u32, u32),
    pub max_size: (u32, u32),
    pub default_size: (u32, u32),
    pub refresh_interval: Option<Duration>,
    pub permissions: Vec<Permission>,
}

pub struct RoleRequirement {
    pub roles: Vec<Role>,         // ["chat", "mail", "smtp"]
    pub mode: RequireMode,        // ANY oder ALL
}
```

**Installationsregel:** Wenn `required_roles` nicht erfüllt sind (kein passender Service im Inventory), kann das Widget NICHT installiert werden. Fehlermeldung: "Installiere zuerst einen Service mit Rolle X". Im Bundle MIT einem passenden Service: geht.

---

## Bot

```rust
pub struct BotResource {
    pub meta: ResourceMeta,
    pub channels: Vec<ChannelType>,       // Telegram, Matrix, Discord
    pub commands: Vec<BotCommand>,        // /broadcast, /verify
    pub required_roles: Vec<RoleRequirement>,
    pub triggers: Vec<BusTrigger>,        // Welche Bus-Events
    pub permissions: Vec<Permission>,
    pub tokens_required: Vec<TokenDef>,   // Welche API-Tokens nötig
}
```

---

## Bridge

```rust
pub struct BridgeResource {
    pub meta: ResourceMeta,
    pub target_role: Role,                // Welche Rolle diese Bridge bedient
    pub target_service: String,           // "kanidm", "outline", "forgejo"
    pub methods: Vec<BridgeMethod>,
}

pub struct BridgeMethod {
    pub standard_name: String,            // "user.create" (Rollen-API)
    pub http_method: HttpMethod,
    pub endpoint: String,                 // "/v1/person"
    pub request_mapping: FieldMapping,
    pub response_mapping: FieldMapping,
}
```

Bridges mappen die standardisierte Rollen-API auf die echte Service-API. Siehe [Bridges](bridges.md).

---

## Theme-Ressourcen

### ColorScheme

Farbpalette. Standardisierter Aufbau — alle Pflicht-Variablen MÜSSEN vorhanden sein:

```toml
[colors]
bg-base = "#0c1222"
bg-surface = "#141c30"
bg-elevated = "#1a2440"
bg-card = "#141c30"
bg-input = "#0f1729"
text-primary = "#e8edf5"
text-secondary = "#a0aec0"
text-muted = "#6b7a99"
primary = "#4d8bf5"
primary-hover = "#6ba0ff"
primary-text = "#ffffff"
accent = "#f59e0b"
success = "#10b981"
warning = "#f59e0b"
error = "#ef4444"
border = "#2a3654"
border-focus = "#4d8bf5"
```

### Style (STANDARDISIERT)

Spacing, Radius, Shadows. Striktes Schema — alle Felder definiert:

```toml
[style]
# Radius
radius-sm = "4px"
radius = "8px"
radius-lg = "16px"

# Spacing
spacing-xs = "4px"
spacing-sm = "8px"
spacing-md = "16px"
spacing-lg = "24px"
spacing-xl = "32px"

# Shadows
shadow-sm = "0 1px 2px rgba(0,0,0,0.1)"
shadow = "0 2px 4px rgba(0,0,0,0.1)"
shadow-lg = "0 4px 12px rgba(0,0,0,0.15)"
shadow-glow = "0 0 20px rgba(77,139,245,0.15)"

# Borders
border-width = "1px"

# Scrollbar
scrollbar-width = "6px"

# Sidebar
sidebar-width-collapsed = "48px"
sidebar-width-expanded = "220px"

# Transitions
transition-fast = "150ms ease"
transition = "200ms ease"
transition-slow = "300ms ease"
```

### FontSet, CursorSet, IconSet, ButtonStyle, WindowChrome, AnimationSet

Jeweils eigene Ressource mit eigenem standardisierten Schema. Einzeln installierbar, einzeln austauschbar.

---

## Bundle (Theme-Bundle als Beispiel)

Ein Theme-Bundle fasst maximal EINE Ressource pro Typ zusammen:

```toml
[package]
id = "midnight-blue"
name = "Midnight Blue"
type = "bundle"
tags = ["dark", "blue", "kde", "professional"]

[bundle.resources]
color_scheme = "midnight-blue-colors"
style = "midnight-blue-style"
font_set = "inter-font"
icon_set = "lucide-icons"
cursor_set = "default-cursor"
button_style = "rounded-buttons"
window_chrome = "kde-standard"
animation_set = "smooth-animations"
```

Jede referenzierte Ressource muss im Store existieren.

---

## Validierungsstatus

Jede Ressource im Store hat einen sichtbaren Status:

| Status | Bedeutung |
|---|---|
| ✅ Komplett | Alle Pflichtfelder da, Signatur gültig, Abhängigkeiten erfüllt |
| ⚠️ Unvollständig | Fehlen Tags, Beschreibung zu kurz, optionale Felder leer |
| ❌ Kaputt | Pflichtfelder fehlen, Signatur ungültig, Abhängigkeiten nicht erfüllbar |

Der Status ist dauerhaft sichtbar — nicht nur beim Upload.

---

Weiter: [Bridges](bridges.md) | [Store](../programme/store/README.md) | [Inventory](inventory.md)
