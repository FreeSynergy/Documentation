# Store — Das Wissen & Der Paketmanager

[← Zurück zum Index](../../INDEX.md)

---

## Was der Store ist

Der Store ist der **universelle Paketmanager** von FreeSynergy. Vergleichbar mit dnf/apt, aber flexibler. Er verwaltet ALLES: Programme, Services, Themes, Sprachen, Widgets, Bots, Tasks.

Gleichzeitig ist er ein **Wissensspeicher** der allen Programmen hilft — der Conductor fragt "Kennst du diesen Service?" und der Store liefert Metadaten, Rollen, Variablen-Typen.

## Eigenständigkeit

Der Store funktioniert **ohne jedes andere FreeSynergy-Programm**. Er ist ein Git-Repository. CLI, API und GUI bieten identische Funktionen.

## Der Bootstrap-Prozess

```
FreeSynergy.Init (minimales Binary)
  → Klont den Store via gitoxide
    → Store installiert ALLES:
       Node, Desktop, Conductor, Services, Themes, Sprachen, ...
```

Siehe [Init](../init/README.md).

---

## Paket-Typen

| Typ | Inhalt | Beispiele |
|---|---|---|
| `app` | FreeSynergy-Kernanwendung | Node, Desktop, Conductor |
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
tags = ["server", "minimal", "node", "conductor", "proxy"]

[bundle]
packages = [
    "node",
    "conductor",
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

**Jedes Paket ist ein Objekt** mit Icon und Metadaten. Überall wo ein Paket angezeigt wird (Store, Desktop, Conductor, Settings) sieht man Icon, Name, Version, Tags.

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
- Podman/Quadlet-Setup → Script im `conductor`-Paket
- Firewall → Script im `zentinel`-Paket
- IAM-Bootstrap → Script im `kanidm`-Paket
- FCOS-spezifisches → Script im `fcos-support`-Paket

---

## Abhängigkeiten

Pakete können Abhängigkeiten deklarieren:

```toml
[dependencies]
fsn-node = ">= 0.5.0"
fsn-conductor = ">= 0.3.0"

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

Der [Conductor](../conductor/README.md) nutzt das bei der YAML-Analyse: "Kennst du diesen Service?" → Store ergänzt Daten, überschreibt NICHT.

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

Das Store-Git enthält ausschließlich **Manifeste** (TOML-Dateien, klein, textbasiert). Kompilierte Binaries gehören **nicht** ins Git — sie liegen auf **GitHub Releases**.

```
Store-Git (klein, nur Text):
  apps/node/manifest.toml        ← Metadaten, Version, Download-URL
  apps/desktop/manifest.toml
  apps/conductor/manifest.toml

GitHub Releases (große Dateien):
  FreeSynergy/Node/releases/v0.5.0/
    fsn-node-x86_64-linux.tar.gz
    fsn-node-aarch64-linux.tar.gz
  FreeSynergy/Desktop/releases/v0.5.0/
    fsn-desktop-x86_64-linux.tar.gz
```

Der Store liest das Manifest, findet die Download-URL, lädt das Binary von GitHub Releases. So bleibt das Store-Repo schlank und schnell klonbar.

**Regel für eigene Repos vs. Crates/Module:**

| Eigenes Repo | Crate/Modul im Eltern-Repo |
|---|---|
| Kann **alleine** laufen (Node, Desktop, Store, Init) | Macht nur als **Teil** eines anderen Programms Sinn |
| Beispiel: `FreeSynergy/Node`, `FreeSynergy/Desktop` | Beispiel: Browser → Crate in Desktop, Conductor → Crate in Node |

```
FreeSynergy/Desktop/          ← EIN Repo
  crates/
    fsn-ui/
    fsn-browser/              ← Browser ist ein Crate, kein eigenes Repo
    fsn-desktop/
```

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
| Paket-Typen + Manifest | `fsn-pkg` (`manifest.rs`) | `PackageType` (9 Varianten: app, container, bundle, …), `PackageMeta`, `BundleManifest` |
| Release-Channels | `fsn-pkg` (`channel.rs`) | `ReleaseChannel`: Stable/Testing/Nightly |
| Versionierung + Rollback | `fsn-pkg` (`versioning.rs`) | `VersionManager`, `VersionRecord`, `RollbackError` |
| Paket-Signierung | `fsn-crypto` (`signing.rs`, feature: `signing`) | `FsnSigningKey`, `FsnVerifyingKey`, `PackageSignature` — ed25519-dalek v2, SHA-256 |
| Signatur-Verifikation | `fsn-pkg` (`signing.rs`) | `SignatureVerifier`, `SignaturePolicy` (RequireSigned/TrustUnsigned) |
| SQLite-Tracking | `fsn-db` (`installed_package.rs`) | `installed_packages` Tabelle — `InstalledPackageRepo` |
| Store-Client | `fsn-store` | `StoreClient`, `Catalog<M>`, `CatalogCache` |
| Abhängigkeits-Auflösung | `fsn-pkg` (`dependency_resolver.rs`) | `DepGraph`, `DependencyResolver` (Kahn's Algorithmus) |
| Install-Lifecycle | `fsn-pkg` (`installer.rs`) | `PackageInstaller`, Hooks, `EventBus` |

## Repo

https://github.com/FreeSynergy/Store

---

Weiter: [Init](../init/README.md) | [Pakete](../../konzepte/pakete.md) | [Desktop](../desktop/README.md)
