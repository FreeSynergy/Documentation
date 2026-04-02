# fs-store

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-store` · `/home/kal/Server/fs-store/`
**Typ:** Program (Workspace: 3 Crates)
**Capabilities:** `store.catalog`, `store.search`, `store.install`, `store.update`

---

## Was ist das?

`fs-store` ist der Paketmanager von FreeSynergy.
Er liest den Katalog aus `FreeSynergy/Store` (Git-Repo), sucht Pakete,
installiert, deinstalliert und aktualisiert sie.
Erkennt automatisch ob ein Display vorhanden ist — GUI wenn ja, CLI sonst.

---

## Workspace-Struktur

```
fs-store/crates/
├── fs-store      — Core-Library: Katalog, Pipeline, Inventory, Package-Traits
├── fs-store-app  — GUI-App (Dioxus, Migration zu iced geplant per G2)
└── fs-store-cli  — CLI-Binary (fs-store)
```

---

## Design

```
Design Patterns:
  Pipeline  — Install-Pipeline (InstallStep-Trait)
  MVC       — Store-UI (StoreController, StoreModel, StoreView via FsView-Trait)

Objekte:
  Package (Trait)       — Basis-Interface für alle Paket-Typen
  AppPackage            — Native Binaries (gRPC/REST, Capabilites, Storage)
  ContainerPackage      — Container-Dienste (Ports, Variablen, Features)
  BundlePackage         — Meta-Pakete (Component-Liste + Render-Engine-Wahl)
  ThemePackage          — Visual Themes
  LanguagePackage       — Sprach-Packs
  IconSetPackage        — Icon-Sets
  WidgetPackage         — Desktop Widgets
  TaskPackage           — Automation Templates
  ExternalPackage       — Externe Services (nur Metadaten)
  RepoPackage           — Zusätzliche Store-Repositories

  Inventory             — Zentraler Hub: alle Pakete + Install-Status
  StoreReader           — Liest TOML-Kataloge (Local oder HTTP)
  InstallRecord         — Eine installierte Paket-Version

Install-Pipeline (Pipeline Pattern):
  InstallStep (Trait)   — Eine Phase der Installation
  Pipeline              — Geordnete Liste von Steps
  InstallContext        — Mutable Context (req, target, fs_dir, urls)
  InstallTarget         — Container | Rpm | Deb | Flatpak | AppImage
  PipelineEvent         — Progress-Events für UI/CLI

  Schritte:
    DownloadStep        — Icon + Artifact aus Store holen
    ValidateStep        — Checksum/Signatur prüfen (Placeholder bis Phase 3)
    InstallFileStep     — Datei schreiben + System-Installer aufrufen
    AdapterInstallStep  — Adapter automatisch mit installieren
    InventoryStep       — fs-inventory via REST benachrichtigen (best-effort)
    RegistryStep        — fs-registry via REST benachrichtigen (best-effort)
    PublishEventStep    — install::completed auf Bus publishen (best-effort)
```

---

## Install-Pipeline (Phase 2)

Jede Installation läuft durch eine geordnete Pipeline:

```
Download → Validate → Install → AdapterInstall → Inventory → Registry → Event
```

- **Fehler** in einem Step → Pipeline abborten, `PipelineEvent::Failed`
- **Skip** = Step nicht applicable (z.B. Bundle hat kein File) → weitermachen
- **Best-effort** Steps (Inventory, Registry, Bus) → nie aborting

Adapter-Auto-Install: wenn ein `program`/`container`-Paket installiert wird,
installiert der `AdapterInstallStep` automatisch den deklarierten Adapter.
Der Adapter registriert seine Capability in `fs-registry`.

---

## Store-Katalog (`FreeSynergy/Store`)

Der Katalog ist ein separates Git-Repo (`/home/kal/Server/Store/`).

```
Store/
├── catalog.toml                    — Namespace-Index
├── packages/
│   ├── apps/                       — Native Binaries (FreeSynergy-Programme)
│   ├── containers/                 — Container-Dienste (Fork + 3rd-party)
│   ├── adapters/                   — Adapter-Pakete (Trait-Impl)
│   ├── bundles/                    — Meta-Pakete
│   ├── themes/                     — Visual Themes
│   ├── i18n/                       — Sprach-Packs
│   ├── icons/                      — Icon-Sets
│   ├── widgets/                    — Desktop Widgets
│   ├── tasks/                      — Automation Templates
│   └── external/                   — Externe Services
└── init/                           — Bootstrap (fs-init, kein Package)
```

### package.toml-Format (Phase 2 — vollständig)

```toml
[package]
id      = "forgejo"
name    = "Forgejo"
type    = "container"         # app | container | adapter | bundle | artifact | fork
version = "9.0.0"
summary = "…"
icon    = "icon.svg"
tags    = ["git", "oidc"]

