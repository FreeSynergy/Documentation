# Store — Das Wissen & Der Paketmanager

[← Zurück zum Index](../../INDEX.md)

---

## Was der Store ist

Der Store ist der **universelle Paketmanager** von FreeSynergy. Vergleichbar mit dnf/apt, aber flexibler. Er verwaltet ALLES: Programme, Services, Themes, Sprachen, Widgets, Bots, Tasks.

Gleichzeitig ist er ein **Wissensspeicher** der allen Programmen hilft — der Container App Manager fragt "Kennst du diesen Service?" und der Store liefert Metadaten, Rollen, Variablen-Typen.

## Eigenständigkeit

Der Store funktioniert **ohne jedes andere FreeSynergy-Programm**. Er ist ein Git-Repository. CLI, API und GUI bieten identische Funktionen.

## Der Bootstrap-Prozess

```
FreeSynergy.Init (minimales Binary)
  → Klont den Store via gitoxide
    → Store installiert ALLES:
       Node, Desktop, Container App Manager, Services, Themes, Sprachen, ...
```

Siehe [Init](../init/README.md).

---

## Paket-Typen

| Typ | Inhalt | Beispiele |
|---|---|---|
| `app` | FreeSynergy-Kernanwendung | Node, Desktop, Container App Manager |
| `container` | Service-Modul (Quadlet + Config) | Kanidm, Forgejo, Outline |
| `bundle` | Meta-Paket das andere Pakete zusammenfasst | server-minimal, desktop-full |
| `language` | Sprach-Snippets (.ftl) | Deutsch, Französisch, Arabisch |
| `theme` | Visuelles Theme | Midnight Blue, Nordic |
| `widget` | Desktop-Widget | Uhr, System-Info, Nachrichten |
| `bot` | Bot-Definition | Broadcast, Gatekeeper |
| `bridge` | Service-zu-Service-Adapter | Forgejo→Matrix |
| `task` | Automatisierungs-Template | "Docs ins Wiki", "Daily Digest" |

**Hinweis:** Libraries (`fsn-*` Crates) sind KEINE eigenständigen Pakete. Sie sind Abhängigkeiten die mit den Anwendungen mitkommen und in einem shared-Ordner leben.

---

## Bundles (Meta-Pakete)

Wie bei dnf group — vordefinierte Sammlungen beliebiger Pakete (Apps, Widgets, Themes, …):

```toml
[package]
id = "server-minimal"
name = "Server Minimal"
type = "bundle"
description = "Minimale Server-Installation"
tags = ["server", "minimal", "node", "container-app", "proxy"]

[bundle]
packages = [
    "node",
    "container-app",
    "zentinel",
    "kanidm",
]
optional = [
    "forgejo",
    "outline",
    "stalwart",
]
```

```toml
[package]
id = "desktop-full"
name = "Desktop Vollständig"
type = "bundle"
tags = ["desktop", "full", "themes", "widgets"]

[bundle]
packages = [
    "desktop",
    "theme-midnight-blue",
    "theme-cloud-white",
    "theme-nordic",
    "lang-en",
    "lang-de",
    "widget-clock",
    "widget-sysinfo",
]
```

---

## Versionierung

### SemVer (MAJOR.MINOR.PATCH)

- **PATCH** (0.5.1 → 0.5.2): Bugfixes. API-kompatibel. Sicheres Update.
- **MINOR** (0.5.0 → 0.6.0): Neue Features. API-kompatibel. Sicheres Update.
- **MAJOR** (0.x → 1.0): Breaking Changes. Muss neu konfiguriert/installiert werden.

Jede Version = Git-Tag. Changelog aus Git-Commit-Messages. Rollback auf ältere Version jederzeit möglich.

### Parallele Versionen

Jedes Paket wird mit Versionsnummer installiert:

```
/opt/freesynergy/packages/kanidm-1.5.0/
/opt/freesynergy/packages/kanidm-1.4.0/   ← alte Version, noch da
```

**Standard:** `latest` installieren, vorherige Versionen löschen.
**Explizit:** Zwei Versionen parallel betreiben wenn gewünscht.

```bash
fsn store install kanidm                    # latest, alte löschen
fsn store install kanidm --version 1.4.0    # spezifische Version
fsn store install kanidm --keep-previous    # latest + alte behalten
```

### Release-Channels

