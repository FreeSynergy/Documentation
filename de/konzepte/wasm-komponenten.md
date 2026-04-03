# WASM-Komponenten (fs-packages)

[← Zurück zum Index](../INDEX.md)

---

## Überblick

FreeSynergy UI-Komponenten können als **WASM-Plugins** ausgeliefert werden.  
Das ermöglicht:

- **Dynamisches Laden** zur Laufzeit (kein Neustart)
- **Vorab-Kompilierung** für Compile-Time-Sicherheit
- **Sandbox-Isolation** per WASI-Capabilities (O7)
- **Store-Distribution** als `artifact`-Paket

---

## Architektur

```
Store (artifact: wasm-component)
     │
     ▼
fs-plugin-runtime
  PluginRuntime::load_file(path, sandbox)
     │
     ├── wasmtime   — WASM-Ausführung
     ├── WASI P1    — sandboxed I/O
     └── PluginHandle::execute(&ctx) → PluginResponse
```

---

## Design Pattern: Plugin

```rust
// fs-plugin-sdk: WASM-Seite
pub trait PluginImpl {
    fn handle(&self, ctx: &PluginContext) -> PluginResponse;
}

// fs-plugin-runtime: Host-Seite
pub struct PluginRuntime { engine: Engine }
impl PluginRuntime {
    pub fn load_file(&self, path: &Path, sandbox: PluginSandbox) -> Result<PluginHandle>;
}
```

---

## PluginSandbox (WASI Capabilities)

```rust
PluginSandbox::minimal()          // keine Rechte
    .allow_read("/some/path")     // nur lesen
    .allow_write("/some/path")    // lesen + schreiben
    .with_env("KEY", "value")     // einzelne Env-Variable
```

Für UI-Komponenten gilt: **minimaler Sandbox** (O7):

```rust
PluginSandbox::minimal()
    .allow_read(paths.config.join("components"))
```

Netzwerk- und DB-Zugriff: **nie direkt** — immer über FS-Services via gRPC.

---

## WASM-ABI

Jedes WASM-Modul exportiert:

| Export | Typ | Beschreibung |
|--------|-----|--------------|
| `plugin_manifest_json()` | `fn() -> i32` | Zeiger auf JSON-Manifest im WASM-Memory |
| `plugin_free(ptr, len)` | `fn(i32, i32)` | gibt den Manifest-Buffer frei |
| `handle(...)` | via JSON-RPC | Verarbeitet Kommandos |

Das Manifest (JSON):

```json
{
  "id": "inventory-list",
  "version": "0.1.0",
  "description": "Shows installed applications",
  "commands": ["render", "on_event"]
}
```

---

## Komponenten-Deklaration in `package.toml`

```toml
[[component]]
id      = "inventory-list"
slot    = "fill"
wasm    = "components/inventory_list.wasm"
config  = "components/inventory_list.toml"

[component.bounds]
min_width  = 160
max_width  = 0    # 0 = unbegrenzt
min_height = 0
max_height = 0
```

---

## Lebenszyklus einer WASM-Komponente

```
Store install
  └─ package.toml gelesen
  └─ .wasm + .toml kopiert nach ~/.local/share/freesynergy/{paket}/components/

Desktop-Start
  └─ HotReloadWatcher überwacht components/
  └─ ComponentRegistry::register(WASM-Wrapper)

Render-Frame
  └─ LayoutInterpreter::interpret(descriptor)
  └─ registry.get("inventory-list")
  └─ component.render(ctx) → Vec<LayoutElement>
  └─ Engine interpretiert LayoutElement-Baum

Hot-Reload
  └─ inotify: .wasm geändert
  └─ HotReloadEvent::ComponentChanged(path)
  └─ PluginRuntime::load_file(new_wasm, sandbox)
  └─ registry.register(new_handle)  [ersetzt alten]
```

---

## Lifecycle-Hooks (PluginTrait)

```rust
pub trait PluginInstall: Send + Sync {
    fn on_install(&self, event: &LifecycleEvent) -> Result<PluginResponse>;
}
pub trait PluginRemove: Send + Sync {
    fn on_remove(&self, event: &LifecycleEvent) -> Result<PluginResponse>;
}
pub trait PluginUpgrade: Send + Sync {
    fn on_upgrade(&self, event: &LifecycleEvent) -> Result<PluginResponse>;
}
// Convenience-Supertrait:
pub trait PluginLifecycle: PluginInstall + PluginRemove + PluginUpgrade {}
```

---

## Sandbox-Grenzen (O7)

| Aktion | Erlaubt | Verboten |
|--------|---------|---------|
| Daten lesen | gRPC → FS-Services | Direkter DB-Zugriff |
| Daten schreiben | Bus-Events emittieren | Direkter Filesystem-Zugriff |
| UI rendern | Eigene Slot-Area | Andere Slots |
| Netzwerk | Über FS-Services | Direkter Netzwerkzugriff |
| Umgebung | Explizit gewährte ENV-Vars | Ambient Environment |

---

## Store-Distribution

WASM-Komponenten werden als `artifact`-Paket im Store verteilt:

```toml
# Store/packages/inventory-list/package.toml
[package]
id      = "inventory-list"
type    = "artifact"
version = "0.1.0"

[[artifact]]
kind = "wasm-component"
file = "inventory_list.wasm"
```

Der Store installiert die `.wasm`-Datei in das Komponenten-Verzeichnis.  
Der `HotReloadWatcher` erkennt die neue Datei und registriert die Komponente.

---

## Repos

| Repo | Inhalt |
|------|--------|
| `fs-packages/fs-plugin-sdk` | WASM-Plugin-SDK (für Plugin-Autoren) |
| `fs-packages/fs-plugin-runtime` | Host-Runtime (wasmtime + WASI) |
| `fs-render` | `ComponentTrait` + `ComponentRegistry` |
| `fs-render` | `HotReloadWatcher` (inotify) |