[provides]
capabilities = ["git-hosting", "git.api"]

[requires]
capabilities = ["iam.oidc-provider"]

[adapter]                     # Auto-Install: Adapter wird mitinstalliert
id   = "fs-auth"
repo = "FreeSynergy/fs-auth"

[storage]                     # Wo das Paket Daten ablegt
global = "/var/lib/freesynergy/forgejo"
config = "/etc/freesynergy/forgejo"
cache  = "/var/cache/freesynergy/forgejo"

[install_targets]             # Welche Install-Targets verfügbar sind
container = true
rpm       = false

[[api.grpc]]                  # gRPC-API (für Doku-Tab)
service     = "FreeSynergy.Node"
port        = 50051
description = "gRPC API"

[[api.rest]]                  # REST-API (für Doku-Tab)
base        = "/api/v1"
port        = 3000
proto       = "http"
description = "REST API"
spec        = "/api/swagger"

[contract]                    # Routen für Zentinel-Proxy
upstream_tls = false

[[contract.routes]]
id          = "main"
path        = "/"
port        = 3000
```

### Bundles (Phase 2)

| Bundle       | Inhalt                                               |
|--------------|------------------------------------------------------|
| `minimal`    | node + registry + sqlite                             |
| `server`     | Minimal + auth + inventory + session + zentinel(opt) |
| `workstation`| Server + Desktop + render-engine(User-Wahl) + Apps  |
| `developer`  | Workstation + Forgejo + erweiterte Tools             |
| `zentinel`   | Zentinel + Zentinel Control Plane                    |

**Render-Engine-Wahl im Workstation-Bundle:**
Der Installer zeigt dem User ein Select-Dropdown mit den verfügbaren Engines.
`fs-info` erkennt ob ein Display-Server verfügbar ist und schlägt die passende
Engine vor (iced bei Wayland/X11, TUI sonst).

---

## CLI

```bash
# Pakete auflisten
fs-store list
fs-store list --namespace containers
fs-store list --search forgejo

# Paket-Details
fs-store info forgejo

# Installierte Pakete
fs-store installed

# Paket installieren
fs-store install forgejo
fs-store install forgejo --target container
fs-store install kanidm --env ADMIN_PASSWORD=secret

# Paket entfernen
fs-store remove forgejo

# Update-Check
fs-store update              # alle prüfen
fs-store update forgejo      # einzelnes Paket

# Optionale Service-URLs
fs-store install forgejo \
  --inventory-url http://localhost:8082 \
  --registry-url  http://localhost:8081 \
  --bus-url       http://localhost:8090
```

---

## i18n

FTL-Datei: `fs-i18n/locales/en/store.ftl`
Keys-Datei: `fs-store-app/src/i18n/keys.rs`

Alle user-facing Texte — UI, CLI, Pipeline-Progress — sind als FTL-Keys definiert.
Kein roher String im Code.

---

## Standalone-Test (Phase 2.4)

```bash
# Ohne fs-desktop (CLI reicht)
fs-store list --local /home/kal/Server/Store

# Ohne laufenden fs-node
FS_STORE_LOCAL=/home/kal/Server/Store fs-store list
```