| Channel | Zweck |
|---|---|
| `stable` | Getestet, empfohlen. Default. |
| `testing` | Neue Features, möglicherweise instabil |
| `nightly` | Aktuellster Stand, kann kaputt sein |

```bash
fsn store install kanidm --channel testing
fsn store config set default-channel stable
```

---

## Paket-Signierung

Jedes Paket wird signiert. **ed25519 als Default**, aber austauschbar.

```toml
[package.signature]
algorithm = "ed25519"          # Default, kann geändert werden
key_id = "freesynergy-main"
signature = "base64:..."
```

Der Store prüft die Signatur **vor** jeder Installation. Unsignierte Pakete werden nicht installiert (kann mit `--trust-unsigned` überschrieben werden, zeigt Warnung).

Unterstützte Algorithmen:
- ed25519 (Default)
- ed448
- RSA (Legacy-Kompatibilität)

---

## Paket-Metadaten (Pflicht)

```toml
[package]
id = "kanidm"                   # Eindeutiger Name (KEIN Typ-Prefix!)
name = "Kanidm"                 # Anzeigename
version = "1.5.0"               # SemVer, aus Git-Tag
type = "container"              # app, container, bundle, language, theme, ...
description = "Modern identity management"
icon = "kanidm"                 # SVG oder Icon-Name (PFLICHT)
tags = ["iam", "oidc", "scim", "mfa", "webauthn", "identity", "rust"]
author = "Kanidm Project"
license = "MPL-2.0"
homepage = "https://kanidm.com"
source = "https://github.com/FreeSynergy/Store"
```

**Jedes Paket ist ein Objekt** mit Icon und Metadaten. Überall wo ein Paket angezeigt wird (Store, Desktop, Container App Manager, Settings) sieht man Icon, Name, Version, Tags.

Jedes Paket MUSS ein Icon haben. Kein Icon → generisches Icon für den Typ.

---

## Tag-System

Tags sind das **primäre Suchinstrument**. Schlechte Tags = Paket unsichtbar.

| Pakettyp | Tag-Regeln |
|---|---|
| `container` | Alle Rollen + Unterrollen + kompatible Standards |
| `language` | Sprach-Code, Region, Anwendungs-ID |
| `theme` | Farb-Namen, Stil |
| `widget` | Funktion, Datenquelle |
| `bot` | Plattform, Funktion |
| `task` | Quell-Service, Ziel-Service, Funktion |
| `app` | Funktion, Plattform |
| `bundle` | Enthaltene Pakete, Zweck |

### Filter-Kombinationen

```
tags contains "iam"                                    # Alle IAM-Services
type = "container" AND tags contains "rust"            # Rust-Container
type = "language" AND tags contains "de"               # Deutsche Pakete
tags contains "mail" OR tags contains "smtp"           # Alles rund um Mail
```

---

## Paket-Lifecycle

```
Browse/Suche im Store
  → Installieren (Download + Signatur-Check + Abhängigkeiten auflösen + Scripts)
    → Konfigurieren
      → Nutzen
        → Update (neue Version, alte optional behalten)
          → Rollback (auf vorherige Version wechseln)
            → Deinstallieren ("Daten behalten?")
              → Vollständig löschen (alles weg)
```

Jedes Paket kann Scripts haben:

| Script | Wann |
|---|---|
| `pre_install` | Vor der Installation (Voraussetzungen prüfen) |
| `post_install` | Nach der Installation (Konfiguration, Setup) |
| `pre_remove` | Vor der Deinstallation (Aufräumen, Warnung) |
| `post_remove` | Nach der Deinstallation (Cleanup) |

Die Scripts aus der alten `fsn-install.sh` werden aufgeteilt:
- SSH-Key-Setup → Script im `node`-Paket
- Podman/Quadlet-Setup → Script im `container-app`-Paket
- Firewall → Script im `zentinel`-Paket
- IAM-Bootstrap → Script im `kanidm`-Paket
- FCOS-spezifisches → Script im `fcos-support`-Paket

---

## Abhängigkeiten

Pakete können Abhängigkeiten deklarieren:

```toml
[dependencies]
fsn-node = ">= 0.5.0"
fsn-container-app = ">= 0.3.0"

[optional-dependencies]
kanidm = ">= 1.4.0"     # Empfohlen, aber nicht Pflicht
```

Der Store löst Abhängigkeiten automatisch auf (wie dnf). Bei Konflikten: Warnung + Benutzer entscheidet.

---

## Store als Wissensquelle

Programme fragen den Store nach Metadaten:

