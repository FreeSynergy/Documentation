# fs-init

[← Zurück zum Index](../INDEX.md)

**Repo:** `FreeSynergy/fs-init` · `/home/kal/Server/fs-init/`
**Typ:** Binary (`fs-init`)
**Capabilities:** `init.bootstrap`

---

## Was ist das?

`fs-init` ist das Bootstrap-Binary für FreeSynergy.
Es erkennt die System-Capabilities via `fs-info`, führt einen interaktiven Install-Wizard durch
und klont den offiziellen Store-Katalog auf den Node — ohne `git` als System-Dependency.

---

## Design Pattern

| Pattern | Verwendung |
|---------|-----------|
| **Strategy** | `BootstrapStrategy`-Trait — wählt Gui / Tui / Headless basierend auf Capabilities |
| **State Machine** | `WizardMachine` — 7 Schritte in Sequenz, Back-Navigation möglich |

---

## Ablauf (Phase 1)

```
1. Capabilities erkennen (fs-info: display_server / terminal / headless)
2. Strategy wählen: GuiBootstrap | TuiBootstrap | HeadlessBootstrap
3. Wizard ausführen (7 Schritte, alle via println! / stdin):
   Welcome → Capability → Engine → Bundle → Confirm → Progress → Done
4. Im Progress-Schritt: Store-Katalog klonen
5. Strategy::launch(): Next-Steps anzeigen
```

---

## Wizard-Schritte

| Schritt | Klasse | Inhalt |
|---------|--------|--------|
| 1 Welcome | `WelcomeStep` | Begrüßung, erkannter Modus |
| 2 Capability | `CapabilityStep` | OS, Arch, Display, Terminal, Container, Modus (nur Info) |
| 3 Engine | `EngineStep` | Render-Engine wählen (iced / bevy / tui / none) |
| 4 Bundle | `BundleStep` | Bundle wählen (Minimal / Server / Workstation / Developer) |
| 5 Confirm | `ConfirmStep` | Zusammenfassung + y/N |
| 6 Progress | `ProgressStep` | Store klonen (Phase 2: Pakete installieren) |
| 7 Done | `DoneStep` | Pfad + Next Steps |

---

## Bootstrap-Modi

| Modus | Bedingung | Engine-Default | Bundle-Default |
|-------|-----------|----------------|----------------|
| **GUI** | `WAYLAND_DISPLAY` oder `DISPLAY` gesetzt | iced | Workstation |
| **TUI** | kein Display, aber TTY | tui | Server |
| **Headless** | kein Display, kein TTY | none | Server |

---

## Install-Targets

| Target | Wann vorgeschlagen |
|--------|--------------------|
| Container | Podman oder Docker erkannt |
| RPM | `/etc/redhat-release` oder `/etc/fedora-release` |
| DEB | `/etc/debian_version` |
| AppImage | Fallback |

---

## CLI

```bash
# Interaktiver Wizard (Standard)
fs-init

# Nur Store klonen, kein Wizard
fs-init --clone-only

# Eigene Store-URL
fs-init --store-url https://github.com/MyOrg/MyStore.git

# Eigenes Zielverzeichnis
fs-init --target-dir /opt/freesynergy/store

# Bestimmter Branch
fs-init --branch staging
```

| Flag | Default | Beschreibung |
|------|---------|-------------|
| `--store-url` | `https://github.com/FreeSynergy/Store.git` | Store-Repo-URL |
| `--target-dir` | `~/.local/share/freesynergy/store` | Zielverzeichnis |
| `--branch` | `main` | Git-Branch |
| `--clone-only` | false | Nur klonen, kein Wizard |

---

## Store-Pfad

```
~/.local/share/freesynergy/store/
```

(früher: `~/.local/share/fsn/store/` — mit Phase 1 auf neue Namenskonvention migriert)

---

## Implementierung

- Kein System-`git` nötig — nutzt `gix` (Pure-Rust)
- HTTP-Transport via `reqwest` + `rustls` (kein `libgit2`)
- Idempotent: wenn Store bereits vorhanden → Clone überspringen
- i18n: alle User-facing Texte als FTL-Keys in `keys.rs` (en Fallback fest eingebaut)

---

## Modul-Übersicht

```
src/
  main.rs            — Entry point, CLI-Args, Ablauf-Steuerung
  capability.rs      — BootstrapCapability, BootstrapMode, DisplayEnv, ContainerRuntime
  error.rs           — FsInitError
  keys.rs            — FTL-Key-Konstanten (en Fallback)
  store_clone.rs     — clone_store(), default_store_dir()
  strategy/
    mod.rs           — BootstrapStrategy-Trait, select_strategy(), run()
    gui.rs           — GuiBootstrap
    tui.rs           — TuiBootstrap
    headless.rs      — HeadlessBootstrap
  wizard/
    mod.rs           — WizardStep-Trait, WizardMachine, WizardState, WizardResult
    welcome.rs       — Schritt 1
    capability_step.rs — Schritt 2
    engine.rs        — Schritt 3
    bundle.rs        — Schritt 4
    confirm.rs       — Schritt 5
    progress.rs      — Schritt 6
    done.rs          — Schritt 7
```

---

## Ausstehend (Phase 2+)

| Phase | Inhalt |
|-------|--------|
| Phase 2 | Progress-Schritt: echte Paket-Installation via fs-store Pipeline |
| Phase 3+ | GUI-Wizard via fs-render / iced statt println! |
| Phase 5.1 | bootc install to-disk Integration (nach Kanidm) |
