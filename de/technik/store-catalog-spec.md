# Store Catalog Specification

[← Zurück zum Index](../INDEX.md)

---

## Überblick

Der FreeSynergy Store verwendet ein dateibasiertes Katalog-System im Git-Repository `FreeSynergy/Store`.
Jedes Paket liegt in einem eigenen Verzeichnis mit einer `catalog.toml`-Datei.

---

## Verzeichnisstruktur

```
Store/
  packages/
    adapters/           ← Trait-Implementierungen (DbEngine, RenderEngine, LLM, Channel)
      catalog.toml      ← Namespace-Index
      fs-db-engine-sqlite/
      fs-db-engine-postgres/
      fs-gui-engine-iced/
      fs-gui-engine-tui/
      fs-gui-engine-bevy/
      fs-llm-mistral/
      fs-llm-openai/
      fs-channel-matrix/
      fs-channel-telegram/
    apps/               ← Native Rust-Binaries
      catalog.toml
      init/
      auth/
      node/
      desktop/
      store/
      ...
    containers/         ← Containerisierte Dienste (Podman Kube)
      catalog.toml
      kanidm/
      stalwart/
      tuwunel/
      zentinel/
      ...
    bundles/            ← Gruppen von Paketen (Minimal, Server, Workstation, Developer)
    themes/             ← Theme-Artifacts
    i18n/               ← Sprachpaket-Artifacts
    icons/              ← Icon-Theme-Artifacts
    widgets/            ← WASM-Komponenten
    tasks/              ← Aufgaben-Definitionen
    repos/              ← Git-Repository-Referenzen
    external/           ← Externe Programme (nicht-FreeSynergy)
```

---

## Package-Typen

| Typ | Beschreibung | Beispiele |
|---|---|---|
| `app` | Natives Rust-Binary (cross-platform) | fs-init, fs-node, fs-desktop, fs-store |
| `adapter` | Implementiert Trait, registriert Capability | fs-db-engine-sqlite, fs-gui-engine-iced |
| `container` | Containerisierter Dienst (Podman Kube) | Kanidm, Stalwart, Tuwunel, Zentinel |
| `bundle` | Gruppe von Paketen | Minimal, Server, Workstation, Developer |
| `artifact` | Reine Daten (Sprachen, Themes, WASM) | en, de, fs-theme-cyan, fs-widget-clock |
| `fork` | 3rd-party Fork mit FreeSynergy-Patches | kanidm, stalwart, tuwunel, zentinel |
| `widget` | WASM-Komponente für den Desktop | fs-widget-clock, fs-widget-weather |
| `task` | Automatisierbare Aufgaben-Definition | backup-db, update-all |
| `repo` | Git-Repository-Referenz (Forgejo) | forgejo-hosted repos |

---

## catalog.toml — Pflichtfelder (alle Typen)

```toml
[package]
id          = "eindeutiger-bezeichner"      # Kebab-case, ohne Namespace-Prefix
name        = "Anzeigename"
type        = "app | adapter | container | ..."
version     = "0.1.0"                       # SemVer
summary     = "Kurzbeschreibung (1 Satz)"
description = "Langbeschreibung (Markdown)"
description_file = "i18n/en/description.ftl"  # Optional: Datei statt inline
icon        = "icon.svg"
tags        = ["tag1", "tag2"]
screenshots = [                             # Optional, aber empfohlen
    "https://raw.githubusercontent.com/FreeSynergy/Store/main/screenshots/pkg-1.png",
]
author      = "FreeSynergy"
license     = "MIT"

[package.origin]
git       = "https://github.com/FreeSynergy/repo"
publisher = "FreeSynergy"
```

---

## catalog.toml — Typ-spezifische Felder

### `app` — Native Binary

```toml
[source]
type     = "github_release"
repo     = "FreeSynergy/fs-example"
artifact = "fs-example-x86_64-unknown-linux-gnu.tar.gz"

[distribution]
linux-x86_64   = "https://github.com/FreeSynergy/fs-example/releases/download/v{version}/..."
linux-aarch64  = "..."
macos-x86_64   = "..."
macos-aarch64  = "..."
windows-x86_64 = "..."

[app]
data_dir   = "{data_root}/example"
config_dir = "{config_root}/example"

[[binaries]]
name   = "fs-example"
binary = "fs-example"

[provides]
capabilities = ["example.capability"]
bus_messages = ["example.event"]

[requires]
capabilities = []
```

### `adapter` — Trait-Implementierung

```toml
[provides]
capabilities = ["trait.name.impl"]

[requires]
capabilities = []

[install_targets]
artifact  = true
container = false

[sources]
repo = "https://github.com/FreeSynergy/fs-example-adapter"
```

### `container` — Podman Kube

```toml
[provides]
capabilities = ["service.name"]

[requires]
capabilities = []

[storage]
config = "/etc/freesynergy/example"
cache  = "/var/cache/freesynergy/example"

[install_targets]
container = true

[[api.grpc]]
service     = "FreeSynergy.Example"
proto       = "example.proto"
port        = 50060
description = "gRPC service"

[[api.rest]]
base        = "/api/v1/example"
port        = 8060
proto       = "http"
description = "REST API"
spec        = "/swagger-ui"

[contract]
# Service-level agreements / interface contracts
```

### `fork` — 3rd-party Fork

```toml
[package.fork]
source     = "fork"
fork_of    = "upstream/repo"
fork_repo  = "FreeSynergy/forked-repo"
fork_url   = "https://github.com/FreeSynergy/forked-repo"
```

---

## Namespace-Kataloge (catalog.toml im Verzeichnis)

Namespace-Verzeichnisse (adapters/, apps/, containers/) enthalten eine eigene
`catalog.toml` mit einem `[catalog]`-Block und `[[packages]]`-Einträgen:

```toml
[catalog]
id      = "adapters"
name    = "Adapters"
icon    = "icon.svg"
summary = "Trait implementations"

[[packages]]
id      = "fs-db-engine-sqlite"
catalog = "fs-db-engine-sqlite/catalog.toml"
```

---

## Capabilities — Namenskonvention

```
Kategorie.subkategorie.impl
Beispiele:
  db.engine.sqlite        ← DbEngine-Trait, SQLite-Impl
  db.engine.postgres      ← DbEngine-Trait, PostgreSQL-Impl
  render.engine.iced      ← RenderEngine-Trait, iced-Impl
  render.engine.tui       ← RenderEngine-Trait, ratatui-Impl
  render.engine.3d        ← RenderEngine-Trait, 3D-fähig (Bevy)
  auth.oauth2             ← OAuthProvider-Trait
  auth.sso                ← SsoProvider-Trait
  channel.matrix          ← Channel-Trait, Matrix-Impl
  channel.telegram        ← Channel-Trait, Telegram-Impl
  llm.mistral             ← LlmAdapter-Trait, Mistral-Impl
  proxy.ingress           ← ReverseProxy, Ingress-Funktion
  proxy.tls               ← TLS-Termination
```

---

## Screenshots

Screenshots liegen im Store-Repository unter `screenshots/`:
```
Store/
  screenshots/
    kanidm-1.png
    kanidm-2.png
    stalwart-1.png
    desktop-1.png
    desktop-2.png
    ...
```

Konvention: `{package-id}-{nummer}.png` (kebab-case, 1-basiert).
Empfohlene Größe: 1280×800px, max. 500KB.

---

## Verwandte Dokumentation

- [konzepte/pakete.md](../konzepte/pakete.md)
- [konzepte/capabilities.md](../konzepte/capabilities.md)
- [architektur/repositories.md](../architektur/repositories.md)