```
GET /api/store/know/service/kanidm
→ Rollen, Variablen-Typen, Ports, Healthcheck

GET /api/store/know/variable/REDIS_URL
→ Typ: url, Rolle: cache.redis

GET /api/store/know/compatible?role=iam
→ Welche installierten Services haben diese Rolle?
```

Der [Container App Manager](../container_app/README.md) nutzt das bei der YAML-Analyse: "Kennst du diesen Service?" → Store ergänzt Daten, überschreibt NICHT.

---

## CLI-Interface

```bash
# Suche
fsn store search "mail"
fsn store search --type container --tag smtp
fsn store search --type bundle

# Info
fsn store info kanidm
fsn store info --version 1.4.0 kanidm

# Installieren
fsn store install kanidm
fsn store install kanidm --version 1.4.0
fsn store install kanidm --keep-previous
fsn store install kanidm --channel testing
fsn store install server-minimal              # Bundle

# Update
fsn store update kanidm
fsn store update --all

# Rollback
fsn store rollback kanidm                     # Vorherige Version

# Deinstallieren
fsn store remove kanidm
fsn store remove --keep-data kanidm
fsn store remove --purge kanidm               # Alles weg

# Installierte Pakete
fsn store list
fsn store list --type container
fsn store list --outdated

# Catalog
fsn store sync
fsn store sync --offline
```

## GUI — Store-Oberfläche im Desktop

### Sideboard (Pinned)

Der unterste Punkt im Sideboard ist **immer** ⚙️ **Einstellungen** (angepinnt). Dieser führt zur Repository-Verwaltung:

- Alle konfigurierten Repositories werden aufgelistet
- Jedes Repository kann **aktiviert** oder **deaktiviert** werden
- Repositories können **hinzugefügt** oder **entfernt** werden
- **Ausnahme:** Das Haupt-Repository (`freesynergy-main`) kann nur deaktiviert, aber **nicht gelöscht** werden

### Paket-Filter

Oben in der Store-Ansicht stehen vier Filter:

| Filter | Zeigt |
|---|---|
| **Alle** | Alle verfügbaren Pakete aller aktiven Repositories |
| **Installiert** | Nur installierte Pakete |
| **Verfügbar** | Pakete die noch nicht installiert sind |
| **Aktualisierbar** | Installierte Pakete für die eine neue Version vorliegt |

### Icons

Jedes Paket **muss** ein Icon haben. Im Store-UI werden Icons überall angezeigt: in der Paketliste, in der Detailansicht, in der Installationsbestätigung. Fehlt ein Icon im Paket, wird das generische Icon für den jeweiligen Pakettyp verwendet.

### Schnell-Installation vs. Detailansicht

Von der Paketliste aus gibt es zwei Wege:

1. **Direkt installieren** — ein Klick auf den Install-Button startet die Installation sofort
2. **Detailansicht** — Klick auf das Paket öffnet die Detailseite; von dort aus kann ebenfalls installiert werden

Ein **Pfeil oben links** (← Zurück) führt von der Detailansicht zurück zur Liste.

### Installations-Feedback

Nach einer Installation erscheint ein einfaches **Popup**:
- ✅ Installation erfolgreich
- ❌ Installation fehlgeschlagen

Kein Konfigurations-Zwischenschritt, keine weiteren Dialoge. Fertig.

### Detailansicht eines Pakets

Die Detailansicht zeigt alle relevanten Informationen zu einem Paket — schön aufbereitet:

| Bereich | Inhalt |
|---|---|
| **Header** | Icon, Name, Version, Typ-Badge, Kurzebeschreibung |
| **Vollständigkeit** | Ist das Paket vollständig? Sind alle Pflichtfelder gesetzt? (Icon, Beschreibung, Tags, Signatur, i18n, …) |
| **Sprachen** | Welche Sprachdateien (.ftl) sind enthalten? Liste der unterstützten Sprachen |
| **Icon** | Vorschau des Paket-Icons |
| **Tags** | Alle Tags als klickbare Filter-Chips |
| **Metadaten** | Autor, Lizenz, Homepage, Quell-Repository, Release-Channel |
| **Abhängigkeiten** | Direkte Abhängigkeiten + optionale Abhängigkeiten |
| **Signatur** | Signatur-Status (signiert / unsigniert + Key-ID) |
| **Screenshots/Previews** | Vorschaubilder wenn im Manifest hinterlegt |

### Bundles in der Detailansicht

Für Pakete vom Typ `bundle` zeigt die Detailansicht zusätzlich:
- Liste aller **enthaltenen Pakete** (inkl. Icons und Kurzbeschreibung)
- Liste der **optionalen Pakete**
- Jedes Paket in der Liste ist **anklickbar** → öffnet die Detailansicht des jeweiligen Pakets

---

## API-Interface

```
GET  /api/store/packages?q=&type=&tags=&installed=
GET  /api/store/packages/:id
GET  /api/store/packages/:id/versions
POST /api/store/install    { "id": "kanidm", "version": "1.5.0" }
POST /api/store/update     { "id": "kanidm" }
POST /api/store/rollback   { "id": "kanidm" }
DELETE /api/store/packages/:id?keep_data=true
GET  /api/store/know/service/:id
GET  /api/store/know/variable/:name
GET  /api/store/know/compatible?role=
POST /api/store/sync
```

---

## Binary-Verteilung

Kompilierte Rust-Binaries gehören **nicht** in Git-Repos. Git ist für Text — `target/`-Ordner können mehrere GB groß werden, GitHub hat ein 100 MB-Limit pro Datei.

**Drei Ebenen:**

**Ebene 1 — Git-Repos (nur Code + Text):**
Jedes Programm hat ein eigenes Repo mit Quellcode, Manifesten, Configs. Kein `target/`, keine `.exe`, keine `.so`.

**Ebene 2 — GitHub Releases (Binaries für Veröffentlichung):**
Nur für getaggte Releases. GitHub Actions kompiliert automatisch wenn ein Tag gepusht wird:
```
git tag v0.5.0 && git push --tags
→ GitHub Actions: kompiliert für linux-x86_64, linux-aarch64, macos-*, windows-x86_64
  → Lädt Binaries als Release-Assets hoch
```

**Ebene 3 — Lokale Entwicklung:**
```bash
cargo build --release
fsn store install --local ./target/release/fsn-node
# Store liest Metadaten aus lokalem Git, kopiert Binary, registriert im Inventory
```

### Manifest-Feld [distribution]

Jedes App-Manifest deklariert die Download-URLs je Plattform:

```toml
[package]
id = "node"
name = "FreeSynergy Node"
version = "0.5.0"
type = "app"
icon = "freesynergy-node"
tags = ["node", "server", "infrastructure"]

[interfaces]
cli  = true
api  = true
wgui = false
tui  = false

[distribution]
linux-x86_64   = "https://github.com/FreeSynergy/Node/releases/download/v{version}/fsn-node-x86_64-linux.tar.gz"
linux-aarch64  = "https://github.com/FreeSynergy/Node/releases/download/v{version}/fsn-node-aarch64-linux.tar.gz"
macos-x86_64   = "https://github.com/FreeSynergy/Node/releases/download/v{version}/fsn-node-x86_64-macos.tar.gz"
macos-aarch64  = "https://github.com/FreeSynergy/Node/releases/download/v{version}/fsn-node-aarch64-macos.tar.gz"
windows-x86_64 = "https://github.com/FreeSynergy/Node/releases/download/v{version}/fsn-node-x86_64-windows.zip"
source         = "https://github.com/FreeSynergy/Node"
```

### Installationsfluss

```
Veröffentlichtes Paket:
  fsn store install node
    → Store liest manifest.toml → erkennt Plattform
      → lädt Binary von GitHub Releases
        → registriert im Inventory

Lokal (Entwicklung):
  cargo build --release
  fsn store install --local ./target/release/fsn-node
    → Store liest Metadaten aus lokalem Git
      → kopiert Binary aus target/
        → registriert im Inventory

Source-Build (Fallback):
  fsn store install node --from-source
    → klont Repo → cargo build --release → wie oben
```

### catalog.toml — Zwei Ebenen

**Root `catalog.toml`** — App-Pakete (Binaries) mit GitHub Release-URLs:

```toml
[[packages]]
id      = "node"
type    = "app"
version = "0.1.0"
repo    = "https://github.com/FreeSynergy/Node"

[packages.distribution]
linux-x86_64  = "https://github.com/FreeSynergy/Node/releases/download/v{version}/fsn-node-x86_64-linux.tar.gz"
linux-aarch64 = "https://github.com/FreeSynergy/Node/releases/download/v{version}/fsn-node-aarch64-linux.tar.gz"
# ... weitere Plattformen
```

**`node/catalog.toml`** — Deployment-Module (Container-Apps, i18n):

```toml
[[packages]]
id          = "iam/kanidm"
name        = "Kanidm"
category    = "deploy.iam"
version     = "0.1.0"
icon        = "shared/icons/kanidm.svg"
path        = "node/modules/iam/kanidm"
```

Binaries werden **nie** im Store-Repo gespeichert. Jedes Programm hat ein eigenes GitHub-Repo und veröffentlicht Binaries über GitHub Releases wenn ein Git-Tag gepusht wird (`git tag v0.5.0 && git push --tags`).

---

## Geplante Features

Features die noch nicht implementiert sind, aber zum Store-Konzept gehören:

| Feature | Beschreibung |
|---|---|
| **Store-Mirrors** | Andere Betreiber hosten eigene Stores. Mehrere Store-Quellen konfigurierbar (wie dnf-Repos). |
| **Lokaler Cache** | Einmal heruntergeladen → offline verfügbar. Kein zweiter Download bei Neuinstallation. |
| **Download-Statistiken** | Zählt wie oft ein Paket installiert wurde. Sichtbar im Store-UI. |
| **Kompatibilitäts-Matrix** | Jedes Paket deklariert "Funktioniert mit Node >= 0.5.0". Warnung bei Inkompatibilität. |
| **Screenshots/Previews** | SVG-Previews im Manifest (besonders sinnvoll für Themes und Widgets). |
| **Changelog-Anzeige** | Automatisch aus Git-Tags generiert, direkt im Store sichtbar. |
| **Bewertungen/Reviews** | Community-Bewertungen (erst wenn Community existiert). |

---

## Verzeichnisstruktur

```
Store/
├── shared/          ← Für ALLE Programme
│   ├── i18n/
│   ├── themes/
│   ├── widgets/
│   ├── bots/
│   └── tasks/
├── node/            ← Für Node
│   ├── modules/
│   └── bridges/
├── desktop/         ← Für Desktop
├── bundles/         ← Meta-Pakete (Bundles)
└── catalog.toml     ← Maschinenlesbarer Gesamt-Index
```

---

## Implementierung (Crates)

| Bereich | Crate | Details |
|---|---|---|
| Ressource-Typen + Meta | `fsn-types` (`resources/meta.rs`) | `ResourceType` (16 Varianten), `ResourceMeta`, `ValidationStatus` |
| Ressource-Structs | `fsn-types` (`resources/*.rs`) | `AppResource`, `ContainerAppResource`, `WidgetResource`, `BotResource`, `BridgeResource`, `BundleResource`, Theme-Ressourcen |
| Release-Channels | `fsn-pkg` (`channel.rs`) | `ReleaseChannel`: Stable/Testing/Nightly |
| Versionierung + Rollback | `fsn-pkg` (`versioning.rs`) | `VersionManager`, `VersionRecord`, `RollbackError` |
| Paket-Signierung | `fsn-crypto` (`signing.rs`, feature: `signing`) | `FsnSigningKey`, `FsnVerifyingKey`, `PackageSignature` — ed25519-dalek v2, SHA-256 |
| Signatur-Verifikation | `fsn-pkg` (`signing.rs`) | `SignatureVerifier`, `SignaturePolicy` (RequireSigned/TrustUnsigned) |
| SQLite-Tracking | `fsn-db` (`installed_package.rs`) | `installed_packages` Tabelle — `InstalledPackageRepo` |
| Store-Client (generisch) | `fsn-store` (Lib.Ext) | `StoreClient`, `Catalog<M>`, `Manifest`-Trait, `I18nBundle` |
| Store-Client (Node) | `fsn-deploy` (`store.rs`) | `StoreClient` (FSN-spezifisch), `StoreEntry` implementiert `Manifest` |
| Abhängigkeits-Auflösung | `fsn-pkg` (`dependency_resolver.rs`) | `DepGraph`, `DependencyResolver` (Kahn's Algorithmus) |
| Install-Lifecycle | `fsn-pkg` (`installer.rs`) | `PackageInstaller`, Hooks, `EventBus` |
| Builder + Validierung | `fsn-builder` (Node) | `fsn-builder validate-store`, `validate`, `analyze`, `fetch-icon`, `publish` |

## Repo

https://github.com/FreeSynergy/Store

---

Weiter: [Init](../init/README.md) | [Pakete](../../konzepte/pakete.md) | [Desktop](../desktop/README.md)
